# Feature Guide

## Complete Feature Reference

This guide provides detailed information about every feature in the Quality Scorecard for Jira app. Each section includes step-by-step instructions, tips, and best practices.

## 📊 Analytics Dashboard

### Overview
The Analytics Dashboard is your central hub for understanding quality metrics across your project.

### Key Features

#### 1. Key Metrics
**What it shows:**
- Total evaluations completed
- Overall average score
- Current quality trend
- Number of active evaluators

**How to use:**
1. Open the Analytics tab
2. Metrics use caching and update automatically every few minutes; click Refresh for the latest
3. Click on any metric for detailed breakdown

**Pro tip:** Set up a dashboard on a monitor for team visibility!

#### 2. Score Distribution Chart
**Purpose:** Visualize how scores are distributed

**Understanding the chart:**
- X-axis: Score levels
- Y-axis: Number of evaluations

**What to look for:**
- Bell curve = normal distribution
- Skewed left = quality issues
- Skewed right = excellent performance

#### 3. Trend Analysis
**Features:**
- Daily, weekly, monthly views
- Moving averages
- Trend indicators (↑ improving, ↓ declining, → stable)

**How to analyze:**
1. Select your time range
2. Look for patterns
3. Correlate with team events (training, new processes)

#### 4. Criteria Breakdown
**Shows:** Performance by individual criteria

**Using this data:**
- Identify weakest areas
- Plan targeted training
- Track improvement initiatives

### Export Options
1. **Click Export** button
2. **Choose format:**
   - CSV for data analysis
   - Excel for formatted reports
3. **Select data range**
4. **Download file**

---

## 👤 Agent Performance Dashboard

### Overview
Track and analyze individual agent performance to provide targeted coaching and recognition.

### Core Features

#### 1. Performance Table
**Columns:**
- Agent name
- Total evaluations
- Average score
- Trend (with percentage)
- Top criteria

**Sorting options:**
- By name (A-Z, Z-A)
- By score (highest/lowest)
- By evaluation count
- By trend

#### 2. Individual Agent View
**Access:** Click on any agent's name

**Details shown:**
- Complete evaluation history
- Score progression over time
- Criteria performance radar chart
- All feedback received
- Comparison to team average

#### 3. Performance Filters
**Available filters:**
- Date range
- Minimum evaluations
- Score threshold
- Specific criteria
- Evaluator

**How to filter:**
1. Click Filter button
2. Set your criteria
3. Apply filters
4. Results update instantly

#### 4. Agent Details
Click an agent to view their detailed performance, including evaluation history, score trends, and criteria breakdown. Use filters and sorting to refine the view.

### Performance Indicators

#### Trend Calculation
- **Improving (↑):** Recent average > historical by 0.5+ points
- **Stable (→):** Change within ±0.5 points
- **Declining (↓):** Recent average < historical by 0.5+ points

---

## 🎯 Evaluator Analytics

### Overview
Monitor evaluator consistency and identify potential bias in scoring.

### Key Metrics

#### 1. Evaluator Statistics
**For each evaluator:**
- Total evaluations performed
- Average score given
- Scoring distribution
- Bias indicators
- Time patterns

#### 2. Bias Detection
**System identifies:**
- **Lenient scorers**: Average 2.5+ points above team
- **Strict scorers**: Average 2.5+ points below team

Bias flags include:
- Outlier (Yes/No)
- Direction (lenient/strict/neutral)

#### 3. Evaluator Matrix
**Shows:** How each evaluator scores each agent

**Reading the matrix:**
- Rows: Evaluators
- Columns: Agents
- Colors: Score ranges
- Numbers: Average scores

**Use cases:**
- Identify evaluator-agent patterns
- Detect favoritism or conflicts
- Ensure fair distribution

---

## 📈 Advanced Analytics

### Overview
View deeper breakdowns and trends based on your selected filters. Focus areas include coverage views and evaluator/agent trends.

### Available Views
- Trend lines with recent vs baseline performance
- Coverage by priority/issue type
- Distribution charts with click-through details

Note: Predictive forecasting, capacity insights, correlations, patterns, and coverage are available in this tab. Initial load may take longer; subsequent loads are near-instant.

## 📝 Evaluation Management

### Creating Evaluations

#### Standard Evaluation Process
1. **Open ticket** to evaluate
2. **Click "Evaluate"** button
3. **Score each criterion** (0-10)
4. **Add feedback** (required for scores < 7)
5. **Review and submit**

#### Multiple Evaluations
**Note:** Each ticket can have multiple evaluations, but only the most recent evaluation is used in analytics calculations.

### Editing Evaluations

#### When you can edit:
- Any time
- With appropriate permissions

#### How to edit:
1. Find evaluation in history
2. Click "Edit" button
3. Modify scores/feedback
4. Save changes
5. Changes are saved

### Evaluation Best Practices

#### Consistency Tips:
- Use the full 0-10 scale
- Provide specific feedback
- Evaluate promptly after ticket resolution
- Be consistent across similar tickets

---

## 📥 Data Export

### Export Formats

#### CSV Export
**Best for:**
- Simple data analysis
- Database imports
- Basic spreadsheets

**Includes:**
- Raw data without formatting
- All selected fields
- UTF-8 encoding

#### Excel Export
**Best for:**
- Formatted reports

**Features:**
- Single worksheet
- Bold headers and auto-sized columns

### Export Options

#### Data Selection
1. **Full export**: All available data
2. **Filtered export**: Current filter applied
3. **Custom range**: Specific date range
4. **Selective fields**: Choose columns

#### Manual Export
**Export data on demand:**
1. Click the Export button in any dashboard
2. Choose format (CSV or Excel)
3. Download file

---

## ⚙️ Settings & Configuration

### User Preferences

#### Display Settings
- **Theme**: Light/Dark mode toggle
- **Date Range**: Filter data by time period

### Project Settings (Admin Only)

#### Available Admin Settings
**Administrators can manage:**
1. **Project Access Control** - Add/remove allowed projects
2. **Evaluator Name Management** - Override displayed names (manual mapping coexists with auto-resolved names)
3. **Coverage Targets** - Set evaluation coverage goals

#### Permission Requirements
**Based on Jira permissions:**
- **View analytics**: Browse Projects permission
- **Create/edit evaluations**: Edit Issues permission
- **Admin functions**: Administer Projects permission

---

## 🔍 Filtering Data

### Available Filters
**Filter analytics by:**
- Date range
- Evaluator

### Advanced Filters
Advanced filter combinations and saved filters are not available.

---

## 📱 Mobile Compatibility

### Responsive Design
The app works on mobile devices with:
- Responsive layouts that adapt to screen size
- Touch-friendly buttons and controls
- Scrollable tables and charts
- Mobile-optimized navigation

---

## 🔐 Security Features

### Data Protection
- Encrypted storage
- Secure transmission
- Access logging
- Session management

### Activity Tracking
**The app tracks:**
- All evaluations created
- Evaluation edits
- Settings changes (project access, evaluator names, coverage targets)

**Note:** Detailed audit logging is not currently implemented in the UI.

---

## 🎨 Customization

### Available Customizations
**Users can customize:**
- Theme (Light/Dark mode)
- Date range filters
- Export format preferences
- Sort order in tables

---

## 💡 Pro Tips

### Navigation Tips
- Use tabs to navigate between dashboards
- Click on charts for detailed views
- Use filters to focus on specific data
- Export data for offline analysis

### Advanced Features
1. **Interactive Charts**: Click on bar charts to see detailed ticket lists
2. **Data Export**: Export to CSV or Excel for further analysis
3. **Performance Tracking**: Monitor trends over time
4. **Coverage Analysis**: Track evaluation coverage rates

### Best Practices
1. **Regular evaluations**: Daily is better than weekly batches
2. **Consistent standards**: Use evaluation guides
3. **Actionable feedback**: Be specific and constructive
4. **Data hygiene**: Regular cleanup and validation
5. **Team calibration**: Monthly alignment sessions

---

*For more specific workflows, see our [Workflows Guide](./Workflows.md)*

