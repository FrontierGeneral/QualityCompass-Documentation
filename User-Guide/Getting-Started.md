# Getting Started Guide

## Welcome to Quality Scorecard for Jira

Quality Scorecard for Jira lets your team score the quality of Jira tickets against a shared set of criteria, directly from the issue view, and then analyse the results per project, per assignee ("agent") and per evaluator.

## What the app does

- Evaluate ticket quality with 0–10 scores per criterion and optional written feedback
- Keep a full history of evaluations per ticket
- Analyse scores by day, priority, issue type and criterion
- Track agent (assignee) performance and evaluator consistency, including bias detection
- Measure evaluation coverage against targets and spot missing evaluations
- Export every dashboard to CSV or Excel

## First Steps

### 1. Open the app

The app has two entry points:

| Where | What you get |
|-------|--------------|
| **Jira top navigation → Apps → Quality Scorecard for Jira** | The project landing page and the analytics dashboard |
| **Any issue → "Quality Scorecard for Jira" panel in the issue view** | The evaluation form and the evaluation history for that issue |

On the landing page you will see the projects your administrator has enabled for the app. Click a project to open its dashboard. If only one project is enabled, it opens automatically. If you see *"Ask your administrator to add your projects to the QA app's allowed list"*, no project has been enabled yet.

### 2. Understand the dashboard tabs

Which tabs you see depends on the permissions your administrator gave you (see [Who can do what](#who-can-do-what)).

| Tab | Purpose |
|-----|---------|
| **Analytics Dashboard** | Overall KPIs, score distribution, criteria averages, priority and issue-type performance, raw data table |
| **Agent Performance** | Scores per assignee, with a detail view per agent |
| **Evaluator Analytics** | Scores per evaluator, bias detection, missing evaluations |
| **Advanced Analytics** | Coverage vs. targets, trends & capacity, complexity, distribution |
| **Metrics** | Edit the evaluation criteria (administrators) |
| **Settings** | Project access, roles & access, coverage targets (administrators) |

Use the theme toggle in the header to switch between light and dark mode; your choice is remembered.

### 3. Your first evaluation

1. **Open a Jira issue** in an enabled project and find the **Quality Scorecard for Jira** panel.
2. **Score each criterion** with its slider (whole numbers from 0 to 10; the slider starts at the criterion's default, normally 5). If the ticket was evaluated before, the sliders and feedback are pre-filled from the latest evaluation so you can start from there.
3. **Add feedback** (optional) in the *Feedback* box. Be specific and actionable.
4. Click **Save Evaluation**. A confirmation toast appears and the evaluation is stored with your name and the current time.
5. Click **View History** to see the evaluations saved for the issue (the 10 most recent), with the per-criterion scores and feedback of each one.

Every save creates a **new** evaluation record; earlier ones are kept. Dashboards use the most recent evaluation of each ticket.

If the **Save Evaluation** button shows "Not allowed – You do not have permission to evaluate this ticket", ask your administrator for the *Evaluate Tickets* permission in that project.

## Evaluation criteria

Your administrator can change the criteria in the **Metrics** tab, so your instance may differ. The app ships with eight default criteria, each scored 0–10:

| Criterion | What it measures |
|-----------|------------------|
| Clarity of Description | Is the ticket description clear and unambiguous? |
| Completeness of Requirements | Are all requirements captured? |
| Reproducibility (for bugs) | Can the issue be reproduced from the information given? *(optional criterion)* |
| Technical Documentation | Is the technical documentation adequate? |
| Test Coverage | Were tests added/updated appropriately? |
| Code Quality | Quality of the implementation |
| Solution Efficiency | Is the solution efficient and proportionate? |
| Adherence to Standards | Does the work follow team standards? |

The **average score** of an evaluation is the plain average of all criteria scores (no weighting).

### Scoring scale used in dashboards

| Range | Label |
|-------|-------|
| below 5 | Poor |
| 5 – 7.9 | Average |
| 8 – 10 | Good |

## Viewing analytics

### Analytics Dashboard
Pick a **date range** and click **Apply Filters**. The dashboard shows:
- **Average Score** and **Total Evaluations** for the period
- **Evaluations by Day** and **Average Score by Created Day**
- **Score Distribution**, **Criteria Averages**, **Priority Performance**, **Issue Type Performance**
- A **Raw Data** table (10/20/50/100 rows per page, sortable) with ticket, priority, assignee, evaluator, score and feedback. Click a ticket to open it in Jira.

Most charts are based on the ticket's **creation date** in Jira, so if a chart looks empty, widen the date range.

### Agent Performance
Search agents, sort by score/evaluations/name, filter by performance band, and click an agent card for charts (criteria overview, score evolution, distribution, by priority, by issue type) and the list of evaluated tickets.

### Evaluator Analytics
Shows every evaluator's average, evaluation count and frequency. Evaluators whose average differs from the team average by more than **2.5 points** are flagged as *Lenient* or *Strict*. The **Missing Evaluations** section lists tickets in a *Done* status, created in a date range you choose, that have no evaluation yet (pick the range and click **Load**).

### Advanced Analytics
Coverage rate against the configured target, short/medium/long-term trend (improving / stable / declining), evaluation volume, and four detail tabs: Coverage, Trends & Capacity, Complexity, Distribution. Results are cached for speed; click **Refresh** to recompute.

## Exporting data

Every dashboard has an **Export** control with two formats:
- **CSV** – one flat table
- **Excel (.xlsx)** – a workbook with several worksheets (summary, per-criterion data, raw rows)

Exports contain the data currently shown, after your filters. File names follow `<dashboard>-<project>-<date>`.

## Who can do what

Access is granted per project by your Jira administrator through **Settings → Roles & Access**:

| Permission | Lets you |
|------------|----------|
| Dashboard Access | Open the project dashboard |
| Analytics Dashboard / Agent Performance / Evaluator Analytics / Advanced Analytics | See the corresponding tab |
| Metrics | Open the criteria editor tab (saving changes still requires Jira admin rights) |
| Settings | Open the Settings tab and view role audit history |
| Evaluate Tickets | Save evaluations from the issue panel |
| Delete Evaluations | Delete all evaluations of a ticket (from the Evaluator Analytics detail view) |

Jira administrators automatically have everything.

## Tips

**Evaluators**
- Evaluate soon after a ticket is resolved, while the context is fresh.
- Use the whole 0–10 range; clustering around 7–8 hides differences.
- Write feedback the assignee can act on.

**Team leads**
- Review the Analytics Dashboard weekly and Evaluator Analytics monthly to catch bias early.
- Set realistic coverage targets in Settings and check the Coverage tab.

**Agents**
- Open Agent Performance, find your card, and read the feedback on your evaluated tickets.

## Next steps

1. Read the [Feature Guide](./Feature-Guide.md) for every screen in detail.
2. See [Workflows](./Workflows.md) for team processes (calibration, weekly review, onboarding).
3. Check the [FAQ](./FAQ.md) if something does not behave as expected.

---

*Need more help? Contact support@nexusfrontier.net.*
