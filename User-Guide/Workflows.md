# Workflow Guides

## Team Workflows

Step-by-step routines for the roles that typically use Quality Scorecard for Jira. Every step refers to a control that exists in the app (see the [Feature Guide](./Feature-Guide.md)); process steps that happen outside the app (meetings, documents) are marked *(outside the app)*.

---

## Daily workflows

### Morning review (team lead) — 15 minutes

1. **Analytics Dashboard** → set the date range to yesterday (or the last 7 days) → **Apply Filters**. Note **Total Evaluations** and **Average Score**.
2. **Evaluator Analytics** → scroll to **Missing Evaluations** → choose yesterday's range → **Load**. Decide who evaluates the remaining tickets; **Remove** any ticket that should not be evaluated.
3. **Agent Performance** → sort by **Score (Low to High)** and filter **Poor Performance (< 5)** to spot agents who may need coaching.
4. Share the highlights in stand-up *(outside the app)*.

### End-of-day evaluation (evaluator) — 30 minutes

1. In Jira, search for the day's resolved tickets, e.g. `project = PROJ AND statusCategory = Done AND resolved >= -1d`.
2. Open each ticket. In the **Quality Scorecard for Jira** panel, score every criterion and add feedback.
3. **Save Evaluation**. Repeat for the next ticket.
4. If you must change something you already saved, open the ticket again, adjust the sliders (they are pre-filled with the latest values) and save again — the newest evaluation is the one dashboards use.

---

## Weekly workflows

### Personal review (agent) — 20 minutes

1. **Agent Performance** → **Search agents…** for your name → click your card.
2. **Performance Analysis** tab: check **Criteria Performance Overview** for your weakest criteria and **Score Evolution** for the direction.
3. **Evaluated Tickets** tab: read the feedback column; open tickets in Jira where needed.
4. Pick 2–3 improvement goals *(outside the app)*.

### Team review (manager) — 45 minutes

1. **Analytics Dashboard** → date range = last 7 days → **Apply Filters** → review **Criteria Averages**, **Priority Performance**, **Issue Type Performance**.
2. **Export** → Excel for the meeting pack (worksheets: Overview, Daily Evaluations, Score Distribution, Detailed Evaluations).
3. **Agent Performance** → same range → sort by **Score (High to Low)**; note top performers and those in the *Poor* band.
4. **Evaluator Analytics** → same range → check **Biased Evaluators**; toggle **Show Biased Only** to list them. Plan a calibration touch-point if anyone is flagged.
5. **Advanced Analytics** → **Coverage** tab → compare **Coverage Rate** with the **Target**; **Trends & Capacity** for the short-term trend and recommendations.

---

## Monthly workflows

### Calibration session (evaluator team) — 2 hours

1. **Preparation**: a lead picks 5 recently resolved tickets. Each evaluator scores them independently from the issue panel and saves. *(All evaluations are kept, so this is safe to do on real tickets.)*
2. In the session, open each ticket's **View History** dialog to compare evaluations side by side (**View Details** shows per-criterion scores and feedback).
3. Open **Evaluator Analytics** for the month; review **Lenient Evaluator** / **Strict Evaluator** badges and the **Agents** tab of each evaluator's detail sheet for evaluator–agent patterns.
4. Agree on interpretations and record them in the criterion **Description** fields in the **Metrics** tab (administrator) so they are visible on the evaluation form.

### Executive report (QA manager) — 2 hours

1. **Analytics Dashboard**, **Agent Performance**, **Evaluator Analytics**, **Advanced Analytics** → date range = the month → **Export** each to Excel.
2. In **Advanced Analytics** record **Coverage Rate** vs target, the **Long Term** trend, and the **Outlier Detection** list from the **Distribution** tab.
3. Compile the report *(outside the app)*: overall average, month-over-month change, coverage, bias findings, top and bottom performers, recommendations.

---

## Setup workflows

### Enabling a new project (administrator) — 30 minutes

1. **Apps → Quality Scorecard for Jira** → **Settings** (or **Configure Projects** on first use) → search the project → add it. The change appears under **Recent Changes**.
2. Open the project → **Metrics** tab → review the criteria. Criteria are shared across projects; adjust names/descriptions/defaults, add or remove criteria, **Save Changes**.
3. **Settings → Roles & Access** → **Create Role** for evaluators (Evaluate Tickets), leads (Dashboard Access + analytics tabs + Evaluate Tickets) and managers (adds Delete Evaluations, Metrics, Settings). Then **Add Users** and assign roles.
4. **Settings → Coverage Targets** → set **Overall Target (%)** and per-priority targets → **Save**.
5. Ask one evaluator to save a test evaluation from an issue panel and confirm it appears in the **Analytics Dashboard** after **Apply Filters**.

### Onboarding a new evaluator — 1 hour

1. Administrator: **Settings → Roles & Access → Add Users** → assign the evaluator role in each relevant project.
2. Walk through the evaluation form on a sample ticket; show **View History** so they can see how peers scored similar tickets.
3. Shadow period: evaluate 5–10 tickets alongside an experienced evaluator; compare via **View History**.
4. After two weeks, review their card in **Evaluator Analytics** (average vs team, any bias badge) and give feedback.

### Removing a user's access — 5 minutes

**Settings → Roles & Access → Project Users** → remove the user. The action is recorded in the audit list. Their past evaluations remain and continue to count in analytics.

---

## Housekeeping workflows

### Correcting or removing a wrong evaluation

- To **correct**: save a new evaluation on the ticket; dashboards use the latest.
- To **remove entirely** (requires *Delete Evaluations*): **Evaluator Analytics** → open the evaluator's detail sheet → **Evaluated Tickets** → **Delete Evaluation** on the row → confirm. This removes *all* evaluations of that ticket and is logged in the Settings audit list.

### Keeping the Missing Evaluations list meaningful

Tickets that should never be evaluated (duplicates, auto-closed, won't-fix) can be hidden with **Remove** in the Missing Evaluations table so they no longer distort coverage follow-up.

### Sprint retrospective input — 30 minutes

1. **Analytics Dashboard** → date range = sprint dates → **Apply Filters**.
2. Note **Average Score**, **Criteria Averages** and **Issue Type Performance**; click bars to open the ticket list and pick concrete examples.
3. **Export** to CSV and attach to the retrospective page *(outside the app)*.

---

## Quick reference

| Task | Where | Steps |
|------|-------|-------|
| Evaluate a ticket | Issue panel | Score sliders → Feedback → **Save Evaluation** |
| See who evaluated a ticket | Issue panel | **View History** |
| Weekly numbers | Analytics Dashboard | Date range → **Apply Filters** → **Export** |
| Find unevaluated tickets | Evaluator Analytics | Missing Evaluations → range → **Load** |
| Check for bias | Evaluator Analytics | **Show Biased Only** |
| Coverage vs target | Advanced Analytics | **Coverage** tab |
| Give someone access | Settings → Roles & Access | **Add Users** |
| Change criteria | Metrics | Edit → **Save Changes** |

The app has no automation, scheduled reports or alerting features; recurring reports are produced by exporting on a schedule you set in your calendar.

---

*For details on each screen, see the [Feature Guide](./Feature-Guide.md). For questions, see the [FAQ](./FAQ.md).*
