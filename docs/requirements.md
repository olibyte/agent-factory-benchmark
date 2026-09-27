# Requirements: Agent Factory Benchmark

## Overview

This document sets out the requirements for the Agent Factory Benchmark, a small project and task tracker. The source is docs/product-brief.md, and later planning documents, including the architecture, data model, API and security decisions, acceptance tests, and task graph, cite these requirements by their ID.

## Functional requirements

### Access

- **FR-01** A user must be signed in to view or change any project or task.
- **FR-02** A signed-in user must see only the projects they created in their project list.
- **FR-03** A signed-in user must not be able to view or change a project or task that belongs to a different user.

### Projects

- **FR-04** A signed-in user must be able to create a project by giving it a name, and the project must appear in the user's project list immediately after creation.
- **FR-05** A signed-in user must be able to see a list of their own projects, ordered with the most recently created project first.
- **FR-06** A signed-in user must be able to open a project to see its list of tasks.
- **FR-07** The app must reject a project creation request whose name is empty or longer than 200 characters.
- **FR-08** A project's task list must show a message indicating that it has no tasks when the project contains none.

### Tasks

- **FR-09** A signed-in user must be able to create a task within a project by giving it a title, and the task must appear in the project's task list immediately after creation with status Todo and priority Medium.
- **FR-10** A user must be able to give a task an optional description at creation, or leave it blank.
- **FR-11** A user must be able to edit a task's title and description after it is created, and the task must show the updated values afterward.
- **FR-12** A user must be able to delete a task.
- **FR-13** Deleting a task must remove it from its project's task list and must not change any other task.
- **FR-14** A task must belong to exactly one project for its entire life and must not be movable to a different project.
- **FR-15** The app must reject a task creation or edit request whose title is empty or longer than 200 characters.

### Status and priority

- **FR-16** Every task must have exactly one status at a time, one of Todo, In Progress, or Done.
- **FR-17** A user must be able to change a task's status to any of the three statuses, in any order, at any time.
- **FR-18** A change to a task's status must be reflected immediately in the project's task list.
- **FR-19** Every task must have exactly one priority at a time, one of Low, Medium, or High.
- **FR-20** A user must be able to change a task's priority to any of the three levels at any time.
- **FR-21** A change to a task's priority must be reflected immediately in the project's task list.

### Filtering

- **FR-22** A user must be able to filter a project's task list to show only tasks with a chosen status.
- **FR-23** A user must be able to filter a project's task list to show only tasks with a chosen priority.
- **FR-24** A user must be able to filter a project's task list by status and priority together, showing only tasks that match both.
- **FR-25** Clearing a filter must return the task list to showing every task in the project.
- **FR-26** A filtered task list must show a message indicating that no tasks match when no tasks satisfy the chosen filter.

### Dashboard

- **FR-27** A signed-in user must be able to view a dashboard that shows a count of their tasks per status across all of their projects.
- **FR-28** A signed-in user must be able to view a dashboard that shows a count of their tasks per priority across all of their projects.
- **FR-29** The dashboard's counts must match the current underlying tasks at the time the dashboard is viewed.
- **FR-30** A signed-in user with no tasks must see a dashboard showing zero counts rather than an error.

## Non-functional requirements

- **NFR-01** The app must enforce access control on every read and every change to project or task data at the point the data is accessed, not only by hiding controls in the interface.
- **NFR-02** The app must validate all project and task input, including name, title, description, status, and priority, and must reject invalid values with a clear error instead of storing them.
- **NFR-03** Every action available in the interface, including creating, editing, deleting, changing status or priority, and filtering, must be operable using only a keyboard and must expose an accessible name to assistive technology.
- **NFR-04** The app must show a clear, visible message for every empty list, empty filter result, and failed action, rather than a blank screen or an unhandled error.
- **NFR-05** Every functional requirement must be verifiable by an automated test that checks observable behavior, such as what is displayed, stored, or returned, without inspecting internal implementation details.
- **NFR-06** The project and task data shown in the project list, a project's task list, filtered views, and the dashboard must always be consistent with each other, reflecting the same underlying data at the time of viewing.

## Traceability

| ID | Brief success criterion | Requirements |
| --- | --- | --- |
| SC-1 | Create a project and see it listed | FR-04 |
| SC-2 | Create a task and see it in the task list with status Todo | FR-09 |
| SC-3 | Edit a task's title, description, status, and priority, and see the updated values | FR-11, FR-17, FR-18, FR-20, FR-21 |
| SC-4 | Delete a task and it no longer appears | FR-12, FR-13 |
| SC-5 | Set a task's status and the task list reflects it | FR-16, FR-17, FR-18 |
| SC-6 | Set a task's priority and the task list reflects it | FR-19, FR-20, FR-21 |
| SC-7 | Filter by status alone, priority alone, and both together | FR-22, FR-23, FR-24 |
| SC-8 | View a dashboard with counts per status and per priority that match the underlying tasks | FR-27, FR-28, FR-29 |
| SC-9 | A signed-out user cannot view or change any project or task data | FR-01 |
| SC-10 | A user sees only the projects they are allowed to access | FR-02, FR-03 |

## Out of scope

This document keeps the brief's Non-goals unchanged; see the Non-goals section of docs/product-brief.md for the full list of excluded capabilities. In addition, this document leaves renaming and deleting a project out of v1: the app in this version only supports creating, listing, and opening projects. If project deletion is added in a later version, the brief's assumption applies, that deleting a project deletes all of its tasks so that no project is left holding orphaned tasks.

## Assumptions

- A project name and a task title must be between 1 and 200 characters; there is no uniqueness requirement across projects or tasks.
- Projects are listed with the most recently created project shown first, and a project's task list uses the same order.
- An empty list, whether a project list, a task list, or a filtered task list with no matches, shows a plain message stating that there is nothing to display, rather than being left blank.
- A dashboard for a user with no tasks shows zero counts for every status and priority, rather than an error or a blank page.

## Open questions

- (FR-02, FR-03) Should projects ever be shared between more than one account, or does every project belong to a single owner for the foreseeable roadmap?
- (FR-08) When project deletion is added in a later version, should it always delete its tasks, or should the app warn the user or block deletion when tasks exist?
- (FR-16, FR-19) Are the three priority levels, Low, Medium, and High, sufficient, or does the operator want a different set, such as a fourth level?
- (FR-07, FR-15) Should project names or task titles carry a uniqueness requirement, in addition to the length limit assumed here?
- (FR-01) How should a user sign in, and is a sign-up flow needed, or can the benchmark assume accounts already exist?
- (FR-01, NFR-01) Should a signed-in user be able to delete their own account and all of its associated data?
