# Frequently Asked Questions (FAQ)

## General

### What is Quality Scorecard for Jira?
An Atlassian Forge app for Jira Cloud that lets teams score the quality of tickets against configurable criteria from the issue view and analyse the results in project dashboards (analytics, agent performance, evaluator analytics, advanced analytics).

### Who can use it?
Anyone with a Jira account on the site, **once a Jira administrator has enabled their project and given them a role** in Settings → Roles & Access. Without a role you can neither open the dashboard nor save evaluations. Jira administrators have full access automatically.

### Does it work on Jira Data Center / Server?
No. It is a Forge app and runs only on Jira Cloud.

### Does the app change my Jira issues?
No. The app only reads issues (`read:jira-work`) and stores evaluations in its own Forge storage. It does not request write access to Jira.

---

## Evaluations

### How do I create an evaluation?
Open the issue → **Quality Scorecard for Jira** panel → move the sliders (0–10) → optional **Feedback** → **Save Evaluation**.

### Can I edit an evaluation after saving?
Not in place. Open the ticket again; the sliders are pre-filled with the latest values. Change them and **Save Evaluation** — a new evaluation is stored and dashboards use the most recent one. The earlier evaluation stays in **View History**.

### Can evaluations be deleted?
Yes, by users with the **Delete Evaluations** permission: Evaluator Analytics → open the evaluator's detail sheet → **Evaluated Tickets** → **Delete Evaluation**. This removes *all* evaluations of that ticket (not a single one) and is recorded in the Settings audit list.

### Is feedback mandatory?
No. Feedback is optional for every score.

### Why do I get "Not allowed – You do not have permission to evaluate this ticket"?
Either the ticket's project is not enabled for the app, or you have no role with **Evaluate Tickets** in that project. Ask your Jira administrator.

### How many evaluations can a ticket have?
Unlimited. Dashboards use the latest; **View History** shows the 10 most recent (older ones remain stored and appear in the dashboards' detail sheets and exports).

### Can I evaluate tickets in projects not shown on the landing page?
No. Only projects an administrator has added under Settings → Project Access can be evaluated or analysed.

---

## Scoring

### What do the scores mean?
Each criterion is scored 0–10. Dashboards group averages into **Poor (< 5)**, **Average (5–7.9)** and **Good (≥ 8)**.

### How is the average score calculated?
The plain mean of all criteria scores in the evaluation. Criteria have no weights.

### What criteria are used?
Whatever your administrator configured in the **Metrics** tab. Out of the box there are eight: Clarity of Description, Completeness of Requirements, Reproducibility (for bugs), Technical Documentation, Test Coverage, Code Quality, Solution Efficiency, Adherence to Standards.

### Are criteria per project?
No, the criteria list is shared by all enabled projects on the site.

---

## Analytics

### How current are the dashboards?
New evaluations are available as soon as you **Apply Filters**/reload. For speed, results are cached in your browser (up to 30 minutes for most tabs, up to 2 hours for Advanced Analytics, which also shows the last result instantly and refreshes in the background). Use **Refresh** on Advanced Analytics to force a recompute.

### Why is a chart empty although we saved evaluations?
Most charts group by the ticket's **creation date** in Jira (Advanced Analytics coverage uses the **resolution date**), not by the time of the evaluation. Widen the date range and apply the filter again.

### What does "improving / stable / declining" mean?
A trend is *improving* when the recent average is more than 0.5 points above the earlier average, *declining* when more than 0.5 below, otherwise *stable*. Advanced Analytics shows this for short, medium and long term.

### What does "biased" mean in Evaluator Analytics?
An evaluator whose average differs from the team average by more than **2.5 points** is flagged *Lenient* (higher) or *Strict* (lower).

### What is coverage?
The share of resolved tickets in the period that have at least one evaluation, compared with the target set by your administrator (default 75 % overall; by priority Highest 100, High 100, Medium 50, Low 30, Lowest 20).

### Is there a trend or "active evaluators" number on the Analytics Dashboard?
No. The Analytics Dashboard shows Average Score, Total Evaluations, Evaluations by Day, Average Score by Created Day, Score Distribution, Criteria Averages, Priority Performance, Issue Type Performance and the raw data table. Trends are in Advanced Analytics; evaluator counts are in Evaluator Analytics.

### Can I filter the Analytics Dashboard by evaluator?
No. Use Evaluator Analytics (per evaluator) or export the raw data and filter in a spreadsheet.

### Can I export?
Yes — **Export** on every dashboard, as **CSV** or **Excel (.xlsx)**. Exports contain the data currently filtered. PDF and JSON are not available.

---

## Administration

### How do I give someone access?
Open the project dashboard → **Settings → Roles & Access** → **Create Role** (choose permissions) → **Add Users** → assign the role. Roles are per project.

### How do I change the evaluation criteria?
**Metrics** tab → edit, add, reorder or remove → **Save Changes**. Only Jira administrators can save changes.

### Can I restrict who can create evaluations?
Yes: only users whose role includes **Evaluate Tickets** can save evaluations. This is independent of Jira's own permission scheme (though users still need Jira access to the issue).

### Where can I see who changed settings?
- Project list changes: **Settings → Project Access & Recent Changes**.
- Role changes, user additions/removals and evaluation deletions: the audit list at the bottom of **Settings → Roles & Access** (requires the Settings permission).

### Do Jira project administrators get access automatically?
No. Only users with Jira's global **Administer Jira** permission (site admins) are treated as app administrators. Project admins need a role like everyone else.

### How do I handle GDPR / personal-data requests?
- The app stores evaluator account ids, display names, scores and feedback. It never stores e-mail addresses.
- When an Atlassian account is **closed**, the app learns this through Atlassian's Personal Data Reporting (weekly) and automatically removes that account's evaluations from the global list and its cached name.
- For an **access or erasure request** about a specific user, export the relevant data (Export → CSV) and contact support@nexusfrontier.net for a complete erasure; there is no self-service erasure button in the UI.
- Uninstalling the app deletes all app data for the site.
See the [Privacy Policy](../Legal/Privacy-Policy.md) and [Technical → Security](../Technical/Security.md).

### What happens to the data if we uninstall the app?
Forge deletes the app's storage for your site on uninstall. Export anything you need first.

### Is there a storage limit?
Evaluations are stored as lists in Forge storage, which caps the size of each list. Very active sites (several hundred evaluations in one project) may eventually see save errors; contact support if this happens. See [Entity Storage Migration Plan](../Technical/Entity-Storage-Migration-Plan.md) for the planned change.

---

## Technical

### Is my data secure?
Data is stored in Atlassian's Forge storage (encrypted at rest, HTTPS in transit, isolated per site). The app enforces its permissions on the server, logs denied access attempts, and never sends data outside Atlassian. See [Technical → Security](../Technical/Security.md).

### Can I integrate with other tools?
Via exports (CSV/Excel). There is no public API or webhook.

### Which browsers are supported?
Current versions of Chrome, Edge, Firefox and Safari.

### Does it work on mobile?
The UI is responsive but designed for the desktop Jira experience; the Jira mobile apps do not show Forge Custom UI panels.

### Are there keyboard shortcuts or an in-app Help menu?
No.

---

## Troubleshooting

### The app is slow
- First load of Evaluator Analytics or Advanced Analytics on a large project can take a while; later loads come from cache.
- Narrow the date range.
- The app fetches Jira data in batches of 50 tickets with at most 8 parallel requests; thousands of tickets take proportionally longer.

### I see an account id instead of a name
The name could not be resolved from Jira (deactivated account, or the `read:jira-user` scope was not approved after an upgrade). Ask your administrator to check **Manage apps** for a pending scope approval.

### Export does nothing
Check your browser's download/pop-up blocker and make sure the filtered table has rows.

### I cannot find an evaluation
Widen the date range (charts use ticket creation date), confirm you are in the right project, and check **View History** on the ticket itself.

### Something else is wrong
1. Reload the page.
2. Check your permissions with your administrator.
3. Note the exact error text and time, then contact **support@nexusfrontier.net** — see the [Support SLA](../Legal/Support-SLA.md) for response targets.

---

## More help

- [Getting Started](./Getting-Started.md) · [Feature Guide](./Feature-Guide.md) · [Workflows](./Workflows.md)
- [Technical documentation](../README.md#technical-documentation)
- Feature requests and bug reports: support@nexusfrontier.net

---

*Can't find your answer? Contact support@nexusfrontier.net*
