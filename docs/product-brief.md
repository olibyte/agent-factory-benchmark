# Product brief: Agent Factory Benchmark
## Summary
This is the benchmark app for the Factory v0 workflow in olibyte/agent-toolkit: a small task and issue tracker used to exercise an autonomous software factory end to end. The product is small on purpose, so the factory's planning, building, and testing steps are being tested, not the complexity of the application.

## Target users
The app is for an individual or a small team who need to track simple software or project tasks. A user signs in, creates projects, and manages tasks within them. There is no assumption of a large organization, multiple departments, or complex approval chains. The team is small enough that everyone who can see a project can also work in it.

## Goals
The product goal is a working, minimal task tracker: a signed-in user can organize work into projects, capture tasks, move them through a simple status, prioritize them, and see filtered lists and a summary of where things stand.

The benchmark goal is different but related: every capability in this brief must be small and well defined enough to become one or a few independent build tasks, each with a clear, automatable check. Features that would require open-ended design decisions, ambiguous data relationships, or subjective acceptance criteria are deliberately left out, so the factory's planning and verification steps have a clean, bounded target.

## Scope
### Projects
A signed-in user can create a project with a name. Projects group tasks. A user can see a list of their projects and open one to see its tasks.

### Task creation
Within a project, a user can create a task with a title and an optional description.

### Task editing
A user can edit a task's title, description, status, and priority after it is created.

### Task deletion
A user can delete a task. Deleting a task removes it from its project; it does not affect other tasks.

### Task status
Every task has exactly one status: Todo, In Progress, or Done. A new task starts as Todo. A user can change the status at any time, in any order.

### Task priority
Every task has exactly one priority level, chosen from a small fixed set. A user can change the priority at any time.

### Filtering
Within a project, a user can filter the task list by status, by priority, or by both at once. Clearing a filter returns to the full list.

### Dashboard
A user can see a small summary view showing, at a glance, how many tasks fall into each status and how many fall into each priority, across their projects.

## Non-goals
The following are out of scope for v1 and are not implied by any capability above:

- Comments or discussion threads on tasks
- Assigning tasks to specific people
- Due dates, reminders, or scheduling
- Labels or tags beyond status and priority
- Notifications, emails, or alerts
- File attachments on tasks or projects
- Real-time collaboration or live multi-user updates
- Team roles, permission levels, or ownership transfer
- Import or export of tasks or projects
- Search across tasks or projects beyond the status and priority filters
- Task history, audit logs, or activity feeds
- Sub-tasks, dependencies, or task linking

## Success criteria
- A signed-in user can create a project and immediately see it in their project list.
- A user can create a task in a project and see it appear in that project's task list with status Todo.
- A user can edit a task's title, description, status, and priority, and the updated values are what is shown afterward.
- A user can delete a task, and it no longer appears in the project's task list.
- A user can set a task's status to Todo, In Progress, or Done, and the task list reflects the current status for each task.
- A user can set a task's priority, and the task list reflects the current priority for each task.
- A user can filter a project's task list by status alone, by priority alone, and by both together, and only matching tasks appear.
- A user can view a dashboard that shows a count of tasks per status and per priority, and the counts match the underlying tasks.
- A user who is not signed in cannot view or change any project or task data.
- A user sees only the projects they are allowed to access, not projects belonging to other users or teams.

## Constraints
The application is built on the scaffold already in place: Next.js (App Router), React, TypeScript, Tailwind CSS, and Node 24.

## Assumptions
- Priority uses three levels: Low, Medium, and High. A new task defaults to Medium.
- Each project belongs to one user; projects are not shared between multiple accounts in v1. This keeps access rules simple: a user sees only the projects they created.
- Deleting a project deletes all of its tasks. A project cannot be left holding orphaned tasks.
- The dashboard is scoped to the signed-in user and covers all of their projects combined, rather than requiring a per-project view to be built separately.
- A task belongs to exactly one project and cannot be moved between projects in v1.
- There is one kind of authenticated account. There are no separate admin or guest roles.

## Open questions
- Should projects ever be shared across more than one user, or does every project belong to a single owner for the foreseeable roadmap?
- Should deleting a project always delete its tasks, or should the app warn the user or block deletion when tasks exist?
- Are the three priority levels (Low, Medium, High) sufficient, or does the operator want a different set, such as adding a fourth level?
- Should the dashboard be scoped per project, per user across all projects, or both?
- Does a small team need any shared visibility between accounts in v1, or is single-user ownership acceptable for this benchmark?
- Should task titles or project names have a maximum length or uniqueness requirement?
- Is a minimal sign-up flow needed, or can the benchmark assume accounts already exist and only test sign-in?
