# Architecture: Agent Factory Benchmark

## Overview

The Agent Factory Benchmark is a small, signed-in project and task tracker: a user signs up or signs in, creates or joins projects, and manages tasks inside them, with a dashboard summarizing status and priority across their projects. This document sets out the system's shape, its main components, and how a request reaches the database and back; the data model, API and security decisions, acceptance tests, and task graph that follow cite the decisions here by their AD ID.

## Constraints

- The scaffold is Next.js 16.3.6 with the App Router, React 19.2, TypeScript, Tailwind CSS 4, ESLint 9, and Vitest 5 with jsdom, running on Node 24 with npm 11.16. All of this stays as is.
- CI is GitHub Actions on ubuntu-24.04, running npm ci, npm run lint, npm run build, and npm test, with no database service and no secrets available to the job.
- Build tasks run on a factory worker: an Amazon Linux 2023 machine with no Docker and no browsers, whose checks run the same npm ci, lint, build, and test commands.
- npm 11.16 skips a package's install scripts unless package.json approves them in allowScripts, so a dependency that needs to download or compile a native binary would need that approval and a working toolchain on both the CI runner and the worker.
- Node 24 ships a built-in SQLite module, node:sqlite, that works without a flag, so the app does not need a separate database driver package or a database server.
- The app needs no hosted service at runtime, so deployment and hosting decisions are left out of this document.
- Tests run through npm test with Vitest; there are no browser end-to-end tests, because the worker has no browsers, so acceptance tests exercise server-side functions and rendered components instead.

## Decisions

- **AD-01** Persistence is SQLite in a local file. CI and the worker have no database service, and tests each use a fresh database of their own. (FR-12, NFR-06)
- **AD-02** Tests run with Vitest through npm test, with no browser end-to-end tests since the worker has no browsers; acceptance tests check observable behavior through server-side functions and rendered components. (NFR-05)
- **AD-03** The SQLite driver and data-access layer is Node's built-in node:sqlite module, wrapped by a small set of query functions per entity, rather than a separate driver package or an ORM; node:sqlite needs no native binary and no install-script approval, and the entity set is small enough that hand-written queries stay easy to read. (FR-12, NFR-06)
- **AD-04** Email and password sign-in is implemented as a small session module of the app's own, rather than a maintained auth library, because the app needs only sign-up, sign-in, sign-out, and account deletion with no social or federated providers, and a library built around those providers would add more surface than this app uses. (FR-01, FR-04)
- **AD-05** Passwords are hashed with Node's built-in crypto module's scrypt function, not a native module such as a compiled hashing library, so no dependency needs install-script approval on either CI or the worker. (FR-01, FR-03)
- **AD-06** Sessions are stateless signed cookies, set and read with Next.js's cookies API, with no server-side session table; the cookie carries only the signed-in user's id and is verified on every request that needs it, so there is no separate store to keep in sync with the database. (FR-04, FR-06)
- **AD-07** Every account, project, and task change is a Server Action, not a Route Handler, because every change in this app originates from an in-app form and Server Actions give secure server-side execution together with progressive enhancement, without a separate JSON API surface to maintain. (FR-12, FR-22)
- **AD-08** A data access layer (DAL) is the only place that reads or writes account, project, or task rows; it verifies the caller's session and, for project and task data, confirms the caller owns or is a member of the relevant project before touching SQLite. Server Components and Server Actions call only the DAL, never SQLite directly, so access control sits at the point data is read or changed, not only in the interface. (NFR-01, FR-10, FR-11)
- **AD-09** A proxy.ts at the project root performs an optimistic redirect, sending a visitor with no session cookie away from signed-in pages before any data is fetched; this is a convenience for a snappier redirect and is never the only check, since the DAL repeats the real check on every read and write. (NFR-01, FR-10)
- **AD-10** All account, project, and task input is checked by hand-written validation functions gathered in one module, covering trimmed length, required fields, enum membership for status and priority, and email shape; a schema-validation package is not added because the fields are few and the rules are simple enough to read directly. (NFR-02, FR-03)
- **AD-11** The app's entities are User, Project, ProjectMember, and Task. The accounts module owns User; the projects module owns Project and ProjectMember, the join that records who besides the owner can access a project; the tasks module owns Task, which always belongs to exactly one project. (FR-12, FR-17, FR-20)
- **AD-12** The dashboard computes its status and priority counts at view time by querying tasks across every project the signed-in user owns or is a member of; no separate cached summary table is kept, so the counts can never drift from the underlying tasks. (FR-38, FR-39, FR-40)
- **AD-13** Because participants see each other's changes only on their next page load, a Server Action refreshes only the acting user's own view after a write; no further propagation to other participants is attempted, and each of their page loads reads the current rows from SQLite through the same DAL functions used everywhere else. (FR-21, NFR-06)

## Components

- **Accounts and session** owns the User entity, sign-up, sign-in, sign-out, password confirmation for account deletion, and the signed cookie session that carries a user id across requests.
- **Data access layer (DAL)** owns no entity of its own; it centralizes session verification and the ownership and membership checks that every read and write of Project, ProjectMember, and Task data must pass, so those checks exist in exactly one place.
- **Projects** owns the Project and ProjectMember entities: creating a project, listing the projects a user owns or is a member of, opening a project, and adding or removing members.
- **Tasks** owns the Task entity within a project: creating, editing, deleting, changing status and priority, ordering the list by creation time, and filtering by status, priority, or both.
- **Dashboard** owns no entity; it reads Task rows across every project the signed-in user can access and turns them into per-status and per-priority counts.
- **Presentation** is the set of pages, forms, and lists built with Server and Client Components; it renders what the other components return and shows the required empty-list, empty-filter, and error messages, but never decides on its own what the user is allowed to see or change.
- **Persistence** is the SQLite file, its schema, and the connection helpers that the other components' data-access functions call into; it holds all four entities but carries none of the access-control logic itself.
- **Proxy** owns only the optimistic, cookie-only redirect described in AD-09; it holds no business logic and never reaches the database.

## Request flow

For a page view, for example opening a project's task list, the request first passes through Proxy, which reads the session cookie only and, if it is absent, redirects to the sign-in page as an optimistic convenience; Proxy never queries SQLite. The request then reaches the project page's Server Component, which calls the DAL's session-verification function to check the signed cookie; if the session is missing or invalid, the DAL itself redirects to sign-in, so no page renders without a real check even if Proxy is somehow bypassed. Once the session is confirmed, the Server Component calls a DAL read function such as the one that lists a project's tasks; that function first confirms the signed-in user owns or is a member of the requested project, and only then queries SQLite, which is how NFR-01 is met: access control sits at the point data is read, not only in what the interface chooses to show. The resulting rows are rendered into HTML and returned to the browser.

For a change, for example editing a task's status, the browser submits a form whose action is a Server Action. The Server Action starts by calling the same DAL session-verification function used for reads, then calls a DAL write function for that specific change. The write function re-checks that the user owns or is a member of the task's project, then validates the submitted fields, before it writes to SQLite, again enforcing NFR-01 at the point of the change rather than trusting that the form only appeared for an authorized user. After the write succeeds, the Server Action causes the acting user's own view to re-render with the new data; other participants who have the same project open see the change only the next time they load or reload the page, since updates are not pushed to them live.

## Module layout

```
app/
  layout.tsx
  page.tsx
  login/
    page.tsx
  signup/
    page.tsx
  account/
    page.tsx
  dashboard/
    page.tsx
  projects/
    page.tsx
    [projectId]/
      page.tsx
  actions/
    auth.ts
    projects.ts
    tasks.ts
  lib/
    db.ts
    session.ts
    password.ts
    dal.ts
    validation.ts
  ui/
    forms.tsx
    task-list.tsx
    filters.tsx
```

Route folders under app/ hold only pages and layouts; server-only logic lives in app/actions (one file of Server Actions per domain) and app/lib (persistence, session, password hashing, the DAL, and validation), with app/ui holding the shared presentational pieces those pages render. The single proxy.ts file required by Next.js sits at the project root next to app/, not inside it.

## Dependencies

This design adds no new npm packages: SQLite access uses Node's built-in node:sqlite (AD-03), password hashing and cookie signing use Node's built-in crypto module (AD-05, AD-06), sessions are read and set with Next.js's own cookies API (AD-06), and input validation is a small hand-written module (AD-10), so nothing beyond the existing scaffold needs to be installed.

## Testing

All requirements are tested with Vitest, run through npm test the same way CI already runs it, since neither the worker nor CI has a browser to drive real end-to-end tests. Each test file opens its own SQLite database, either an in-memory connection or a fresh temporary file, and runs the same schema setup the app uses at startup, so tests never share state with each other or with any developer's local database. A signed-in user is simulated by creating a user row directly through the accounts module and then calling the DAL and Server Action functions with that user's verified id, the same functions the app reaches after checking a real session cookie, rather than driving a browser through a sign-in form. Server and Client Components are rendered with the existing Testing Library and jsdom setup so tests can assert on displayed text, including the empty-list, empty-filter, and error messages required elsewhere in the requirements, and Server Actions are invoked directly with constructed form data to assert on validation errors, stored rows, and returned state. This is how NFR-05 is met: every functional requirement is checked through observable behavior, what a rendered component shows, what a server-side function returns, or what ends up in a test's own SQLite database, without inspecting internal implementation details.

## Requirement coverage

| Requirements | Components | Decisions |
| --- | --- | --- |
| FR-01, FR-02, FR-03, FR-04, FR-05, FR-06, FR-07, FR-08, FR-09 | Accounts and session | AD-04, AD-05, AD-06 |
| FR-10, FR-11, NFR-01 | Data access layer | AD-08, AD-09 |
| FR-12, FR-13, FR-14, FR-15, FR-16 | Projects | AD-01, AD-03, AD-11 |
| FR-17, FR-18, FR-19, FR-20, FR-21 | Projects (sharing) | AD-11, AD-13 |
| FR-22, FR-23, FR-24, FR-25, FR-26, FR-27, FR-28, FR-29, FR-30, FR-31, FR-32 | Tasks | AD-07, AD-10 |
| FR-33, FR-34, FR-35, FR-36, FR-37 | Tasks (list and filtering) | AD-01, AD-07 |
| FR-38, FR-39, FR-40 | Dashboard | AD-12 |
| NFR-02 | Validation | AD-10 |
| NFR-03, NFR-04 | Presentation | AD-07 |
| NFR-05 | Testing | AD-02 |
| NFR-06 | Persistence and data access layer | AD-01, AD-13 |

## Out of scope

This document does not fix tables, columns, or indexes; that belongs to the data model step. It does not set session lifetime, cookie attributes, password hashing parameters, or rate limiting; those belong to the security decisions step. It does not write acceptance test cases or a task graph of build tasks; those are the next two steps. It does not choose where the SQLite file lives at runtime, how the app is started, or any hosted service, cloud provider, or container tool, since the app needs no hosted service to run and deployment and hosting are left as an open question below. It does not revisit any of the operator's decisions recorded in docs/requirements.md, and it does not add capabilities beyond docs/requirements.md, such as project deletion, which stays out of v1.

## Assumptions

- Email format is checked with a simple, well-known pattern rather than a full mail-address specification, since the requirements only ask that obviously invalid addresses be rejected.
- The app runs as a single Node process at a time against its SQLite file, consistent with node:sqlite being a straightforward fit for one writer rather than many concurrent server instances.
- "Most recently created first" ordering for projects and tasks is implemented with a stored creation timestamp rather than an incrementing counter, since a timestamp is enough to order rows and is simpler to reason about across the accounts, projects, and tasks modules.
- Confirming account deletion with a password reuses the same check the sign-in flow already performs in the accounts module, rather than a second, separate credential check.

## Open questions

- (AD-01) Where should the SQLite file live and how should it persist across restarts once the app is actually deployed, since deployment and hosting are out of scope here?
- (AD-04, AD-06) Is a small custom session and sign-in implementation acceptable for as long as this app is expected to live, or should a maintained library be revisited once features such as account recovery are needed?
- (FR-17) Should the data model cap how many members a single project can have, or is an unbounded member list acceptable for v1?
- (NFR-06, AD-13) Since other participants only see changes on their next page load, does the dashboard need a manual refresh control, or is a normal page reload enough for this benchmark?
- (FR-09) Does the account-deletion warning need to name each project's task count, which would change what the accounts component reads before deleting, or is naming the affected projects enough?
