# Todo App Prompt

Build a todo app as a single `index.html` file using plain HTML, CSS, and JavaScript — no frameworks, no dependencies.

---

## Design

- Minimal, mobile-first design using system fonts and a light neutral background
- Large heading: **"Make it happen."** with a dynamic aspirational subtitle (see Copy)
- All copy throughout the app should be aspirational and motivating

---

## Add Form

A card-style input area. The top row is always visible; everything else is hidden behind a collapsible **"Deadline, priority & category"** toggle (chevron rotates when open, collapses automatically after a task is added).

**Always visible:**
- Full-width text field — placeholder *"What do you want to accomplish today?"*
- **Add** button — disabled when the field is empty; submits on Enter

**Inside the collapsible (collapsed by default):**

1. **Priority picker** — three pills: **High** (red), **Medium** (neutral, pre-selected), **Low** (blue). Exactly one is always selected. Resets to Medium after adding.

2. **Category picker** — multi-select pill buttons. Four defaults: **Work** (blue), **Personal** (purple), **Ideas** (amber), **Errands** (green). Multiple pills can be selected simultaneously; clicking a selected pill deselects it. A dashed **+ New** pill opens an inline text input to create a custom category (Enter to confirm, Escape to cancel). Custom categories persist to localStorage and receive colours from a rotating palette. Resets to none selected after adding.

3. **Deadline row** — date picker + time picker side by side. Both optional. If a date is set but no time is chosen, default to **10:00 am**. Resets after adding.

---

## Task Items

Each task displays:

- **Circular checkbox** — checking it completes the task
- **Task text** — clicking it enters inline edit mode (active tasks only)
- **Meta row** below the text (only rendered if any content exists):
  - **Priority badge** — shown for High (red) and Low (blue) only; Medium is the default and shows nothing. Clicking the badge filters the list by that priority (toggle — click again to clear).
  - **Category tags** — one coloured pill per category; multiple tags if multiple categories are set. Faded/unstyled when done. Clicking a tag filters the list by that category (toggle).
  - **Due date chip** — textual deadline (see Time Format); colour-coded by urgency. Hidden on completed tasks.
  - **Completed date chip** — shown on completed tasks instead of the due chip (e.g. *"Completed today"*, *"Completed yesterday"*, *"Completed Monday"*, *"Completed 14 Apr"*).
- **Edit button** (pencil icon) — fades in on hover; active tasks only
- **Delete button** (trash icon) — fades in on hover

**Priority left-border accent** — active (non-done) tasks get a thin coloured left border: red for High, blue for Low, none for Medium.

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

## Inline Editing

Clicking the task text or edit button transforms the item in-place:

- Label → text input (pre-filled)
- Meta row → replaced with:
  - Date + time inputs (pre-filled)
  - Priority picker (pre-selected)
  - Category picker (multi-select, pre-selected)
- **Enter** or **clicking away** → saves
- **Escape** → reverts
- Empty text field on save → reverts instead of saving

---

## Sort & Filter Bar

A collapsible **"Sort & filter"** panel sits between the add form and the task list. **Collapsed by default.** When any filters are active, a small badge on the toggle shows the count (sort selection does not count).

Clicking a priority badge or category tag on any task item also applies that filter and opens the panel automatically.

**Sort** (one active at a time, default: Date):
- **Date** — soonest first, then by priority for ties; undated tasks sort by priority only
- **Priority** — High → Medium → Low, then by date within each tier
- **Category** — alphabetical by first category, then priority, then date; uncategorised tasks go last

**Filter by priority** — All · High · Medium · Low (toggle; each pill uses its own colour when active)

**Filter by category** — dropdown showing all categories; highlights when a value is selected

**× Clear filters** — appears when any filter is active; resets both filters

"No matches. Try adjusting your filters." empty state when filters exclude everything.

---

## Behaviour

- Active tasks at top (sorted per above); completed tasks at bottom under *"Accomplished"* label with strikethrough and faded styling
- **Swipe left** on mobile to delete — threshold ~80px; red *"Let it go"* reveal underneath
- Completed tasks record a `completedAt` timestamp; uncompleting clears it
- All state persists to localStorage: todos (`cc_todos_v2`) and custom categories (`cc_custom_cats`)
- Migrate old todos that have a single `category` string to the new `categories` array format on load
- Seed one starter task `"Swipe left to delete"` on first load (no category, no due date, Medium priority)

---

## Animations

- **New task** — slides in from above with spring easing at the top of the active list
- **Completing / deleting** — fades down and out before the list re-renders

---

## Data Model (per todo)

```
id           number    timestamp used as unique key
text         string
done         boolean
priority     string    'high' | 'medium' | 'low'
categories   string[]  array of category names
dueDate      string|null  'YYYY-MM-DD'
dueTime      string|null  'HH:MM'
completedAt  number|null  timestamp
```

---

## Copy

| Context | Text |
|---|---|
| Subtitle — empty | *Every great day starts with a clear intention.* |
| Subtitle — tasks remaining (none done) | *3 goals ready to conquer.* |
| Subtitle — tasks remaining (some done) | *2 of 5 goals achieved. Keep going.* |
| Subtitle — all complete | *All 5 goals crushed. You're on fire.* |
| Empty state heading | *The slate is clean.* |
| Empty state body | *What will you achieve today?* |
| Filter empty state heading | *No matches.* |
| Filter empty state body | *Try adjusting your filters.* |
| Accomplished section label | *Accomplished* |
| Swipe-to-delete reveal | *Let it go* |
