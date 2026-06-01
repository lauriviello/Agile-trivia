Generate the Focus Area Report HTML for the Agile Delivery Leads (ADLs) Jira project. Arguments: $ARGUMENTS

## Context

- **Jira site:** `foshtech.atlassian.net`
- **Jira project:** `AGILE` (Agile Delivery Leads – ADLs, ID: 10866)
- **Assignee:** current authenticated user (`currentUser()`)
- **Issue type:** Task
- **Focus Area field:** `customfield_11107` (select – single option)
- **Quarter field:** `customfield_11173` (select – single option)
- **Output file:** `/Users/lriviello/Documents/Projects/ADL-Dashboard/focus-area-report.html`

## Arguments

- `--help` — show usage and stop
- `--quarter=Q1|Q2|Q3|Q4` — pre-filter tasks to a specific quarter (default: all)
- `--open` — open the HTML in the browser after generating (default: true)

## Steps

### Step 0 — Handle --help / no-arg

If `$ARGUMENTS` contains `--help`, print this card and **stop**:

```
/focus-area-report — Focus Area × Quarter report for AGILE project

FLAGS
  --quarter=Q1|Q2|Q3|Q4   Filter to a specific quarter (default: all)
  --open                   Open the report in the browser after generating (default: true)

EXAMPLES
  /focus-area-report
  /focus-area-report --quarter=Q2
```

### Step 1 — Fetch tasks from Jira

Use the `mcp__claude_ai_Atlassian_Rovo__searchJiraIssuesUsingJql` tool with:

```
cloudId: foshtech.atlassian.net
jql: project = AGILE AND issuetype = Task AND assignee = currentUser() ORDER BY created DESC
fields: ["summary", "status", "customfield_11107", "customfield_11173", "duedate", "priority"]
maxResults: 100
responseContentFormat: markdown
```

From each issue extract:
- `key` — e.g. `AGILE-135`
- `fields.summary`
- `fields.status.name`
- `fields.customfield_11107.value` → Focus Area (null if not set)
- `fields.customfield_11173.value` → Quarter (null if not set)

If `--quarter=QN` was passed, keep only tasks where Quarter equals QN.

### Step 2 — Build statistics

Compute from the fetched tasks:
- `total` — all tasks
- `done` — status = "Done"
- `inProgress` — status = "In Progress"
- `toDo` — status = "To Do"
- `focusAreaCount` — count of distinct non-null Focus Areas

### Step 3 — Build chart data

Group tasks by Focus Area and Quarter. Produce a dataset for Chart.js:
- X axis labels: Q1, Q2, Q3, Q4, "—" (always show all five, even if count is 0)
- One dataset per Focus Area with counts per quarter bucket

Focus Area colour palette:
```
Agile Coach              → #58a6ff
Coaching & Development   → #bc8cff
Cross-BU                 → #3fb950
Improvement Plans        → #f78166
Innovation Ally          → #ffa657
Upskilling & Workshops   → #56d364
```
Any other Focus Area not listed → `#e6edf3`

### Step 4 — Generate HTML

Write the full standalone HTML to `/Users/lriviello/Documents/Projects/ADL-Dashboard/focus-area-report.html`.

The report must include:
1. **Header** — title "Focus Area Report", subtitle "Laura Riviello · Agile Delivery Leads (ADLs) · foshtech.atlassian.net", generation date (today).
2. **Summary stat cards** — Total Tasks, Done, In Progress, To Do, Focus Areas.
3. **Chart.js grouped bar chart** — quarters on X axis, one coloured bar series per Focus Area. Clicking a bar filters the task list to that quarter.
4. **Quarter filter tabs** — "All", Q1, Q2, Q3, Q4, "—", and "Sin clasificar". Active tab highlighted. The "Sin clasificar" tab shows a red count badge with the number of unclassified tasks (tasks where both Quarter and Focus Area are null). When that filter is active and there are no unclassified tasks, show: `"No hay tasks sin Quarter ni Focus Area. ¡Todo clasificado!"`
5. **Focus Area groups** — collapsible sections, one per Focus Area, ordered by count desc. Each group shows a coloured dot, the Focus Area name, and task count badge. Inside: a table with columns Key (linked to `https://foshtech.atlassian.net/browse/<KEY>`), Summary, Quarter, Status (coloured badge).
6. **"Sin Focus Area" group** — tasks with null Focus Area, shown last.
7. **Empty state** — if no tasks exist for the selected quarter, show: `"Aún no se encuentran datos para este Quarter."` centered in muted text.

Dark theme CSS variables (use the same palette as the existing ADL dashboard):
```css
--bg: #0d1117;  --surface: #161b22;  --surface-2: #1c2128;
--border: #30363d;  --text: #e6edf3;  --muted: #8b949e;
--primary: #58a6ff;  --radius: 6px;
```

Status badge colours:
```
Done        → bg #0d2219  text #3fb950
In Progress → bg #1c2a3a  text #58a6ff
To Do       → bg #21262d  text #8b949e
In Review   → bg #261d34  text #bc8cff
Cancelled   → bg #2d1a1a  text #f78166
```

Chart.js must be loaded from CDN: `https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js`

### Step 5 — Open in browser

Unless `--open=false` was passed, run:

```bash
open /Users/lriviello/Documents/Projects/ADL-Dashboard/focus-area-report.html
```

### Step 6 — Confirm

Report back:
- Total tasks fetched
- Breakdown by Focus Area (name + count)
- Output file path
- Any tasks missing Focus Area or Quarter (count only)
