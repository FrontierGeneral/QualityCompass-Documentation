# Getting Started Guide

## Welcome to Quality Compass app! 🎯

This guide will help you get started with the Quality Compass app, a powerful tool for managing quality assurance evaluations directly within your Jira projects.

## What is the Quality Compass app?

The Quality Compass app helps teams:
- ✅ Evaluate ticket quality systematically
- 📊 Track performance metrics over time
- 👥 Monitor agent and evaluator performance
- 📈 Generate comprehensive analytics reports
- 🎯 Identify areas for improvement

## First Steps

### 1. Accessing the App

After your administrator has installed the app, you can access it from any Jira project:

1. **Open your Jira project**
2. **Look for the "Apps" menu** in the project navigation
3. **Click on "Quality Compass app"** to open the dashboard

![Quality Compass app Menu Location](../Images/Screenshots/menu-location.png)

### 2. Understanding the Dashboard

When you first open the app, you'll see the main dashboard with several tabs (availability depends on your permissions):

- **📊 Analytics**: View overall quality metrics and trends
- **👤 Agent Performance**: Track individual agent performance
- **🎯 Evaluator Analytics**: Monitor evaluator consistency (with missing evaluations on-demand)
- **📈 Advanced Analytics**: Coverage, trends, distribution, patterns, and forecasting
- **🧩 Configuration**: Manage evaluation criteria (admin)
- **⚙️ Settings**: Project access, roles & access, and coverage targets (admin)

### 3. Your First Evaluation

Let's create your first ticket evaluation:

#### Step 1: Select a Ticket
1. Navigate to any Jira ticket in your project
2. Look for the **"Evaluate"** button or panel
3. Click to open the evaluation form

#### Step 2: Score the Criteria
For each evaluation criterion (e.g., Accuracy, Completeness, Communication):
1. **Read the criterion description**
2. **Select a score from 0-10** using the slider or input field
   - 0-4.9: Poor performance
   - 5-7.9: Average performance
   - 8-10: Good performance

#### Step 3: Add Feedback
1. **Provide constructive feedback** in the text area
2. Be specific about what was done well and areas for improvement
3. Keep feedback professional and actionable

#### Step 4: Submit
1. **Review your scores** to ensure accuracy
2. **Click "Submit Evaluation"**
3. You'll see a confirmation message

## Understanding Evaluation Criteria

### Default Criteria

The app comes with three default evaluation criteria:

#### 1. Accuracy (🎯)
- **What it measures**: How correct and precise the solution is
- **Scoring guide**:
  - 10: Perfect solution with no errors
  - 7-9: Minor issues that don't affect functionality
  - 4-6: Some errors but generally correct approach
  - 0-3: Major errors or incorrect solution

#### 2. Completeness (✅)
- **What it measures**: Whether all requirements were addressed
- **Scoring guide**:
  - 10: All requirements fully met
  - 7-9: Most requirements met with minor gaps
  - 4-6: Several requirements missing
  - 0-3: Many requirements not addressed

#### 3. Communication (💬)
- **What it measures**: Clarity of communication and documentation
- **Scoring guide**:
  - 10: Crystal clear communication
  - 7-9: Good communication with minor improvements needed
  - 4-6: Adequate but could be clearer
  - 0-3: Poor or confusing communication

## Viewing Analytics

### Quick Stats
At the top of the Analytics dashboard, you'll see:
- **Total Evaluations**: Number of tickets evaluated
- **Average Score**: Overall quality score
- **Trend**: Whether quality is improving or declining
- **Active Evaluators**: Number of people performing evaluations

### Charts and Graphs

#### Score Distribution
Shows how scores are distributed across all evaluations:
- Helps identify if scores cluster around certain values
- Reveals scoring patterns and potential bias

#### Trend Over Time
Displays quality trends:
- Daily, weekly, or monthly views
- Helps track improvement initiatives
- Shows impact of training or process changes

#### Performance by Criteria
Breaks down scores by individual criteria:
- Identifies specific areas needing improvement
- Helps focus training efforts

## Exporting Data

You can export evaluation data for further analysis:

### Export Options
1. **Click the Export button** (usually in the top-right corner)
2. **Choose your format**:
   - **CSV**: For basic spreadsheet analysis
   - **Excel**: For advanced analysis with formatting

### What's Included in Exports
- Ticket details (ID, key, summary)
- Evaluation scores (overall and per criteria)
- Evaluator information
- Timestamps
- Feedback comments

## Performance & Tips

### Performance Notes
- First time you open a tab, data may take longer to compute. Subsequent opens are much faster due to caching.
- Use the Refresh/Apply Filters button to fetch fresh results if needed; cached data will update in the background.

### For Evaluators
1. **Be Consistent**: Use the same standards for all evaluations
2. **Provide Context**: Always include helpful feedback
3. **Stay Objective**: Focus on the work, not the person
4. **Regular Evaluations**: Evaluate tickets promptly for accurate metrics

### For Team Leads
1. **Monitor Trends**: Check analytics weekly
2. **Address Issues Early**: Use data to identify problems quickly
3. **Celebrate Improvements**: Recognize positive trends
4. **Share Insights**: Discuss findings in team meetings

### For Agents
1. **Review Feedback**: Learn from evaluation comments
2. **Track Your Progress**: Monitor your performance metrics
3. **Ask Questions**: Seek clarification on low scores
4. **Apply Learning**: Use feedback to improve future work

## Common Tasks

### Finding Your Evaluations
1. Go to **Analytics** tab
2. Use the **Filter** options
3. Select your name from the **Evaluator** dropdown
4. View your evaluation history

### Checking Your Performance (Agents)
1. Navigate to **Agent Performance** tab
2. Find your name in the list
3. Click to see detailed metrics
4. Review feedback and scores

### Comparing Time Periods
1. Use the **Date Range** selector
2. Choose your comparison periods
3. View side-by-side metrics
4. Identify trends and changes

## Keyboard Shortcuts

Make your workflow faster with these shortcuts:

- **Ctrl/Cmd + E**: Open evaluation form
- **Tab**: Navigate between score fields
- **Enter**: Submit evaluation (when form is complete)
- **Esc**: Cancel current action

## Getting Help

### In-App Help
- Look for **ℹ️ Info icons** next to features for quick help
- Hover over elements for tooltips
- Check the **Help** menu for guides

### Support Resources
- **Documentation**: You're reading it! 📚
- **FAQ Section**: [Common questions answered](./FAQ.md)
- **Contact Support**: support@nexusfrontier.net

## Next Steps

Now that you understand the basics:

1. **Complete 5 evaluations** to get comfortable with the process
2. **Explore the Analytics dashboard** to understand the metrics
3. **Read the [Feature Guide](./Feature-Guide.md)** for advanced features
4. **Check out [Workflows](./Workflows.md)** for team-specific processes

## Quick Reference Card

### Scoring Scale
```
0-4.9  ⭐     Poor
5-7.9  ⭐⭐    Average  
8-10   ⭐⭐⭐   Good
```

### Dashboard Icons
- 📊 Analytics
- 👤 Individual Performance
- 📈 Trends
- 📥 Export
- ⚙️ Settings
- ❓ Help

### Important Terms
- **Evaluation**: Assessment of a ticket's quality
- **Criteria**: Specific aspects being scored
- **Trend**: Direction of performance over time
- **Baseline**: Reference point for comparisons

---

Ready to improve your team's quality? Let's get started! 🚀

*Need more help? Check our [FAQ](./FAQ.md) or contact support.*

