# Clear — Data Structure (MVP)

Mobile-first structured checklist & workspace app.

Target stack:
- MongoDB + Mongoose
- GraphQL (Pothos)
- Expo mobile app
- GCP backend services

Design goals:
- Simple MVP
- Single-user first, multi-user ready
- Checklist-first
- Workspace-centered
- Routine streak tracking
- Optional structured tables per workspace

---

# Core Rules

- Every document includes:
  - userId
  - createdAt
  - updatedAt
- Use ObjectId references between collections
- Keep reads optimized for:
  - Today dashboard
  - Workspace view
  - Routine streaks

---

# User

## users

Represents an app user.

Fields:

- _id: ObjectId
- email?: string
- name?: string
- timezone: string
- createdAt: Date
- updatedAt: Date
- deletedAt?: Date

Notes:
- Even if MVP has no full auth, keep user model.
- Timezone is used for Today and Routine dates.

---

# Workspace

## workspaces

Container for tasks, routines, and optional tables.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id

- name: string
- icon: string
- color?: string

- sortOrder: number
- archivedAt?: Date

- createdAt: Date
- updatedAt: Date
- deletedAt?: Date

Examples:
- Work
- Trading
- Learning
- Personal

---

# Tasks (Checklist Tasks)

## tasks

Checklist-based tasks inside a workspace.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id
- workspaceId: ObjectId → workspaces._id

- title: string
- notes?: string

- status: "todo" | "doing" | "done" | "archived"
- priority: 1 | 2 | 3

- dueAt?: Date

- reminder:
  - enabled: boolean
  - remindAt?: Date
  - repeat?: "none" | "daily" | "weekly"

- subtasks: array of:
  - _id: ObjectId
  - title: string
  - sortOrder: number
  - doneAt?: Date

- completedAt?: Date

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date


Design:
- Subtasks are embedded for simpler mobile reads.

---

# Routine Templates

## routineTemplates

Reusable checklist templates.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id
- workspaceId: ObjectId → workspaces._id

- name: string

- items: array of:
  - title: string
  - sortOrder: number

- schedule:
  - type: "none" | "daily" | "weekly"
  - daysOfWeek?: number[]   # 0–6

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date

Examples:
- Morning routine
- Pre-trade checklist
- Deploy checklist

---

# Routine Instances (Streak Model — Option B)

## routineInstances

Daily instances generated from templates.
Used for streak tracking and completion history.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id
- workspaceId: ObjectId → workspaces._id
- templateId: ObjectId → routineTemplates._id

- date: string  # YYYY-MM-DD in user timezone

- items: array of:
  - title: string
  - doneAt?: Date

- completedAt?: Date

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date

Rules:
- One instance per template per date per user.
- Unique index recommended on (userId, templateId, date).
- Streak = consecutive days with completedAt set.

---

# Workspace Tables (Optional Feature)

Flexible structured journal / tracking tables per workspace.

Use cases:
- Trading journal
- Dev logs
- Fitness tracker
- Study tracker

---

## tables

Table definition.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id
- workspaceId: ObjectId → workspaces._id

- name: string

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date


---

## tableColumns

Column definitions per table.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id
- tableId: ObjectId → tables._id

- name: string

- type:
  - text
  - number
  - date
  - select
  - boolean
  - tags

- options?: string[]   # for select/tags

- required?: boolean
- sortOrder: number

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date


---

## tableRows

Row records per table.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id
- tableId: ObjectId → tables._id

- cells: object map
  - key = columnId
  - value = any (typed by column)

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date

Example cells:

{
  "col_symbol": "XAUUSD",
  "col_direction": "LONG",
  "col_profit": 120,
  "col_mistake": true
}

Design:
- Flexible schema
- No migrations when columns change.

---

# Notifications

Push reminder scheduling.

---

## deviceTokens

Device push tokens.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id

- expoPushToken: string
- platform: "ios" | "android"

- createdAt: Date
- updatedAt: Date

- deletedAt?: Date


---

## notificationSchedules

Scheduled notification jobs.

Fields:

- _id: ObjectId
- userId: ObjectId → users._id

- targetType: "task" | "routineInstance"
- targetId: ObjectId

- fireAt: Date

- status:
  - scheduled
  - sent
  - cancelled
  - failed

- attempts: number
- lastError?: string

- createdAt: Date
- updatedAt: Date


- deletedAt?: Date

Used with:
- Cloud Tasks / background workers
- Push notification sender

---

# Recommended Indexes

tasks:
- (userId, status, dueAt)
- (userId, workspaceId, status)

routineInstances:
- UNIQUE (userId, templateId, date)

tableRows:
- (userId, tableId, createdAt)

notificationSchedules:
- (userId, status, fireAt)

---

# Derived (Computed) Data — Not Stored

Today Tasks:
- tasks where status != done AND dueAt <= endOfToday

Routine Streak:
- count consecutive routineInstances.completedAt days

Workspace Progress:
- completed tasks / total tasks

---

# MVP Limits (Intentional)

To prevent overengineering:

- Max ~12 columns per table
- No formulas in tables (v1)
- No collaboration (v1)
- No offline sync (v1)
- No nested workspaces (v1)

Keep it simple. Ship first.
