# Product Requirements Document (PRD) - TODO App Upgrade

## 1. Overview

Upgrade the existing TODO app (currently title + completed) to improve task organization while keeping the solution simple and teachable. The MVP will include due dates, priorities, and filters, with no backend changes and local storage only.

---

## 2. MVP Scope

- Add `dueDate` field as optional ISO date format `YYYY-MM-DD`.
- Add `priority` field as enum `P1 | P2 | P3`.
- Default `priority` to `P3` when not provided.
- Require `title` for each task.
- Support filters: `All`, `Today`, `Overdue`.
- Keep storage local only (no backend / external storage).
- For `dueDate`, invalid values are ignored and treated as absent.

---

## 3. Post-MVP Scope

- Visually highlight overdue tasks.
- Add sorting with this order: overdue first, then priority (`P1` to `P3`), then due date ascending, then tasks without due date last.

---

## 4. Out of Scope

- Notifications.
- Recurring tasks.
- Multi-user support.
- Keyboard navigation.
- External storage.
