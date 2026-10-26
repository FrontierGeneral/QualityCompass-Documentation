# Installation Guide

## Quick Links
- [Architecture Overview](./Architecture.md) - Understand the system before installation
- [API Reference](./API-Reference.md) - Resolver documentation
- [Development Guide](./Development.md) - Developer setup (deploying your own build)
- [Security Documentation](./Security.md) - Scopes, authorization and data handling
- [Technical Troubleshooting](./Troubleshooting-Technical.md) - Installation issues

## Prerequisites

### For customers (Marketplace install)
- An active **Jira Cloud** site (the app is not available for Jira Data Center/Server).
- **Jira administrator** (site admin) rights to install apps and to perform first-run configuration.
- A modern browser (Chrome, Edge, Firefox, Safari).

### For developers (self-deployment of a build)
- Node.js **22.x** (matches the Forge runtime `nodejs22.x`) and npm 10+.
- Forge CLI (`npm install -g @forge/cli`) and an Atlassian developer account.
- Access to the app source repository.

## Part A – Install from the Atlassian Marketplace

1. In Jira, open **Apps → Explore more apps**.
2. Search for **Quality Scorecard for Jira** and click **Get app** (or **Try it free**).
3. Accept the requested scopes. The app asks for:
   - `read:jira-work` – read issues and projects
   - `read:jira-user` – resolve account ids to display names
   - `storage:app` – store evaluations and configuration in Forge storage
   - `report:personal-data` – Personal Data Reporting (GDPR) to Atlassian
   It does **not** request write access to Jira issues.
4. After installation the app appears in **Apps → Manage apps** and in the **Apps** menu as **Quality Scorecard for Jira**.

Licensing is enabled: the app follows the Marketplace trial/subscription state of your site.

## Part B – First-run configuration (Jira administrator)

### Step 1: Allow projects
The app only works with projects an administrator has explicitly allowed.

1. Open **Apps → Quality Scorecard for Jira**.
2. As an admin with no projects configured you will see **"No projects are configured yet"** → click **Configure Projects** (the same panel is available later as **Settings** on the project landing page, and under **Settings → Project Access & Recent Changes** inside a project dashboard).
3. Search projects by name or key and add them to the allowed list. Each change is recorded in **Recent Changes** with who made it and when.

Until at least one project is allowed, non-admin users see "Ask your administrator to add your projects…" and no Jira searches are performed.

### Step 2: Review evaluation criteria
1. Open a project from the landing page, then the **Metrics** tab.
2. The app seeds eight default criteria on first use:

| Id | Name | Required | Default |
|----|------|----------|---------|
| `clarity` | Clarity of Description | yes | 5 |
| `completeness` | Completeness of Requirements | yes | 5 |
| `reproducibility` | Reproducibility (for bugs) | no | 5 |
| `documentation` | Technical Documentation | yes | 5 |
| `testCoverage` | Test Coverage | yes | 5 |
| `codeQuality` | Code Quality | yes | 5 |
| `efficiency` | Solution Efficiency | yes | 5 |
| `adherence` | Adherence to Standards | yes | 5 |

3. Add, edit, reorder or remove criteria and click **Save Changes**. Only administrators can change criteria once they exist. Criteria are global (shared by all allowed projects) and every criterion weighs equally in the average score.

### Step 3: Grant access
Non-administrators have **no access** until they are given a role.

1. Inside the project dashboard open **Settings → Roles & Access**.
2. **Create Role** – name it (e.g. "QA Analyst") and tick the permissions it grants:
   - Dashboard Access, Analytics Dashboard, Agent Performance, Evaluator Analytics, Advanced Analytics, Metrics, Settings, Evaluate Tickets, Delete Evaluations.
3. **Add User** – search for users in the project and assign one or more roles.
4. Changes appear in the audit list at the bottom of the card.

Typical roles:

| Role | Permissions |
|------|-------------|
| Evaluator | Evaluate Tickets |
| QA Lead | Dashboard Access + all analytics tabs + Evaluate Tickets |
| QA Manager | QA Lead + Delete Evaluations + Metrics + Settings |

Jira administrators automatically hold every permission in every allowed project. Roles and assignments are per project.

### Step 4: Coverage targets (optional)
**Settings → Coverage Targets** lets administrators set the overall coverage target (default 75 %) and per-priority targets (Highest 100, High 100, Medium 50, Low 30, Lowest 20). These drive the Coverage analysis in the Advanced Analytics tab.

### Step 5: Verify
- Open any issue in an allowed project; the **Quality Scorecard for Jira** panel should show the criteria sliders.
- Save a test evaluation; it should appear under **View History** and, after refresh, in the Analytics Dashboard.
- Export a CSV or Excel file from a dashboard.
- Log in as a user without a role and confirm the dashboard is not accessible; assign a role and confirm it becomes available.

## Part C – Deploying a build yourself (developers)

```bash
# 1. Install dependencies and build the Custom UI
npm ci
npm run build          # → dist/

# 2. Authenticate and register your own copy of the app (first time only)
forge login
forge register         # creates a new app id in manifest.yml

# 3. Deploy and install
forge deploy -e development
forge install -e development --site <your-site>.atlassian.net --product jira

# 4. Later deployments
forge deploy -e development
forge install --upgrade -e development   # required when scopes or modules change
```

Notes:
- `npm run build` runs `vite build`; the resolver (`src/index.js`) and the scheduled function (`src/personalDataReporting.js`) are deployed as-is by `forge deploy`.
- `forge tunnel` can proxy the development environment to your local build for faster iteration (see [Development Guide](./Development.md)).
- There is no `.env` file or environment variables to configure; all configuration lives in `manifest.yml` and in Forge storage.
- Do not deploy to `production` of the published app while a Marketplace review is in progress.

## Uninstallation

1. **Apps → Manage apps → Quality Scorecard for Jira → Uninstall**.
2. Forge deletes the app's storage for the site when the app is uninstalled. Export anything you want to keep first (each dashboard has **Export** with CSV/Excel).
3. Browser-side caches (IndexedDB `qc-persisted-cache`) are harmless and expire on their own.

## Upgrades

Marketplace upgrades are applied by Atlassian automatically for minor versions. If a new version requests additional scopes, a Jira administrator must approve the upgrade in **Manage apps**. Evaluation data is stored in a forward-compatible JSON shape; no manual migration has been required between released versions.

## Installation Validation Checklist

- [ ] App listed in Manage apps and enabled
- [ ] At least one project allowed
- [ ] Criteria reviewed in Metrics
- [ ] Roles created and users assigned
- [ ] Issue panel visible on an allowed project's issues
- [ ] Test evaluation saved and visible in Analytics
- [ ] Export works
- [ ] Users without roles cannot access the dashboard

## Support

- Technical troubleshooting: [Troubleshooting-Technical.md](./Troubleshooting-Technical.md)
- Support: support@nexusfrontier.net — see the [Support SLA](../Legal/Support-SLA.md)

---

*Last Updated: October 2026 — reflects app version 1.3.0*
