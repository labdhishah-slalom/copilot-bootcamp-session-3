## MVP

- Epic: Task Data Model
  - Story: Require task title
    - Acceptance Criteria:
      - A task cannot be created without a title.
      - A task cannot be updated without a title.
      - If the title is missing, the task is not saved.
    - Technical Requirements:
      - The frontend already blocks empty titles in `packages/frontend/src/TaskForm.js`; this validation must remain part of the create and edit flows.
      - The current backend already rejects empty titles in `packages/backend/src/app.js` for `POST /api/tasks` and `PUT /api/tasks/:id`; if the MVP is moved to local-only storage, equivalent title validation must exist in the frontend save path before writing to storage.
      - Frontend tests in `packages/frontend/src/__tests__/App.test.js` should cover the no-title save rejection behavior.

  - Story: Add optional due date field
    - Acceptance Criteria:
      - A task may be saved with no due date.
      - A task may be saved with a due date in `YYYY-MM-DD` format.
      - A saved due date is shown when the task is displayed.
    - Technical Requirements:
      - The frontend already exposes a due date input in `packages/frontend/src/TaskForm.js` and renders a due date chip in `packages/frontend/src/TaskList.js`; these components should continue to own due date entry and display.
      - The current code uses `due_date` in the UI and backend schema (`packages/frontend/src/TaskForm.js`, `packages/frontend/src/TaskList.js`, `packages/backend/src/app.js`), while the PRD names the field `dueDate`; implementation should choose one mapping and keep it consistent across storage, UI state, and tests.
      - Tests should verify saving tasks both with and without a due date.

  - Story: Add priority field with P1 P2 and P3
    - Acceptance Criteria:
      - A task includes a priority value.
      - Priority supports only `P1`, `P2`, or `P3`.
      - The selected priority is preserved when the task is saved and shown when the task is displayed.
    - Technical Requirements:
      - The current frontend form in `packages/frontend/src/TaskForm.js` has no priority input and must be extended to capture `P1`, `P2`, and `P3`.
      - The current task list in `packages/frontend/src/TaskList.js` has no priority display and must be extended to render the saved priority value.
      - The current backend schema in `packages/backend/src/app.js` has no priority column; if backend endpoints continue to be used outside MVP, they must be updated to persist and return priority values consistently.
      - Frontend and backend tests should assert that only `P1`, `P2`, and `P3` are accepted and returned.

  - Story: Default priority to P3
    - Acceptance Criteria:
      - When a task is created without an explicit priority, the saved priority is `P3`.
      - When a task is edited and no new priority is selected, the existing priority remains unchanged.
    - Technical Requirements:
      - `packages/frontend/src/TaskForm.js` should initialize new tasks with `P3` so the default is explicit in the UI state.
      - If storage normalization is centralized, the same default should also be enforced at the task save boundary so saved data does not omit priority.
      - Tests should verify the default priority path for task creation.

  - Story: Ignore invalid due date values
    - Acceptance Criteria:
      - If a due date value is not a valid ISO `YYYY-MM-DD` date, it is treated as absent.
      - An invalid due date does not block the task from being saved.
      - A task saved with an invalid due date is stored without a due date.
    - Technical Requirements:
      - The current frontend helper `normalizeDateString` in `packages/frontend/src/TaskForm.js` does not implement the PRD rule to ignore invalid values; save logic must validate the final date string and clear invalid values instead of persisting them.
      - The backend currently accepts `due_date` without ISO validation in `packages/backend/src/app.js`; if backend routes are used, they must normalize invalid values to `null` rather than storing them unchanged.
      - Tests should cover invalid date input on create and update.

- Epic: Task Filtering
  - Story: Show All tasks view
    - Acceptance Criteria:
      - The user can select an `All` view.
      - The `All` view shows the full task list.
    - Technical Requirements:
      - `packages/frontend/src/App.js` and `packages/frontend/src/TaskList.js` currently have no filter state or filter controls; they need a filter selection mechanism and a task list view driven by that state.
      - The current backend `GET /api/tasks` in `packages/backend/src/app.js` only supports `completed` and `search` query parameters and does not provide `All`, `Today`, or `Overdue` filtering.
      - Frontend tests should assert that the `All` view renders the unfiltered list.

  - Story: Show Today tasks view
    - Acceptance Criteria:
      - The user can select a `Today` view.
      - The `Today` view shows only tasks whose due date is today.
    - Technical Requirements:
      - The frontend must add a `Today` filter control and apply date-based filtering against the saved due date values.
      - Date filtering logic should live in one place so `TaskList` receives a filtered dataset rather than duplicating date rules across components.
      - Tests should verify that only tasks due today are shown in the `Today` view.

  - Story: Show Overdue tasks view
    - Acceptance Criteria:
      - The user can select an `Overdue` view.
      - The `Overdue` view shows only tasks whose due date is earlier than today.
    - Technical Requirements:
      - The frontend must add an `Overdue` filter control and compare due dates against the current date using the stored `YYYY-MM-DD` values.
      - Tasks without a due date must not appear in the `Overdue` view because they are not overdue by the PRD rule.
      - Tests should verify that only overdue tasks are shown in the `Overdue` view.

- Epic: Local Task Storage
  - Story: Store tasks in local storage
    - Acceptance Criteria:
      - Tasks persist in browser local storage.
      - Saved tasks remain available after the page is reloaded in the same browser.
    - Technical Requirements:
      - The current frontend in `packages/frontend/src/App.js` and `packages/frontend/src/TaskList.js` uses `fetch` calls to `/api/tasks`; this must be replaced or abstracted for MVP so task reads and writes use browser local storage.
      - The current backend in `packages/backend/src/app.js` uses an in-memory SQLite database, which does not satisfy the PRD storage requirement and should not be required for MVP persistence.
      - Frontend tests should mock and verify local storage reads and writes rather than API responses for the MVP flow.

  - Story: Keep task data local only
    - Acceptance Criteria:
      - The MVP does not require backend storage.
      - The MVP does not require external storage.
    - Technical Requirements:
      - The current network dependency in `packages/frontend/src/App.js` and `packages/frontend/src/TaskList.js` should be removed from the MVP task flow.
      - No new backend endpoints or external persistence integrations should be added for MVP.
      - Existing backend code in `packages/backend/src/app.js` can remain in the repository, but MVP requirements should be satisfied without depending on it.

## Post-MVP

- Epic: Overdue Task Visibility
  - Story: Highlight overdue tasks visually
    - Acceptance Criteria:
      - A task that is overdue has a distinct visual treatment.
      - A task that is not overdue does not use the overdue visual treatment.
    - Technical Requirements:
      - `packages/frontend/src/TaskList.js` already computes display styling per task row and is the right place to add overdue-specific visual treatment.
      - Overdue detection should reuse the same date comparison rule as the `Overdue` filter so UI behavior stays consistent.
      - Frontend tests should verify that overdue tasks render with the distinct visual treatment.

- Epic: Task Sorting
  - Story: Sort overdue tasks first
    - Acceptance Criteria:
      - When task sorting is applied, overdue tasks appear before non-overdue tasks.
    - Technical Requirements:
      - The current backend default ordering in `packages/backend/src/app.js` is `due_date IS NULL, due_date ASC, created_at ASC`, which does not match the PRD post-MVP order.
      - Sorting should be implemented in one place, either in the frontend task preparation flow or in the backend query layer, so the rule is deterministic and testable.
      - Tests should verify overdue-first ordering.

  - Story: Sort tasks by priority
    - Acceptance Criteria:
      - Within the applicable task group, tasks are ordered by priority from `P1` to `P3`.
    - Technical Requirements:
      - Priority ordering cannot be alphabetical; implementation must explicitly define `P1` before `P2` before `P3`.
      - The sorting layer must have access to normalized priority values for every task.
      - Tests should verify `P1`, `P2`, `P3` ordering.

  - Story: Sort tasks by due date ascending
    - Acceptance Criteria:
      - Within the applicable task group, tasks with due dates are ordered from earliest to latest due date.
    - Technical Requirements:
      - The current backend already sorts by `due_date ASC` in `packages/backend/src/app.js`; if sorting remains backend-driven, that rule must be composed with the PRD's overdue and priority ordering.
      - If sorting is frontend-driven, date comparison should use the stored `YYYY-MM-DD` values consistently.
      - Tests should verify ascending due date order.

  - Story: Place tasks without due date last
    - Acceptance Criteria:
      - Tasks without a due date appear after tasks with a due date.
    - Technical Requirements:
      - The current backend query in `packages/backend/src/app.js` already uses `due_date IS NULL` to place undated tasks after dated tasks; if the sorting logic moves to the frontend, the same rule must be preserved.
      - Sorting tests should explicitly cover tasks with missing due dates.