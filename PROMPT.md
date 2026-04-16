# Todo App Prompt

Build a todo app as a single `index.html` file using plain HTML, CSS, and JavaScript — no frameworks, no dependencies.

---

## Design

- Minimal, mobile-first design using system fonts and a light neutral background
- Large heading: **"You've got this."** with a dynamic stats subtitle
- All copy throughout the app should be encouraging and aspirational
- **Dark mode toggle** — fixed top-right; sun/moon icons flanking a small toggle switch. Defaults to system preference; persists to localStorage (`cc_theme`). Smooth CSS transitions on all surfaces when switching.

---

## Add Form

A card-style input area. The top row is always visible; options are hidden behind collapsible toggles arranged as **horizontal pill buttons** in a row below the text input. Each toggle has a chevron that rotates when open. All panels collapse automatically after a task is added.

**Always visible:**
- Full-width text field — placeholder *"What would you like to get done today?"*
- **Add** button — disabled when the field is empty; submits on Enter

**Option toggles (collapsed by default, arranged horizontally):**

1. **Priority** — three pills: **High** (red), **Medium** (neutral, pre-selected), **Low** (blue). Exactly one is always selected. Resets to Medium after adding.

2. **Category** — multi-select pill buttons. Four defaults: **Work** (blue), **Personal** (purple), **Ideas** (amber), **Errands** (green). Multiple pills can be selected simultaneously. A dashed **+ New** pill opens an inline text input to create a custom category (Enter to confirm, Escape to cancel). Custom categories persist to localStorage and receive colours from a rotating palette. Resets to none after adding.

3. **Deadline** — date picker + time picker side by side. Both optional. If a date is set but no time is chosen, default to **10:00**. Resets after adding.

4. **Repeat** — pills: **None** (default), **Daily**, **Weekly**, **Fortnightly**, **Monthly**, **Days of week**, **Custom**. Selecting any non-None option auto-expands the Deadline panel (a due date is required). Resets to None after adding.
   - **Days of week**: shows 7 circular day-toggle buttons (S M T W T F S). Multi-select.
   - **Custom**: shows an "Every [N] [days / weeks / months]" row with a number input and unit pills.

---

## Focus Card

A full-width accent card rendered above the task list whenever there is at least one active task. Shows:
- Small eyebrow: *"Focus mode"*
- A rotating motivational title (one of 7 phrases, selected by `id % 7`)
- The top-priority task's text in large bold type
- Chips for priority (if not medium), categories, due date, and repeat cadence

---

## Task Items

Each task displays:

- **Circular checkbox** — checking it completes the task (see Repeat behaviour below)
- **Task text** — clicking it enters inline edit mode (active tasks only)
- **Meta row** below the text (only rendered if any content exists):
  - **Priority badge** — shown for High (red) and Low (blue) only. Clicking filters the list by that priority (toggle).
  - **Category tags** — one coloured pill per category. Faded/unstyled when done. Clicking filters the list (toggle).
  - **Due date chip** — textual deadline (see Time Format); colour-coded by urgency. Hidden on completed tasks.
  - **Repeat chip** — indigo pill showing cadence (e.g. *"↻ Weekly"*, *"↻ Mon, Wed, Fri"*, *"↻ Every 2 weeks"*). Shown on active repeating tasks only.
  - **Completed chip** — shown on done tasks. Regular tasks: *"Completed today"* / *"Completed yesterday"* / *"Completed Monday"* / *"Completed 14 Apr"*. Repeat log entries: *"Completed once"* / *"Completed 3×"*.
- **Edit button** (pencil icon) — fades in on hover; active tasks only. Clicking it while the item is already in edit mode **saves and closes** the editor.
- **Delete button** (trash icon) — fades in on hover

**Priority left-border accent** — active tasks get a thin coloured left border: red for High, blue for Low, none for Medium.

### Time Format

| Condition | Example |
|---|---|
| Overdue by 1 day | `Overdue · yesterday` |
| Overdue by N days | `Overdue · 3 days ago` |
| Due today | `Today at 3pm` |
| Due tomorrow | `Tomorrow at 10am` |
| Due within 7 days | `Thursday at 2pm` |
| Due further out | `14 May at 10am` |

Times: `3pm`, `10am`, `2:30pm` — omit `:00`. Urgency colouring: overdue → red, due within 3 hours → amber, all other → green.

---

## Repeatable Tasks

When a task has a `repeat` value set:

- **Completing it** does not mark it done. Instead:
  - The active item's `dueDate` advances to the next occurrence (calculated from today, not the original due date).
  - A single **completed log entry** is upserted: if one already exists (`repeatGroupId === id`) its `completionCount` and `completedAt` are updated; otherwise a new done entry is created with `completionCount: 1`.
- The active item remains in the list with its updated due date and a repeat chip.
- The log entry appears in the Accomplished section showing how many times the task has been completed.

**Next-date calculation** (from today):
| Repeat | Advance |
|---|---|
| Daily | +1 day; nearest daily occurrence to the new task's time is always within 12 h |
| Weekly | +7 days |
| Fortnightly | +14 days |
| Monthly | +1 month |
| Days of week | Walk forward day by day until hitting a selected weekday (max 7 iterations) |
| Custom (N days) | +N days |
| Custom (N weeks) | +N×7 days |
| Custom (N months) | +N months |

---

## Overload Warning

When adding a timed task, check all active (undone) tasks for due times within a **±12-hour window** (24 h total, centred on the new task's time). Special case: **daily repeating items** compute the nearest occurrence to the new task's time before comparing (they always fall within the window).

If **4 or more** conflicts are found, show a bottom-sheet modal instead of adding:

- Drag handle, title *"That's a lot for one stretch."*, subtitle with conflict count
- List of conflicting tasks, each showing the task name, due time, and two icon buttons:
  - **Edit (pencil)** — closes the modal, scrolls the item into view in the main list, opens its inline editor. The add form stays populated for retry.
  - **Delete (trash)** — removes the item immediately, updates the conflict count in the subtitle.
- **Let me reconsider** button — closes the modal, add form stays populated
- **Add anyway** button — proceeds with the add
- Tapping the backdrop also closes the modal

---

## Inline Editing

Clicking the task text or edit button transforms the item in-place:

- Label → text input (pre-filled)
- Meta row → replaced with:
  - Date + time inputs (pre-filled)
  - Priority picker (pre-selected)
  - Category picker (multi-select, pre-selected)
  - Repeat picker (pre-selected, with Days-of-week or Custom sub-row if applicable)
- **Enter**, **clicking the edit button again**, or **clicking away** → saves
- **Escape** → reverts
- Empty text field on save → reverts instead of saving

---

## Sort & Filter Bar

A collapsible **"Sort & filter"** panel sits between the add form and the task list. **Collapsed by default.** When any filters are active, a small badge on the toggle shows the count.

Clicking a priority badge or category tag on any task applies that filter and opens the panel automatically.

**Sort** (one active at a time, default: Date):
- **Date** — soonest first, then by priority for ties; undated tasks sort by priority only
- **Priority** — High → Medium → Low, then by date within each tier
- **Category** — alphabetical by first category, then priority, then date; uncategorised tasks go last

**Filter by priority** — All · High · Medium · Low (toggle; each pill uses its own colour when active)

**Filter by category** — dropdown; highlights when a value is selected

**× Clear filters** — appears when any filter is active

---

## Behaviour

- Active tasks at top (sorted per above); completed tasks at bottom under *"Accomplished"* label
- **Swipe left** on mobile to delete — threshold ~80px; red *"Let it go"* reveal underneath
- All state persists to localStorage: todos (`cc_todos_v2`) and custom categories (`cc_custom_cats`)
- On load, migrate todos that have a single `category` string → `categories` array, and add missing repeat fields with safe defaults
- Seed one starter task *"Swipe left to delete"* on first load

---

## Animations

- **New task** — slides in from above with spring easing
- **Completing / deleting** — fades down and out before re-render
- **Overload modal** — backdrop fades in; card slides up from bottom

---

## Data Model (per todo)

```
id               number      timestamp used as unique key
text             string
done             boolean
priority         string      'high' | 'medium' | 'low'
categories       string[]    array of category names
dueDate          string|null 'YYYY-MM-DD'
dueTime          string|null 'HH:MM'
completedAt      number|null timestamp
repeat           string|null 'daily'|'weekly'|'fortnightly'|'monthly'|'weekdays'|'custom'|null
repeatInterval   number      interval value for 'custom' (default 2)
repeatUnit       string      'days'|'weeks'|'months' (default 'weeks')
repeatDays       number[]    selected weekday indices 0–6 for 'weekdays' mode
repeatCount      number      how many times this active item has been completed
repeatGroupId    number|null on a done log entry: id of the originating active task
completionCount  number|null on a done log entry: how many times completed
```

---

## Copy

| Context | Text |
|---|---|
| Page title | *Make it happen.* |
| Heading | *You've got this.* |
| Subtitle — empty | *Every great day starts with a clear intention.* |
| Subtitle — tasks remaining (none done) | *N tasks waiting for you. You've got this.* |
| Subtitle — some done | *N of M tasks done. Keep it up!* |
| Subtitle — all complete | *All N tasks done. Well done today!* |
| Empty state heading | *All clear!* |
| Empty state body | *Add something you'd like to get done today.* |
| Filter empty state heading | *No matches.* |
| Filter empty state body | *Try adjusting your filters.* |
| Accomplished section label | *Accomplished* |
| Swipe-to-delete reveal | *Let it go* |
| Focus card eyebrow | *Focus mode* |
| Overload modal title | *That's a lot for one stretch.* |
