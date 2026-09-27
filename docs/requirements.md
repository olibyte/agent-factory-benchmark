# Requirements: Agent Factory Benchmark

## Overview

This document sets out the requirements for the Agent Factory Benchmark, a small project and task tracker. The sources are docs/product-brief.md and the operator's decisions recorded below, and where the two differ, the operator's decisions take precedence over the brief's assumptions, for example the brief assumed single-owner projects while the operator has decided that projects are shared. Later planning documents, including the architecture, data model, API and security decisions, acceptance tests, and task graph, cite these requirements by their ID.

## Decisions

- Projects are shared between one owner and any number of members, replacing the brief's assumption that every project belongs to a single owner; the owner and members can both fully manage the project's tasks. (FR-12, FR-17, FR-18, FR-20)
- Project deletion itself stays out of v1. When it is added later, the app must warn the user before deleting a project and its tasks, following the same warn-before-delete pattern already required for account deletion. (FR-09)
- The three priority levels, Low, Medium, and High, are confirmed as sufficient for v1, with no extra level added. (FR-22, FR-31, FR-32)
- Project names and task titles do not need to be unique, either across the app or within a project. (FR-13, FR-24)
- A sign-up flow is required, since the app cannot assume accounts already exist. (FR-01)
- Email and password is the sign-in method chosen to meet an MVP bar; no other sign-in method is required for v1. (FR-01, FR-04)
- A signed-in user can delete their own account, guarded by a password confirmation, with cleanup of the projects they own and their memberships elsewhere. (FR-07, FR-08, FR-09)
- Sharing is kept as small as Trello's model: an owner and members are the only two kinds of participant, and updates from one participant reach others on their next page load rather than live, in keeping with the rule of thumb to use Trello, or a very basic Jira, as the north star for later product questions. (FR-19, FR-21)

## Functional requirements

### Accounts

- **FR-01** A visitor must be able to sign up for an account with an email address and a password, and signing up must sign the user in.
- **FR-02** The app must treat email addresses as unique regardless of letter case and must reject a sign-up whose email address is already registered.
- **FR-03** The app must reject a sign-up whose password is fewer than 8 characters.
- **FR-04** A user must be able to sign in with their email address and password.
- **FR-05** A failed sign-in attempt must show a single message stating that the email address or password is wrong, without indicating which one.
- **FR-06** A signed-in user must be able to sign out.
- **FR-07** A signed-in user must be able to delete their own account after confirming with their password.
- **FR-08** Deleting an account must sign the user out, must prevent them from signing in again, and must make the email address available for a new sign-up.
- **FR-09** Deleting an account that owns any project must first show a warning naming those projects and stating that their members will lose access, must then delete those projects together with their tasks, and must remove the deleted user from every project where they were a member.

### Access

- **FR-10** A user must be signed in to view or change any project or task.
- **FR-11** A signed-in user must not be able to view or change a project, or any of its tasks, that they neither own nor are a member of.

### Projects

- **FR-12** A signed-in user must be able to create a project by giving it a name, the creator must become the project's owner, and the project must appear in the user's project list immediately.
- **FR-13** The app must reject a project creation request whose name, after trimming leading and trailing spaces, is empty or longer than 200 characters.
- **FR-14** A signed-in user's project list must show every project they own or are a member of, and must indicate which of those projects they own.
- **FR-15** A signed-in user must be able to open a project to see its list of tasks.
- **FR-16** A project's task list must show a message indicating that it has no tasks when the project contains none.

### Sharing

- **FR-17** The owner of a project must be able to add a member to the project by the email address of an existing account.
- **FR-18** The owner of a project must be able to remove a member from the project.
- **FR-19** Only the owner of a project must be able to add or remove its members.
- **FR-20** An owner and a member of a project must both be able to view the project and to create, edit, and delete its tasks, and to change a task's status and priority.
- **FR-21** A change made by one project participant must appear to other participants the next time they load the page, not immediately or live.

### Tasks

- **FR-22** A signed-in user must be able to create a task within a project by giving it a title, must be able to choose its priority at creation, and the task must start with status Todo and, if no priority is chosen, priority Medium.
- **FR-23** A user must be able to give a task an optional description at creation and may leave it blank, and the app must reject a description longer than 2000 characters.
- **FR-24** The app must reject a task creation or edit request whose title, after trimming leading and trailing spaces, is empty or longer than 200 characters.
- **FR-25** A user must be able to edit a task's title and description after it is created, and the task must show the updated values afterward.
- **FR-26** A user must be able to delete a task, and deleting it must remove it from its project's task list without changing any other task.
- **FR-27** A task must belong to exactly one project for its entire life and must not be movable to a different project.
- **FR-28** A project's task list must show its tasks ordered with the most recently created task first.

### Status and priority

- **FR-29** Every task must have exactly one status at a time, one of Todo, In Progress, or Done.
- **FR-30** A user must be able to change a task's status to any of the three statuses, in any order, at any time, and the project's task list must reflect the change immediately.
- **FR-31** Every task must have exactly one priority at a time, one of Low, Medium, or High.
- **FR-32** A user must be able to change a task's priority to any of the three levels at any time, and the project's task list must reflect the change immediately.

### Filtering

- **FR-33** A user must be able to filter a project's task list to show only tasks with a chosen status.
- **FR-34** A user must be able to filter a project's task list to show only tasks with a chosen priority.
- **FR-35** A user must be able to filter a project's task list by status and priority together, showing only tasks that match both.
- **FR-36** Clearing a filter must return the task list to showing every task in the project.
- **FR-37** A filtered task list must show a message indicating that no tasks match when no tasks satisfy the chosen filter.

### Dashboard

- **FR-38** A signed-in user must be able to view a dashboard that shows a count of tasks per status across every project they own or are a member of.
- **FR-39** A signed-in user must be able to view a dashboard that shows a count of tasks per priority across every project they own or are a member of.
- **FR-40** The dashboard's counts must match the current underlying tasks at the time it is viewed, and a user with no tasks must see zero counts rather than an error.

## Non-functional requirements

- **NFR-01** The app must enforce access control on every read and every change to account, project, and task data at the point the data is accessed, not only by hiding controls in the interface.
- **NFR-02** The app must validate all account, project, and task input, including email, password, name, title, description, status, and priority, and must reject invalid values with a clear error instead of storing them.
- **NFR-03** Every action available in the interface, including signing up, signing in, creating, editing, deleting, changing status or priority, filtering, adding or removing a member, and deleting an account, must be operable using only a keyboard and must expose an accessible name to assistive technology.
- **NFR-04** The app must show a clear, visible message for every empty list, empty filter result, and failed action, rather than a blank screen or an unhandled error.
- **NFR-05** Every functional requirement must be verifiable by an automated test that checks observable behavior, such as what is displayed, stored, or returned, without inspecting internal implementation details.
- **NFR-06** The project and task data shown in the project list, a project's task list, filtered views, and the dashboard must always be consistent with each other, reflecting the same underlying data at the time of viewing.

## Traceability

| ID | Brief success criterion | Requirements |
| --- | --- | --- |
| SC-1 | Create a project and see it listed | FR-12 |
| SC-2 | Create a task and see it in the task list with status Todo | FR-22 |
| SC-3 | Edit a task's title, description, status, and priority, and see the updated values | FR-25, FR-30, FR-32 |
| SC-4 | Delete a task and it no longer appears | FR-26 |
| SC-5 | Set a task's status and the task list reflects it | FR-29, FR-30 |
| SC-6 | Set a task's priority and the task list reflects it | FR-31, FR-32 |
| SC-7 | Filter by status alone, priority alone, and both together | FR-33, FR-34, FR-35 |
| SC-8 | View a dashboard with counts per status and per priority that match the underlying tasks | FR-38, FR-39, FR-40 |
| SC-9 | A signed-out user cannot view or change any project or task data | FR-10 |
| SC-10 | A user sees only the projects they are allowed to access | FR-11, FR-14 |

## Out of scope

This document keeps the brief's Non-goals unchanged; see the Non-goals section of docs/product-brief.md for the full list of excluded capabilities, including assignees, comments, due dates, labels, notifications, attachments, real-time collaboration, and import and export. In addition, this document leaves email verification and password reset out of v1, because both would need email delivery. Project deletion itself also stays out of v1: the app in this version only supports creating, listing, and opening projects, and deleting a project and its tasks is not yet possible. When project deletion is added, the app must warn the user before deleting a project and its tasks, naming the project and explaining the consequence, following the same pattern as the account-deletion warning. Within the sharing model, inviting a person who has no account, a member leaving a project on their own, and transferring ownership of a project are all out of scope.

## Assumptions

- A user's project list is ordered with the most recently created project shown first.
- Removing a member from a project does not delete or reassign the tasks already in that project; the tasks remain part of the project for the owner and any remaining members.
- Confirming account deletion with a password uses the same check as signing in, with no separate confirmation step such as re-entering the email address.
- Task descriptions are plain text, with no formatting, links, or attachments beyond the 2000-character limit.

## Open questions

- (FR-14, FR-17) Should a project's member list be visible to its owner and members, so they can see who else has access?
- (FR-09) Does the account-deletion warning need to list each named project's task count, or is naming the projects enough detail?
- (FR-17) Is there a limit to how many members a single project can have?
