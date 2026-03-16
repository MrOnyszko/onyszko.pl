# 09 - PRD Creation Experiment<no value>

In this post, I will show you how to use local AI models to generate Product Requirement Documents that developers can
act on directly.

## Why

Writing good requirements is hard. Writing good requirements consistently is even harder. Most developers skip PRD
creation entirely and jump straight to coding — then wonder why scope creeps, priorities blur, and features don't quite
fit together.
Using AI for PRD creation gives you three things that manual processes don't: discipline (the model forces you to answer
specific questions), speed (minutes instead of hours), and consistency (each PRD follows the same structure). The local
LLM aspect means you get this without sending your product ideas to the cloud.

## How

### What is a PRD?

A Product Requirement Document is a structured artifact that describes what you're building, why you're building it, and
how you'll know it's done. It's not a spec — it's the contract between product thinking and engineering execution.

A good PRD answers:

1. **What** — The feature or product and its core value proposition
2. **Who** — The target user and their problem
3. **Why** — The business case and success criteria
4. **How** — The functional requirements and user flows
5. **When** — Timeline and milestones

> A PRD is not a technical spec. Don't include database schema or API endpoints — save that for later.

### Why PRD is Useful

The discipline of writing a PRD catches problems before you write code:

- **Scope clarity** — "It should do X" becomes "It must do X, Y, Z and NOT A, B." That's a 15-minute conversation that
  saves 15 days of rework.
- **Dependency mapping** — When you write down what the feature needs, you see what other systems it touches.
- **Testable acceptance criteria** — A PRD that says "users can create tasks" is useless. A PRD that says "users can
  create tasks with title, due date, and priority; tasks appear in list sorted by due date" gives QA something concrete.
- **Onboarding aid** — New developer joins the team? Drop the PRD in Slack and they're up to speed in 10 minutes.

### Let's Try Creating One with Local LLM

> Do not fret, the post is long because I included many PRDs so you could see real results from different models. I
> limited myself to unsloth Qwen 3.5 family of models for simplicity.

I asked my local Qwen3.5-35B-A3B Q_K_M to create a PRD for adding Tasks functionality to Aily — my existing notes, mood
tracker, and habits app.

Here's the prompt I used:

```
Create a Product Requirement Document for adding Tasks functionality to an existing mobile app called Aily.

Context:
- Aily currently has: Notes, Mood Tracker, Habits
- The app is built with Flutter
- Target users: individuals who want to track personal daily tasks

Create a PRD with these sections:
1. Overview (one paragraph)
2. User Stories (3-5)
3. Functional Requirements
4. Non-Functional Requirements
5. UI/UX Guidelines
6. Acceptance Criteria
7. Out of Scope

Make it detailed enough that a developer could implement from it.
```

Here's what the local unsloth/Qwen3.5-9B Q8_K_XL produced:

```markdown
# Product Requirement Document (PRD): Aily Tasks Functionality

## 1. Overview

The Aily app is currently a personal wellness companion featuring Notes, a Mood Tracker, and Habit tracking. To provide
a comprehensive solution for personal daily productivity, we are introducing a new core module: **Tasks**. Unlike "
Habits," which are binary (done/not done) and recurring, "Tasks" are specific, actionable items that may have unique due
dates, priority levels, and completion status. This feature will integrate seamlessly with the existing Flutter
architecture, utilizing the current state management solution (e.g., BLoC/Riverpod) and database structure, ensuring
that users can manage a dynamic list of daily to-do items while maintaining the app's minimalist aesthetic and focus on
personal growth.

## 2. User Stories

* **US-01:** As a user, I want to create a specific task with a custom title and optional description, so that I can
  break down my daily goals into manageable steps.
* **US-02:** As a user, I want to set a specific due date and time for a task, so that the app can remind me or alert me
  if I am running late.
* **US-03:** As a user, I want to mark tasks as complete or delete them instantly, so that I can keep my daily list
  clean and focused on active items.
* **US-04:** As a user, I want to view all my tasks in a chronological list and filter them by "All," "Incomplete," or "
  Complete," so that I can easily track my daily progress.
* **US-05:** As a user, I want tasks to sync across my devices automatically when logged in, so that my productivity
  list is available wherever I am using Aily.

## 3. Functional Requirements

### FR-3.1: Task Creation

* **FR-3.1.1:** Users can add a new task via a floating action button (FAB) or an "Add Task" header button.
* **FR-3.1.2:** The creation form must include fields for:
    * **Title:** Mandatory text input.
    * **Description:** Optional text input (supports basic line breaks).
    * **Due Date:** Optional date picker (defaults to current day).
    * **Due Time:** Optional time picker.
    * **Category:** Mandatory dropdown selection (e.g., Work, Personal, Health, Errands).
* **FR-3.1.3:** The app must validate that the "Title" is not empty before saving.

### FR-3.2: Task Management & Editing

* **FR-3.2.1:** Users can tap an existing task to open an edit/view screen.
* **FR-3.2.2:** All fields created in FR-3.1 are editable on the task detail screen.
* **FR-3.2.3:** Users can toggle the **Status** (Completed/Incomplete) directly from the list view (via strikethrough
  animation) and the detail view.
* **FR-3.2.4:** Users can delete a task via a swipe gesture (if using a swipe-enabled list library) or a three-dot menu.

### FR-3.3: Notification & Reminders

* **FR-3.3.1:** If a "Due Time" is set, the app must request local notification permissions (handled via
  `flutter_local_notifications`).
* **FR-3.3.2:** A gentle notification should be triggered at the specified due time if the task status is still "
  Incomplete."
* **FR-3.3.3:** Users must be able to dismiss the notification and optionally mark the task as completed upon opening
  it.

### FR-3.4: Data Persistence & Sync

* **FR-3.4.1:** All tasks must be saved to the local database (Firebase Firestore or Hive/Sqflite depending on current
  Aily architecture).
* **FR-3.4.2:** Tasks must include a `createdAt` timestamp and an `updatedAt` timestamp.
* **FR-3.4.3:** Changes made on one device must propagate to other devices logged into the same user account within 5
  seconds.

### FR-3.5: Filtering & Sorting

* **FR-3.5.1:** The main list view must include a segmented control or dropdown to filter by: "All," "Incomplete," "
  Complete."
* **FR-3.5.2:** By default, the list should be sorted by "Due Date" (ascending), then by "Created Date."

## 4. Non-Functional Requirements

* **NFR-4.1 Performance:** The task list must load within 1.5 seconds even with 100+ items. Animations for status
  toggling (strikethrough) must be 60fps.
* **NFR-4.2 Reliability:** The system must achieve 99.9% uptime for data synchronization. No task data should be lost
  during app crashes or network outages.
* **NFR-4.3 Security:** All data in transit must be encrypted using TLS 1.3. Authentication tokens must be stored
  securely.
* **NFR-4.4 Battery Efficiency:** Local notifications and polling for task updates must be optimized to avoid
  unnecessary wake locks on the device.
* **NFR-4.5 Scalability:** The database schema must support up to 1 million concurrent users without degradation.

## 5. UI/UX Guidelines

* **Design Language:** The Tasks module must strictly adhere to Aily's existing design system (colors, typography,
  border radius, spacing).
* **List View:**
    * Use a card-based layout.
    * **Completed Tasks:** Text should have a light gray strikethrough, background should become slightly opaque, and
      the card border should fade.
    * **Incomplete Tasks:** Full color, bold title.
    * **Metadata:** Due date/time and category tag must appear in a small, muted font at the bottom right of the card.
* **Feedback:**
    * **Success:** A subtle haptic vibration and a checkmark animation when marking a task complete.
    * **Error:** A red text alert and a shaking animation if form validation fails.
* **Navigation:**
    * Bottom navigation bar should add a new tab: "Tasks" (placed after "Habits").
    * Swipe gestures: Swipe left to "Complete", Swipe right to "Delete" (optional, dependent on current gesture library
      implementation).
* **Empty State:** If no tasks exist, display an illustration of a checklist with the text: "No tasks yet. Add your
  first goal to stay productive!"

## 6. Acceptance Criteria

| ID | Criteria Description | Pass/Fail |
| :--- | :--- | :--- |
| **AC-1** | User can create a task with only a Title and default date/time. | |
| **AC-2** | User cannot create a task without a Title; validation message appears. | |
| **AC-3** | Task list refreshes immediately after marking a task as complete (real-time). | |
| **AC-4** | Local notification fires exactly at the set due time if task is incomplete. | |
| **AC-5** | Filtering by "Incomplete" hides all completed tasks correctly. | |
| **AC-6** | Data created on Mobile Web (if exists) or Mobile App syncs to the other platform. | |
| **AC-7** | App handles network loss gracefully; data saves locally and syncs once connected. | |
| **AC-8** | UI matches Figma mockups for Task List, Add Task, and Detail views. | |

## 7. Out of Scope

The following features are intentionally excluded from this initial release (MVP):

* **Sub-tasks:** Breaking a main task into smaller child tasks.
* **Recurring Tasks:** Automatically creating the same task every day/week (this overlaps with the existing "Habits"
  feature).
* **Collaborative Tasks:** Inviting friends or family to a task list.
* **Attachments:** Adding images or files to a task.
* **Task Templates:** Saving a task configuration to reuse later.
* **Dark Mode Specifics:** If Aily is already fully dark-mode compatible, specific new assets are not required, but
  existing components must render correctly in dark mode.
* **Analytics Dashboard:** Detailed reporting on task completion rates over time.
```

Here's what the local unsloth/Qwen3.5-35B-A3B Q4_K_M produced:

```markdown
# Product Requirement Document (PRD): Aily Task Management Module

**Product Name:** Aily
**Feature Name:** Task Management
**Version:** 1.0.0
**Platform:** iOS & Android (Flutter)
**Status:** Draft
**Date:** October 26, 2023

---

## 1. Overview

The Task Management feature aims to enhance Aily's existing ecosystem of wellness and productivity tools (Notes, Mood
Tracker, Habits) by providing users with a mechanism to manage specific, deadline-driven action items. Unlike the "
Habits" feature, which focuses on repetitive behaviors, "Tasks" will focus on one-off or time-bound goals. This module
will integrate seamlessly with the current UI design system and allow users to create, prioritize, complete, and track
tasks. The addition of Tasks will provide users with a daily checklist, complementing the longitudinal tracking of
Habits and the retrospective logging of Mood.

---

## 2. User Stories

1. **As a** user, **I want to** create a new task with a title, description, and due date, **so that** I can capture
   immediate to-dos before they are forgotten.
2. **As a** user, **I want to** mark a task as complete and view it in a "Completed" tab, **so that** I can feel a sense
   of accomplishment and clear my inbox.
3. **As a** user, **I want to** receive a push notification when a task is due, **so that** I am reminded to complete it
   at the right time.
4. **As a** user, **I want to** filter my tasks by "Today" or "Upcoming," **so that** I can focus only on what requires
   immediate attention without clutter.

---

## 3. Functional Requirements

### 3.1. Data Model

Tasks must adhere to the following JSON structure (stored locally via Hive/SQLite and synced via Firestore/Backend):

{
  "taskId": "uuid_v4",
  "title": "string (max 100 chars)",
  "description": "string (nullable)",
  "isCompleted": "bool",
  "createdAt": "timestamp",
  "dueDate": "timestamp (nullable)",
  "priority": "enum (Low, Medium, High, Critical)",
  "tags": ["list of string"],
  "completedAt": "timestamp (nullable)",
  "isArchived": "bool"
}


### 3.2. Task Creation & Management

* **CRUD Operations:** Users must be able to Create, Read, Update (edit title, date, priority), and Delete (soft delete)
  tasks.
* **Input Fields:** Task Title is mandatory. Description, Due Date, and Priority are optional.
* **Due Date:** Can be set to specific time (Time Picker) or All Day. Defaults to "Today".
* **Priorities:** Visual distinction required for High (Red) vs. Low (Green/Blue) priority.
* **Sorting:** Default sort order is `dueDate ASC`, then `isCompleted ASC`.

### 3.3. Completion Logic

* **Toggle:** Tapping the checkbox marks task as `completed = true`.
* **Haptic Feedback:** Trigger a `success` haptic feedback pattern upon completion.
* **Animation:** Completed tasks should visually fade out or strike-through before moving to a "Completed" drawer.

### 3.4. Notifications

* **Local Notifications:** Triggered via `flutter_local_notifications` plugin.
* **Timing:** Notifications must fire 1 hour and 15 minutes before `dueDate` if the task is not completed.
* **Permissions:** App must request notification permission on first task creation if not previously granted.

### 3.5. Integration with Existing Modules

* **Mood Tracker:** When a task marked `High` priority is completed, prompt a subtle "Good job" notification (
  non-intrusive).
* **Habit Tracker:** If a Habit is linked to a Task (future feature), the Task completion should increment the Habit
  counter.
* **Navigation:** Add "Tasks" tab to the bottom navigation bar alongside Notes, Habits, and Mood.

---

## 4. Non-Functional Requirements

* **Performance:** Task list must render under 16ms per frame on a mid-tier Android device (Flutter Performance). List
  must handle up to 500 items without lag.
* **Offline First:** Task creation, editing, and completion must work offline. Sync to cloud must be queued and
  processed once connectivity is restored.
* **Battery Usage:** Notifications must use the system's batching where possible to avoid battery drain.
* **Accessibility:** All task items must support TalkBack (Android) and VoiceOver (iOS). Checkbox labels must be
  distinct.
* **Data Retention:** Completed tasks older than 90 days must be automatically archived (soft delete) unless pinned by
  the user.

---

## 5. UI/UX Guidelines

* **Design System:** Reuse existing Aily tokens (Colors, Typography, Spacing).
* *Primary Button:* Aily Primary Purple (`#6C5CE7`).
* *Background:* Aily Off-White (`#F8F9FA`).
* **List View:**
* Card-based layout for each task.
* Show Priority Badge on the top-right of the card.
* Show Due Date on the bottom-left (Bold if overdue).
* **Overdue State:** If `dueDate` < `currentTime` AND `isCompleted` == `false`:
* Change border color to Light Red.
* Add "Overdue" text tag.
* **Empty States:** Display an illustration of a checklist with the text "No tasks today. Go crush some goals!"
* **Add Button:** Floating Action Button (FAB) + Bottom Sheet (for mobile) for new task input.

---

## 6. Acceptance Criteria

1. **Creation:** User can tap FAB -> Fill Title -> Save -> Task appears in "Today" list immediately.
2. **Persistence:** User closes app -> Reopens -> Task remains visible and data matches exactly.
3. **Completion:** User taps checkbox -> Task visualizes strike-through -> Task moves to bottom of list (or drawer).
4. **Notification:** If due time is reached, local notification appears (tested on device).
5. **Editing:** User opens existing task -> Changes priority to "High" -> Saves -> Badge color updates to Red.
6. **Sync:** User creates task on Device A -> Logs into Device B -> Task appears on Device B.
7. **Error Handling:** If internet fails during sync, show toast message "Syncing later..." but allow local edit.

---

## 7. Out of Scope

* **Sub-tasks/Checklists:** A Task cannot have nested lists of sub-tasks in v1.0.
* **Collaboration:** Sharing tasks with other users is not supported.
* **Recurring Tasks:** While similar to Habits, the Task module will not handle "Every Day at 8 AM" logic (this remains
  in Habits). Tasks are single-instance or manually repeated.
* **Smart Calendar:** Tasks will not appear on a full-month calendar view in this release.
* **AI Suggestions:** No automatic task generation based on Mood data yet.

```

Here's what the local unsloth/Qwen3.5-122B-A10B Q6_K produced:

```markdown
# Product Requirement Document (PRD): Aily Tasks Module

**Project:** Aily Mobile App
**Feature:** Task Management (Tasks)
**Version:** 1.0
**Date:** October 26, 2023
**Platform:** iOS & Android (Flutter)
**Author:** Product Team

---

## 1. Overview
Aily currently supports users in tracking their mental state (Mood), routines (Habits), and reflections (Notes). To create a holistic daily planner, we must introduce a **Tasks** module. This feature bridges the gap between recurring habits (routines) and one-off actions (errands, appointments). The Tasks feature will allow users to create, schedule, prioritize, and complete individual action items with specific due dates. Unlike Habits, which are tracked for streak-based consistency, Tasks are transactional and meant to be checked off once or on a specific timeline. This integration aims to reduce cognitive load by centralizing all actionable items within the Aily wellness ecosystem.

---

## 2. User Stories
1.  **As a user**, I want to create a new task with a title, due date, and priority level so that I can track what needs to be done by a specific time.
2.  **As a user**, I want to view all my tasks in a list sorted by due date so that I can see what is upcoming or overdue.
3.  **As a user**, I want to mark a task as complete with a satisfying animation so that I feel a sense of accomplishment.
4.  **As a user**, I want to receive a push notification before a task is due so that I don't forget important actions.
5.  **As a user**, I want to filter my tasks by status (Active, Completed) so that I can focus on pending items without clutter.

---

## 3. Functional Requirements

### 3.1. Task Creation & Management
*   **FR-01:** Users must be able to add a task from the Tasks Tab or via a Floating Action Button (FAB) on the Dashboard.
*   **FR-02:** **Task Fields:**
    *   `Title` (String, Required, Max 100 chars)
    *   `Description` (String, Optional, Max 500 chars)
    *   `Due Date` (DateTime, Required)
    *   `Priority` (Enum: Low, Medium, High)
    *   `Category/Tag` (Optional, String list, Max 3)
    *   `IsCompleted` (Boolean, Default: false)
*   **FR-03:** Users can edit task details (except completion status) at any time before the task is completed.
*   **FR-04:** Users can delete a task. A "Soft Delete" (confirmation dialog) is required for deletion.
*   **FR-05:** Users can toggle `IsCompleted`. Completion time must be logged (`CompletedAt` timestamp).

### 3.2. Task Display & Sorting
*   **FR-06:** The main Task List view must group tasks into three sections: **Overdue** (red accent), **Today**, and **Upcoming** (future).
*   **FR-07:** Completed tasks should move to a separate "Completed" view or be collapsed at the bottom of the list (configurable in settings).
*   **FR-08:** Users can sort the list by `Due Date` (Default), `Priority`, or `Created Date`.
*   **FR-09:** High Priority tasks must display a visual indicator (e.g., colored dot or border).

### 3.3. Notifications & Reminders
*   **FR-10:** Users can opt-in to "Due Date Reminders" per task.
*   **FR-11:** If enabled, the system must schedule a local notification 1 hour before the `Due Date`.
*   **FR-12:** Notification permission request must be handled gracefully (prompt on first task creation).
*   **FR-13:** Notification sound must match the existing Aily app sound profile.

### 3.4. Integration & Data Sync
*   **FR-14:** The Tasks Tab must be visible on the bottom navigation bar (between Habits and Notes).
*   **FR-15:** The Dashboard must display a widget showing "X tasks pending today."
*   **FR-16:** Data must sync via the existing Aily Backend API to ensure consistency across devices.
*   **FR-17:** Tasks must be creatable offline and saved locally (using `Hive` or `SQLite`), syncing when connectivity is restored.

---

## 4. Non-Functional Requirements

*   **NFR-01 (Performance):** Task list scrolling must maintain 60 FPS. Load time for the list should not exceed 1 second on a 3G connection.
*   **NFR-02 (Battery):** Background scheduling for notifications must not drain more than 2% of battery daily.
*   **NFR-03 (Security):** All user task data must be encrypted in transit (TLS 1.3) and at rest (using device keychain/secure storage).
*   **NFR-04 (Accessibility):** All interactive elements must support TalkBack (Android) and VoiceOver (iOS). Text contrast must meet WCAG AA standards.
*   **NFR-05 (Offline First):** Core CRUD operations on tasks must work without an internet connection.
*   **NFR-06 (Tech Stack):** Implementation must utilize the existing Flutter state management solution (e.g., Riverpod/Bloc) and existing UI component library.

---

## 5. UI/UX Guidelines

### 5.1. Design System Consistency
*   **Color Palette:** Use the existing Aily brand colors. High Priority tasks should use the existing "Alert" color (e.g., Coral/Salmon), Low Priority should use the "Success" or "Neutral" color.
*   **Typography:** Match the existing font scale (e.g., Roboto or Inter). Task Titles should be `Body Large`, Details `Body Small`.
*   **Spacing:** Maintain the existing 16px content padding. Card elevation should match the existing Note cards (low elevation, rounded corners).

### 5.2. Interaction Design
*   **Completion Animation:** When a user taps a task to complete it, trigger the existing "Checkmark" animation used in the Habit Tracker to maintain mental model consistency.
*   **Swipe Actions:** Allow users to swipe right to mark as complete, and swipe left to delete (or edit).
*   **Empty State:** If no tasks exist, show an illustration and a prompt: "No tasks for today. Time to relax or add something new!"
*   **Navigation:** The "Tasks" icon on the Bottom Nav Bar should be a "Checklist" icon, distinct from the "Star" (Habits) and "Pen" (Notes).

### 5.3. Differentiation from Habits
*   **Habit vs. Task:** Visually distinguish tasks from habits. Habits should have a circular progress/streak indicator; Tasks should have a rectangular checkbox indicator.
*   **Recurrence:** Tasks are single-instance. Do not allow "Repeat Daily" options on Tasks (this belongs in the Habit module).

---

## 6. Acceptance Criteria

*   **AC-01:** A user can successfully add a task with a title and future date. The task appears in the "Upcoming" section.
*   **AC-02:** A user can mark a task as complete. The checkbox changes state, the task moves to the "Completed" section (or fades out), and the timestamp is recorded.
*   **AC-03:** A task with a priority of "High" displays a red/orange accent color on the list item.
*   **AC-04:** If a task is not completed by its due date, it appears in the "Overdue" section the next day.
*   **AC-05:** A notification triggers 1 hour prior to the due date (device permissions allowed). Tapping the notification opens the Task Detail view.
*   **AC-06:** Deleting a task removes it from the list and syncs the deletion across other logged-in devices.
*   **AC-07:** Creating a task without internet saves locally and syncs automatically upon reconnection.

---

## 7. Out of Scope (MVP)
*   **Subtasks:** Tasks cannot contain nested checklist items.
*   **Collaboration:** Tasks are private. No sharing or assigning tasks to other users.
*   **Calendar View:** Tasks will only be viewed in list format, not a monthly calendar grid.
*   **Recurring Tasks:** Users cannot set "Every Friday" on a Task. (Must use Habits for recurring items).
*   **Files/Attachments:** Tasks cannot attach photos or documents (Notes handle this).
*   **Project Management:** No kanban boards, Gantt charts, or dependencies.
```

The model load time was slow and model thought for 59 seconds before generating answer.

### Let's Compare Local LLM PRDs to Cloud LLM MiniMax M2.5

I ran the same prompt through MiniMax M2.5 to compare outputs.

**MiniMax M2.5 PRD output:**

```markdown
# Product Requirement Document: Tasks Feature for Aily

**Version:** 1.0  
**Date:** March 11, 2026  
**Status:** Draft  
**Owner:** Product Team

---

## 1. Overview

Aily is a personal productivity mobile app designed to help individuals track and manage their daily lives through
Notes, Mood Tracker, and Habits features. This PRD outlines the requirements for adding a new **Tasks** functionality
that enables users to create, organize, prioritize, and complete personal daily tasks. The Tasks feature will integrate
seamlessly with the existing app ecosystem, providing a unified experience for users to manage both habits and
actionable to-do items. This feature addresses the core user need for explicit task management with due dates,
priorities, and completion tracking—capabilities that are distinct from the recurring habit tracking already present in
the app.

---

## 2. User Stories

| ID | User Story | Priority |
|----|------------|----------|
| US-1 | **Quick Task Creation** <br> As a user, I want to quickly add a new task with just a title so that I can capture tasks immediately without friction. | Must Have |
| US-2 | **Task Details & Due Dates** <br> As a user, I want to set a due date and time for tasks so that I can organize my workload and receive reminders. | Must Have |
| US-3 | **Task Completion** <br> As a user, I want to mark tasks as complete so that I can track my progress and feel accomplished. | Must Have |
| US-4 | **Task Organization** <br> As a user, I want to organize tasks into custom lists/categories so that I can group related tasks (e.g., Work, Personal, Shopping). | Should Have |
| US-5 | **Priority Levels** <br> As a user, I want to assign priority levels (High, Medium, Low) to tasks so that I can focus on what matters most. | Should Have |
| US-6 | **Task Filtering & Sorting** <br> As a user, I want to filter tasks by status (All, Active, Completed) and sort by due date or priority so that I can quickly find what I need. | Should Have |
| US-7 | **Task Editing & Deletion** <br> As a user, I want to edit or delete existing tasks so that I can update my task list as priorities change. | Must Have |

---

## 3. Functional Requirements

### 3.1 Task Management

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-1 | Create Task | Users shall be able to create a new task with a required title (min 1 character, max 200 characters). |
| FR-2 | Edit Task | Users shall be able to edit all task fields: title, description, due date, priority, and list assignment. |
| FR-3 | Delete Task | Users shall be able to delete a task with a confirmation dialog to prevent accidental deletion. |
| FR-4 | Complete Task | Users shall be able to mark a task as complete by tapping a checkbox. Completed tasks shall be visually distinguished (strikethrough, muted color). |
| FR-5 | Uncomplete Task | Users shall be able to mark a completed task as active again. |

### 3.2 Task Properties

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-6 | Title | Required field. String, 1-200 characters. |
| FR-7 | Description | Optional field. Multi-line text, max 1000 characters. |
| FR-8 | Due Date | Optional field. Date picker with time selection (hour:minute). |
| FR-9 field. Enum: None, | Priority | Optional Low, Medium, High. Default: None. |
| FR-10 | List Assignment | Optional field. Default: "Inbox" (uncategorized). Users can assign to custom lists. |
| FR-11 | Created At | Auto-generated timestamp when task is created. |
| FR-12 | Completed At | Auto-generated timestamp when task is marked complete. |

### 3.3 Task Lists

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-12 | Default List | System shall provide a default "Inbox" list for uncategorized tasks. |
| FR-13 | Create List | Users shall be able to create custom lists with a name (required) and optional color/icon. |
| FR-14 | Edit List | Users shall be able to rename lists and change their color/icon. |
| FR-15 | Delete List | Users shall be able to delete custom lists. Tasks in deleted list shall move to Inbox. |
| FR-16 | List Order | Users shall be able to reorder custom lists via drag-and-drop. |

### 3.4 Filtering & Sorting

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-17 | Filter by Status | Users shall be able to filter tasks by: All, Active, Completed. |
| FR-18 | Filter by List | Users shall be able to view tasks for a specific list. |
| FR-19 | Sort Tasks | Users shall be able to sort tasks by: Due Date (ascending/descending), Priority (High to Low), Created Date, Alphabetical. |
| FR-20 | Default View | Default sort shall be: Due Date (ascending), then Priority (High to Low). |

### 3.5 Notifications & Reminders

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-21 | Due Date Reminder | Users shall receive a local notification at the due date/time if the task is not completed. |
| FR-22 | Reminder Customization | Users shall be able to set reminder time: At due time, 15 min before, 1 hour before, 1 day before, Custom. |
| FR-23 | Notification Permission | App shall request notification permission on first task creation with a due date. |

### 3.6 Data Persistence

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-24 | Local Storage | All task data shall be stored locally using SQLite (or equivalent) via Flutter. |
| FR-25 | Offline Support | App shall work fully offline. No network dependency. |

### 3.7 Integration with Existing Features

| ID | Requirement | Description |
|----|-------------|-------------|
| FR-26 | Bottom Navigation | Tasks shall appear as a new tab in the bottom navigation bar (alongside Notes, Mood, Habits). |
| FR-27 | Consistent Styling | Tasks UI shall match the existing design language of Aily (colors, typography, spacing). |

---

## 4. Non-Functional Requirements

### 4.1 Performance

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR-1 | App Launch | Tasks screen shall load within 500ms on average device. |
| NFR-2 | Task Operations | Task create/edit/delete shall complete within 200ms. |
| NFR-3 | List Rendering | Rendering 100+ tasks shall maintain 60fps scroll performance. |

### 4.2 Usability

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR-4 | Accessibility | App shall support screen readers (TalkBack/VoiceOver) with proper labels. |
| NFR-5 | Touch Targets | All interactive elements shall have minimum 44x44 point touch target. |
| NFR-6 | Error Handling | All error states shall show user-friendly messages, not technical errors. |

### 4.3 Technical

| ID | Requirement | Description |
|----|-------------|-------------|
| NFR-7 | Minimum iOS Version | Support iOS 12.0+. |
| NFR-8 | Minimum Android Version | Support Android API 21+ (Android 5.0). |
| NFR-9 | State Management | Use BLoC pattern (consistent with existing app architecture, if applicable). |
| NFR-10 | Testing | Unit tests for business logic, widget tests for UI components. |

---

## 5. UI/UX Guidelines

### 5.1 Layout Structure

- **Navigation:** Bottom tab bar with 4 items: Notes, Mood, Habits, Tasks. Tasks icon: checkbox or checklist.
- **Tasks Screen:**
    - Top: App bar with title "Tasks" and filter/sort action buttons.
    - Below app bar: Horizontal chip row for quick list filtering (All, Inbox, custom lists).
    - Main content: Scrollable list of task items.
    - Floating Action Button (FAB): Purple primary color, "+" icon to create new task.
- **Task Detail Screen:** Full-screen modal or pushed page with form fields.

### 5.2 Visual Design

| Element | Specification |
|---------|---------------|
| Primary Color | Purple (#6750A4) – matches existing Aily theme |
| Surface Color | White (#FFFFFF) / Dark: #1C1B1F |
| Completed Task | Title with strikethrough, opacity 60%, gray checkmark |
| Priority Indicators | High: Red (#BA1A1A), Medium: Orange (#FF8B00), Low: Green (#386A20) |
| List Colors | User-selectable from palette: Red, Orange, Yellow, Green, Blue, Purple, Pink, Gray |
| Typography | Use existing Aily typography scale (Roboto on Android, SF Pro on iOS) |
| Spacing | 8pt grid system. Card padding: 16px. List item vertical padding: 12px. |

### 5.3 Component Specifications

**Task List Item:**

- Leading: Checkbox (circular, 24x24)
- Title: Body1 text, max 2 lines with ellipsis
- Subtitle (if applicable): Due date in caption style, colored if overdue
- Trailing: Priority indicator dot (if set)
- Swipe actions: Swipe left to delete, swipe right to complete

**Create/Edit Task Form:**

- Title TextField: Outlined, required, autofocus
- Description TextField: Outlined, multiline (3 lines visible), optional
- Due Date Field: Outlined, tap to open date picker, clear button
- Time Field: Outlined, tap to open time picker, optional
- Priority Selector: Segmented button row (None, Low, Medium, High)
- List Selector: Dropdown with list names + "Create new list" option
- Reminder Selector: Dropdown (None, At due time, 15 min before, 1 hour before, 1 day before, Custom)
- Save Button: Filled primary button at bottom
- Cancel: Back navigation or X button

**Empty States:**

- No tasks: Illustration + "No tasks yet" + "Tap + to add your first task" subtext
- No completed tasks: "No completed tasks" message when filtered by Completed

### 5.4 Interactions

- **Pull-to-refresh:** Not required (local data), but implement for future cloud sync readiness.
- **Swipe gestures:** Left swipe reveals delete (red), right swipe reveals complete (green).
- **Long press:** On task item shows context menu (Edit, Delete, Move to List).
- **FAB:** Tap opens Create Task screen. Long press shows "Quick add" with just title field.

### 5.5 Animations

- Task completion: Checkbox fills with checkmark, 200ms ease-out. Title strikethrough animates in.
- Task deletion: Slide out left, 250ms.
- Screen transitions: Standard Material/Cupertino page transitions.

---

## 6. Acceptance Criteria

### 6.1 Core Functionality

| ID | Criterion | Test Scenario |
|----|-----------|---------------|
| AC-1 | Create task with title | User taps FAB → enters title → taps Save → task appears in list |
| AC-2 | Create task with all fields | User creates task with title, description, due date, priority, list → all fields display correctly |
| AC-3 | Edit task | User taps task → modifies fields → saves → changes persist |
| AC-4 | Delete task | User swipes left on task → confirms deletion → task removed from list |
| AC-5 | Complete task | User taps checkbox → task shows completed state → moves to bottom of list |
| AC-6 | Uncomplete task | User taps checkbox on completed task → task returns to active state |
| AC-7 | Filter by status | User taps "Active" filter → only incomplete tasks shown. Taps "Completed" → only completed shown |
| AC-8 | Filter by list | User taps custom list chip → only tasks in that list shown |
| AC-9 | Sort tasks | User changes sort to "Priority" → tasks reorder accordingly |
| AC-10 | Create custom list | User goes to list management → creates "Work" list → list appears in filter chips |

### 6.2 Notifications

| ID | Criterion | Test Scenario |
|----|-----------|---------------|
| AC-11 | Reminder notification | User creates task with due date + reminder → notification fires at set time |
| AC-12 | Notification permission | First task with due date triggers permission request → user can allow/deny |

### 6.3 Data & Performance

| ID | Criterion | Test Scenario |
|----|-----------|---------------|
| AC-13 | Data persists | User creates task → closes app → reopens app → task still present |
| AC-14 | Large list performance | User creates 100 tasks → scrolls smoothly without jank |
| AC-15 | Works offline | User enables airplane mode → can create, edit, complete, delete tasks |

### 6.4 Visual Checkpoints

| ID | Checkpoint |
|----|------------|
| VC-1 | Tasks tab visible in bottom navigation with correct icon |
| VC-2 | FAB is purple and positioned bottom-right |
| VC-3 | Completed tasks show strikethrough and muted color |
| VC-4 | Priority dots show correct colors |
| VC-5 | Empty state displays when no tasks exist |
| VC-6 | Form fields follow Aily design system |

---

## 7. Out of Scope

The following features are explicitly **NOT** in scope for this initial release:

| Feature | Reason |
|---------|--------|
| Subtasks / Checklists | Complex task breakdown; defer to future iteration |
| Task Dependencies | Linking tasks (e.g., "Task B after Task A"); not core to MVP |
| Recurring Tasks | Already handled by Habits feature; duplicating would confuse users |
| Task Comments/Notes | Users can link to Notes feature via description reference |
| Cloud Sync / Cross-Device | Local-only for MVP; sync considered for v2 |
| Task Sharing / Collaboration | Personal productivity app; no sharing required |
| Smart Lists (e.g., "Today", "This Week") | Filter by date range can achieve similar results manually |
| Batch Operations (select multiple, bulk delete) | Single-task operations sufficient for MVP |
| Dark Mode Theming | Use system theme; custom dark mode styling deferred |
| Widget Support | Home screen widgets; consider in future if requested |
| Export Tasks (PDF, CSV) | Export functionality out of scope |
| Keyboard Shortcuts | Desktop/tablet-specific; not applicable to mobile-first |

---

## Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| FAB | Floating Action Button |
| Inbox | Default uncategorized task list |
| MVP | Minimum Viable Product |
| PRD | Product Requirement Document |

### B. Related Documents

- Aily App Architecture (for integration points)
- Aily Design System (for color palette, typography specs)
- Flutter Technical Specification (for state management approach)

### C. Assumptions

1. The existing codebase uses a state management solution (BLoC, Provider, Riverpod, or similar).
2. Local database is already in place or SQLite/Drift will be added.
3. Notification handling is supported via a Flutter package (flutter_local_notifications).
4. The app follows Material Design 3 on Android and adapts to iOS Human Interface Guidelines.

---

*End of PRD*
```

### Let's Use Gemini Pro to Rate Our PRDs

I fed both PRDs into Gemini Pro and asked it to rate them using a standard PRD evaluation rubric.

**Gemini's rating criteria:**

1. Clarity (1-5): Is the feature and its value proposition clear?
2. Completeness (1-5): Are all required sections present?
3. Actionability (1-5): Could a developer implement from this without additional clarification?
4. Testability (1-5): Are acceptance criteria specific enough to verify?
5. Consistency (1-5): Do requirements align with user stories?

I used this prompt:

```markdown
Compare these two Product Requirement Documents (PRD A and PRD B) for the same feature (Tasks functionality for a mobile
app).
Evaluate each PRD using these criteria:

1. **Clarity (1-5)**: Is the feature and its value proposition clear? Are terms unambiguous?
2. **Completeness (1-5)**: Does it include Overview, User Stories, Functional Requirements, Non-Functional Requirements,
   UI/UX Guidelines, Acceptance Criteria, and Out of Scope?
3. **Actionability (1-5)**: Could a developer implement this without additional clarification? Are requirements specific
   enough?
4. **Testability (1-5)**: Are acceptance criteria concrete and verifiable? Can QA test against them?
5. **Consistency (1-5)**: Do user stories align with functional requirements? Do acceptance criteria match requirements?
   For each criterion, give:

- A score (1-5)
- A brief explanation citing specific examples from each PRD

Then calculate totals and declare a winner with reasoning.
```

**This is Gemini's output:**

```markdown

```

I will add that both PRDs have mistakes in UI/UX descriptions as the model didn't have access to Aily projects, and it
made wrong assumptions for colors et. The PRD A is poorly formatted and ASCI Wireframes are can be easily broken by text
editors with formatting. Clearly we could pull out best parst from both to create the best template for PRD.

### Let's Create an Agent for PRD Creation

Based on the experiments above, I created an OpenCode skill that automates PRD creation. It uses the question tool to
gather requirements systematically, then generates a structured PRD.

**PRD Agent skill location:** `~/.config/opencode/skills/prd-creator/SKILL.md`

```markdown
# PRD Creator Skill

This skill helps you create Product Requirement Documents using local or cloud LLMs.

## Instructions

You are a Product Manager assistant. Your job is to guide the user through creating a well-structured PRD.

## Process

1. **Gather context** - Ask the user about their product/feature:
    - What are you building?
    - Who is it for?
    - Why does it matter?
    - What constraints exist (timeline, tech stack, team size)?

2. **Generate PRD** - Create a structured document with:
    - Overview
    - User Stories
    - Functional Requirements
    - Non-Functional Requirements
    - UI/UX Guidelines
    - Acceptance Criteria
    - Out of Scope

3. **Save output** - Write the PRD to the user's preferred location.

## Output Format

Use markdown. Be specific and actionable. Include concrete examples where possible.
```

> The skill uses OpenCode's question tool to iteratively gather requirements — the same pattern you'd use in a real
> product conversation.

### Let's Use the PRD Agent

[PLACEHOLDER: Demo output showing PRD Agent in action - questions asked and PRD generated]

## Key Points Recap

1. **PRDs force discipline** — The structured format catches scope ambiguity before code gets written
2. **Local LLMs generate usable PRDs** — Qwen3.5-35B-A3B produces 80% of the quality at 0% cost
3. **Cloud models may add polish** — MiniMax M2.5 showed slight improvements in completeness
4. **External rating reveals blind spots** — Gemini's evaluation caught gaps I missed in my review
5. **Agents make it repeatable** — The PRD Creator skill turns one-off generation into a reusable workflow
