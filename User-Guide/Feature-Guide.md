# Feature Guide

## Complete Feature Reference

This guide describes every screen and control in Quality Scorecard for Jira as shipped. Labels in **bold** match the text you see in the app.

---

## Evaluation panel (issue view)

Found in the **Quality Scorecard for Jira** panel on any issue of an enabled project.

### Evaluation form
- One slider per criterion, whole numbers **0–10**. Sliders start at the criterion's default (normally 5) or, if the ticket already has an evaluation, at the latest evaluation's values.
- **Feedback** text area (optional).
- **Save Evaluation** stores a new evaluation with your name and timestamp and shows an *Evaluation Saved* confirmation.
- **View History** opens the Evaluation History dialog.

Notes:
- Each save creates a new record; previous evaluations are never overwritten. Dashboards use the latest evaluation of each ticket.
- You need the **Evaluate Tickets** permission in the ticket's project; otherwise saving shows *Not allowed*.
- The average score is the plain mean of all criteria scores.

### Evaluation History
A table of the evaluations saved for the issue, newest first: **Date**, **Evaluator**, **Score**, and a **View Details** action that reveals the per-criterion scores and the feedback of that evaluation. The dialog shows the **10 most recent** evaluations; there is no paging control, so older evaluations of a heavily re-evaluated ticket are visible only through the dashboards' detail sheets and exports.

---

## Project landing page

Opened from **Apps → Quality Scorecard for Jira**.
- Lists the projects enabled for the app with their evaluation counts; click a card to open its dashboard.
- Opens the dashboard automatically when exactly one project is enabled.
- Administrators see a **Settings** button (and, when nothing is configured yet, **Configure Projects**) to manage the project allow-list.
- A theme toggle (light/dark) is available on every page; the preference is saved.

---

## Analytics Dashboard tab

### Filters card
- **Date Range** picker (start–end). Click **Apply Filters** to reload.
- **Export** with a format selector: **CSV** or **Excel**.

### Cards
| Card | Content |
|------|---------|
| **Average Score** | Mean score of all evaluations in the range |
| **Total Evaluations** | Number of evaluations submitted |
| **Evaluations by Day** | Bar chart of evaluations per day |
| **Average Score by Created Day** | Average of evaluated tickets grouped by the ticket's creation date |
| **Score Distribution** | Histogram of average scores |
| **Criteria Averages** | Average per criterion |
| **Priority Performance** | Average score by Jira priority |
| **Issue Type Performance** | Average score by Jira issue type |

Clicking a bar in the Priority / Issue Type / Distribution charts opens a **ticket detail sheet** listing the matching tickets (priority, assignee, evaluator, score) with links to Jira.

### Raw Data table
All evaluated tickets in the range with ticket key, summary, priority, assignee, evaluator, score and feedback. Sortable columns; **10 / 20 / 50 / 100** rows per page; click the ticket key to open it in Jira.

What the Analytics Dashboard does **not** have: an evaluator filter, a trend KPI, an "active evaluators" KPI or weekly/monthly roll-ups. Use Evaluator Analytics and Advanced Analytics for those.

---

## Agent Performance tab

"Agents" are the Jira **assignees** of evaluated tickets.

### Filters card
- **Search agents…** (name)
- Sort: **Score (High to Low)**, **Score (Low to High)**, **Most Evaluations**, **Least Evaluations**, **Name (A-Z)**, **Name (Z-A)**
- Performance band: **All Agents**, **Poor Performance (< 5)**, **Average Performance (5-7.9)**, **Good Performance (≥ 8)**
- **Date Range** + **Apply Filters**
- **Export** (CSV / Excel)

### Summary cards
**Total Agents**, **Average Score** (weighted by evaluation count), **Total Evaluations**, **Performance Distribution** (count of Poor / Avg / Good agents).

### Agent cards and detail view
Each card shows the agent's average, evaluation count and top criteria. Clicking a card opens the agent detail modal with two tabs:
- **Performance Analysis** – Criteria Performance Overview, Score Evolution (by ticket creation date), Score Distribution, Performance by Priority, Performance by Issue Type, Detailed Criteria Scores.
- **Evaluated Tickets** – table of the agent's evaluated tickets with evaluator and feedback; click to open in Jira.

Trend direction shown in agent analytics: **improving** when the recent average is more than 0.5 points above the earlier average, **declining** when more than 0.5 below, otherwise **stable**.

---

## Evaluator Analytics tab

### Filters card
- **Search evaluators…**
- Sort: **Most Evaluations**, **Least Evaluations**, **Highest Average**, **Lowest Average**, **Name (A-Z)**, **Name (Z-A)**
- **Min evaluations** threshold
- **Date Range** + **Apply Filters**
- **Show Biased Only** switch
- **Export** (CSV / Excel)

### Summary cards
**Total Evaluators**, **Team Average Score**, **Biased Evaluators** (with the ±2.5 pts threshold), **Avg Frequency** (evaluations per day).

### Bias detection
An evaluator is flagged when their average differs from the team average by more than **2.5 points**: **Lenient Evaluator** (higher) or **Strict Evaluator** (lower); otherwise **Balanced Evaluator**. The Bias Detection card lists lenient and strict evaluators separately.

### Evaluator Scoring cards and detail sheet
Click an evaluator card to open the detail sheet with tabs **Overview** (Overall Average, Total Evaluations, Activity Period), **Criteria** (per-criterion averages), **Agents** (how this evaluator scores each agent – the evaluator/agent matrix), and **Trends** (daily average scores across all of this evaluator's evaluations by ticket created date – the card subtitle says "last 30 days" but no 30-day cut-off is applied). The **Evaluated Tickets** table in the sheet lists the evaluator's tickets with feedback; users with the **Delete Evaluations** permission see a **Delete Evaluation** column to remove *all* evaluations of a ticket after a confirmation dialog ("This action cannot be undone").

### Missing Evaluations
Pick a date range and click **Load**. The table lists tickets in a *Done* status category, created in the range, that have no evaluation (up to 500). Columns: **Ticket**, **Summary**, **Priority**, **Assignee**, **Days Since Resolved** (shown in red after 7 days), **Actions** and **Remove**; sortable by ticket, priority and days; 10/20/50/100 rows per page.
- **Evaluate Now** opens the ticket in Jira so you can evaluate it from the issue panel.
- **Remove** hides a ticket from the list and from coverage totals (the choice is remembered site-wide for all users of the app; the issue panel shows a note on such tickets but still allows evaluation).

---

## Advanced Analytics tab

### Controls
**Date Range** (defaults to the last 30 days), **Export Format** (Excel / CSV), **Apply Filters**, **Export**, **Refresh** (recomputes; results are otherwise served from cache for up to two hours).

Unlike the other tabs, the date range here filters evaluations by their **evaluation date**, not by the ticket's created date; coverage additionally uses the ticket's **resolution date**. Select the export format explicitly before clicking **Export**: the control displays *Excel* first, but the export runs as CSV until a format has been chosen.

### Quick stats
- **Coverage Rate** – evaluated share of resolved tickets vs the configured **Target**
- **Trend Analysis** – **Short Term / Medium Term / Long Term**, each *improving*, *stable* or *declining*
- **Evaluation Volume** – evaluations in the period and approximate evaluations per day

### Detail tabs
| Tab | Cards |
|-----|-------|
| **Coverage** | Overall Coverage Analysis, Coverage by Ticket Type, Coverage by Priority, Sampling Effectiveness (bias indicators and recommendations) |
| **Trends & Capacity** | Trend Analysis, Capacity Analysis, Warnings & Recommendations |
| **Complexity** | Complexity by Ticket Type, Complexity by Priority |
| **Distribution** | Score Distribution Histogram, Normal Distribution Comparison (Bell Curve), Outlier Detection (click an outlier to open the ticket) |

Trend Analysis compares evaluations from the last 7 / 30 / 90 days (*improving* / *stable* / *declining*). Capacity Analysis uses a fixed optimal capacity of 10 evaluations per day; Warnings flag declining performance, more than 15 evaluations per day, or a recent average below 6. Outliers are scores more than two standard deviations from the mean.

Coverage is calculated from tickets by **resolution date**; coverage targets come from **Settings → Coverage Targets** (default overall 75 %; by priority Highest 100, High 100, Medium 50, Low 30, Lowest 20).

---

## Metrics tab (administrators)

The **Evaluation Criteria** editor:
- **Add Criteria** creates a new entry ("New Criteria") you can then **Edit**.
- For each criterion: **Name**, **Description**, **Default Value (1-10)**, **Required** switch; arrows reorder; the trash icon removes. The **Required** flag is informational only – the evaluation form always shows every criterion with a value, so nothing is enforced by it.
- **Save Changes** writes the list. Criteria are shared by all enabled projects.

Only Jira administrators can save changes to existing criteria. Removing a criterion does not delete historical scores; old evaluations keep their stored values and the criterion id is shown where the name is no longer known.

---

## Settings tab (administrators)

Three sub-tabs:

### Project Access & Recent Changes
- Inside a project's dashboard this sub-tab shows **Current Project Access** for that project only, with a remove action. To **add** projects (search by name or key, **Add Project**) use the project allow-list on the landing page (**Settings** / **Configure Projects**), where the full **Allowed Projects** list is shown.
- **Recent Changes** shows the last 10 add/remove actions with who did it and when.
- Only enabled projects can be opened, evaluated or searched by the app.

### Roles & Access
- **Create Role** – name plus a set of permissions: Dashboard Access, Analytics Dashboard, Agent Performance, Evaluator Analytics, Advanced Analytics, Metrics, Settings, Evaluate Tickets, Delete Evaluations.
- **Existing Roles** table with edit (**Edit Role Permissions**) and delete actions.
- **Add Users** – search users in the project, pick roles, **Add**. **Project Users** table lists assignments with a remove action.
- Audit list with a date range and **Load** showing role creation/deletion, users added/removed and evaluation deletions.
- Roles are defined **per project**. Jira administrators do not need a role.

### Coverage Targets
**Overall Target (%)** and per-priority targets used by Advanced Analytics → Coverage. **Save** is admin-only.

---

## Data export

Available on the Analytics, Agent Performance, Evaluator Analytics and Advanced Analytics tabs, plus a per-table export in ticket detail sheets.

| Dashboard | Excel worksheets |
|-----------|------------------|
| Analytics | Overview, Daily Evaluations, Score Distribution, Detailed Evaluations |
| Agent Performance | Summary, Agent Performance, Top Performers |
| Evaluator Analytics | Summary, Evaluator Performance, Missing Evaluations (when loaded) |
| Advanced Analytics | Complexity, Coverage, Forecast, Patterns, Distribution (the *Patterns* sheet always reads "N/A" – peak-hour analysis is no longer computed) |

CSV exports are flat tables in UTF-8. In CSV files, cell values beginning with `=`, `+`, `-` or `@` are prefixed with an apostrophe to prevent spreadsheet formula injection; Excel files write values as typed text cells. Exports always reflect the current filters. The Agent Performance export's "Team Average Score" is a plain mean across agents, whereas the on-screen **Average Score** card is weighted by evaluation count, so the two can differ. PDF and JSON exports are not available.

---

## Filtering

| Where | Filters |
|-------|---------|
| Analytics Dashboard | Date range |
| Agent Performance | Search, sort, performance band, date range |
| Evaluator Analytics | Search, sort, minimum evaluations, biased-only, date range |
| Advanced Analytics | Date range |
| Missing Evaluations | Date range |

There are no saved filters, no per-evaluator filter on the Analytics Dashboard, and no keyboard shortcuts.

---

## Security and privacy features (user-visible)

- Access to each tab and action is controlled by roles set by Jira administrators and enforced by the backend.
- Project access changes and role changes are recorded and visible under Settings.
- Only evaluation scores, feedback, your Atlassian account id and display name are stored by the app; it never writes to Jira issues.

See [Technical → Security](../Technical/Security.md) for details.

---

## Not included (to avoid confusion)

The app does not provide: evaluation editing in place (save a new evaluation instead), automation or alert rules, scheduled e-mail reports, PDF export, a Help menu, keyboard shortcuts, a mobile-specific layout (the UI is responsive but designed for desktop Jira), or a public REST API.

---

*For team processes, see the [Workflows Guide](./Workflows.md). For questions, see the [FAQ](./FAQ.md).*
