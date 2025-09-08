# Workflow Guides

## Team-Specific Workflows

This guide provides step-by-step workflows for different team roles and common scenarios in the Quality Compass app.

---

## Daily Workflows

### Morning Review Workflow (Team Lead)

**Time Required:** 15-20 minutes

**Steps:**
1. **Open Analytics Dashboard**
   - Check overnight evaluation activity
   - Note any significant score changes

2. **Review Alerts**
   - Check for quality threshold breaches
   - Identify tickets needing attention

3. **Check Agent Performance**
   - Sort by trend to identify declining performance
   - Note agents needing coaching

4. **Review Yesterday's Metrics**
   - Compare to previous day
   - Identify patterns or anomalies

5. **Plan Day's Focus**
   - Schedule coaching sessions if needed
   - Assign evaluators to priority tickets

**Output:** Daily quality briefing for team standup

---

### End-of-Day Evaluation Workflow (Evaluator)

**Time Required:** 30-45 minutes

**Steps:**
1. **Filter Today's Resolved Tickets**
   ```
   Status = Resolved AND resolved >= -1d
   ```

2. **Prioritize Evaluation Order**
   - Customer-reported issues first
   - High-priority tickets second
   - Standard tickets last

3. **Perform Evaluations**
   - Open each ticket
   - Review resolution thoroughly
   - Score each criterion
   - Add constructive feedback

4. **Review Your Work**
   - Check evaluation summary
   - Ensure feedback is actionable
   - Submit evaluations

5. **Update Status**
   - Mark evaluated tickets
   - Note any requiring follow-up

**Output:** All day's tickets evaluated with feedback

---

## Weekly Workflows

### Weekly Performance Review (Individual Agent)

**Time Required:** 30 minutes

**Steps:**
1. **Access Agent Performance Dashboard**
   - Find your name in the list
   - Click for detailed view

2. **Review Your Metrics**
   - Overall average score
   - Trend direction and percentage
   - Individual criteria scores

3. **Analyze Feedback**
   - Read all evaluator comments
   - Identify common themes
   - Note specific improvement areas

4. **Review vs Previous Week**
   - Check score progression
   - Identify what changed
   - Understand impact of improvements

5. **Create Action Plan**
   - List 2-3 improvement goals
   - Define specific actions
   - Set measurable targets

**Output:** Personal improvement plan for the week

---

### Weekly Team Performance Review (Manager)

**Time Required:** 1 hour

**Steps:**
1. **Generate Weekly Report**
   - Set date range to past 7 days
   - Include all team members
   - Export to Excel

2. **Analyze Team Metrics**
   - Overall team average
   - Distribution of scores
   - Trend analysis

3. **Identify Top Performers**
   - Highest average scores
   - Most improved agents
   - Consistent performers

4. **Identify Coaching Needs**
   - Lowest performers
   - Declining trends
   - Specific criteria weaknesses

5. **Review Evaluator Consistency**
   - Check bias indicators
   - Review evaluator averages
   - Note calibration needs

6. **Prepare Team Meeting**
   - Create presentation slides
   - Highlight achievements
   - Plan improvement initiatives

**Output:** Weekly team performance presentation

---

## Monthly Workflows

### Monthly Calibration Session (Evaluator Team)

**Time Required:** 2 hours

**Steps:**
1. **Preparation (Before Meeting)**
   - Select 5 sample tickets
   - Each evaluator scores independently
   - Don't share scores yet

2. **Meeting Setup**
   - Share screen with app open
   - Display Evaluator Analytics
   - Have scoring guide ready

3. **Review Sample Tickets**
   For each ticket:
   - Display ticket details
   - Each evaluator shares scores
   - Discuss score differences
   - Agree on "correct" score
   - Document reasoning

4. **Analyze Patterns**
   - Review evaluator bias indicators
   - Discuss scoring tendencies
   - Identify inconsistencies

5. **Establish Standards**
   - Update scoring guidelines
   - Clarify criteria interpretations
   - Document edge cases

6. **Action Items**
   - Individual adjustments needed
   - Training requirements
   - Follow-up schedule

**Output:** Calibrated scoring standards and action plan

---

### Monthly Executive Report (QA Manager)

**Time Required:** 2-3 hours

**Steps:**
1. **Data Collection**
   - Export monthly analytics
   - Generate trend reports
   - Compile agent rankings
   - Calculate team metrics

2. **Create Executive Summary**
   - Overall quality score
   - Month-over-month change
   - Key achievements
   - Areas of concern

3. **Analyze Trends**
   - Long-term quality trends
   - Seasonal patterns
   - Impact of initiatives

4. **ROI Calculation**
   - Quality improvements
   - Customer satisfaction correlation
   - Cost savings from fewer defects

5. **Prepare Visualizations**
   - Trend charts
   - Score distributions
   - Agent performance matrix
   - Criteria breakdown

6. **Develop Recommendations**
   - Resource needs
   - Training programs
   - Process improvements
   - Tool enhancements

**Output:** Executive presentation with insights and recommendations

---

## Project-Based Workflows

### New Project Setup

**Time Required:** 1 hour

**Steps:**
1. **Configure Project Access**
   - Add project to allowed projects
   - Set up evaluator permissions
   - Configure admin access

2. **Define Evaluation Criteria**
   - Review default criteria
   - Customize for project needs
   - Set weights if needed
   - Add project-specific criteria

3. **Set Quality Thresholds**
   - Define score ranges
   - Set alert thresholds
   - Configure notifications

4. **Create Evaluation Templates**
   - Standard evaluation template
   - Bug evaluation template
   - Feature evaluation template

5. **Train Team**
   - Schedule training session
   - Review evaluation process
   - Practice with sample tickets
   - Address questions

6. **Initial Baseline**
   - Evaluate sample of existing tickets
   - Establish baseline metrics
   - Set improvement targets

**Output:** Fully configured project ready for evaluations

---

### Sprint Retrospective Analysis

**Time Required:** 45 minutes

**Steps:**
1. **Filter Sprint Data**
   - Set date range to sprint duration
   - Filter by sprint tag/version

2. **Generate Sprint Metrics**
   - Total evaluations
   - Average quality score
   - Score by ticket type
   - Criteria breakdown

3. **Review vs Previous Sprint**
   - Quality trend
   - Volume changes
   - Performance shifts

4. **Identify Patterns**
   - Common issues
   - Recurring feedback
   - Systematic problems

5. **Prepare Retrospective Input**
   - Quality highlights
   - Improvement areas
   - Specific examples
   - Actionable recommendations

**Output:** Quality insights for sprint retrospective

---

## Special Workflows

### Onboarding New Evaluator

**Time Required:** 2 hours

**Steps:**
1. **System Access**
   - Grant necessary permissions
   - Provide login credentials
   - Verify app access

2. **Training Session**
   - Review evaluation criteria
   - Demonstrate scoring process
   - Practice on sample tickets
   - Discuss feedback guidelines

3. **Shadowing Period**
   - Pair with experienced evaluator
   - Review 10 tickets together
   - Discuss scoring decisions
   - Compare scores

4. **Independent Practice**
   - Evaluate 5 tickets solo
   - Review with mentor
   - Adjust scoring approach
   - Refine feedback style

5. **Calibration Check**
   - Compare to team averages
   - Identify any bias
   - Provide guidance
   - Schedule follow-up

**Output:** Trained and calibrated evaluator

---

### Quality Crisis Response

**When:** Quality scores drop significantly

**Time Required:** Varies

**Steps:**
1. **Immediate Analysis**
   - Identify when decline started
   - Check affected areas
   - Review recent changes
   - Analyze error patterns

2. **Root Cause Investigation**
   - Interview team members
   - Review process changes
   - Check external factors
   - Analyze specific failures

3. **Emergency Meeting**
   - Present findings
   - Discuss immediate actions
   - Assign responsibilities
   - Set recovery timeline

4. **Implement Fixes**
   - Address root causes
   - Increase evaluation frequency
   - Provide additional training
   - Add quality checks

5. **Monitor Recovery**
   - Daily quality checks
   - Track improvement
   - Adjust actions as needed
   - Report progress

**Output:** Quality recovery plan and execution

---

### Annual Performance Review

**Time Required:** 4-5 hours

**Steps:**
1. **Compile Annual Data**
   - Export full year's evaluations
   - Generate annual statistics
   - Create trend analysis
   - Calculate improvements

2. **Individual Reviews**
   For each agent:
   - Annual average score
   - Improvement trajectory
   - Strengths and weaknesses
   - Peer comparison
   - Development recommendations

3. **Team Analysis**
   - Team achievements
   - Quality milestones
   - Process improvements
   - Training effectiveness

4. **Strategic Planning**
   - Next year's quality targets
   - Resource requirements
   - Training plans
   - Tool enhancements

5. **Documentation**
   - Individual review documents
   - Team performance report
   - Strategic plan
   - Budget requirements

**Output:** Comprehensive annual review package

---

## Automation Workflows

### Automated Daily Report

**Setup Time:** 30 minutes

**Steps:**
1. **Configure Schedule**
   - Go to Settings → Automation
   - Set daily trigger time
   - Select recipients

2. **Define Report Content**
   - Yesterday's evaluations
   - Quality score
   - Top/bottom performers
   - Alerts and exceptions

3. **Set Filters**
   - Projects to include
   - Minimum evaluation threshold
   - Score ranges for alerts

4. **Test Configuration**
   - Run test report
   - Verify data accuracy
   - Check formatting
   - Confirm delivery

**Output:** Automated daily quality reports

---

### Alert Configuration

**Setup Time:** 20 minutes

**Steps:**
1. **Define Alert Conditions**
   - Score below threshold
   - Trend declining for X days
   - No evaluations for Y hours
   - Bias detection

2. **Set Recipients**
   - Team leads for performance alerts
   - Evaluators for bias warnings
   - Admins for system issues

3. **Configure Notifications**
   - Email notifications
   - In-app alerts
   - Slack integration (if available)

4. **Test Alerts**
   - Trigger test conditions
   - Verify delivery
   - Check message content

**Output:** Proactive alert system

---

## Best Practice Workflows

### Continuous Improvement Cycle

**Frequency:** Ongoing

**Steps:**
1. **Measure**
   - Current quality metrics
   - Baseline performance
   - Problem areas

2. **Analyze**
   - Root cause analysis
   - Pattern identification
   - Impact assessment

3. **Improve**
   - Implement changes
   - Provide training
   - Adjust processes

4. **Control**
   - Monitor results
   - Maintain improvements
   - Prevent regression

5. **Standardize**
   - Document successful changes
   - Update procedures
   - Train team

**Output:** Sustained quality improvement

---

## Quick Reference Workflows

### 5-Minute Quality Check
1. Open Analytics Dashboard
2. Check today's average score
3. Review any alerts
4. Note significant changes
5. Take action if needed

### Ticket Evaluation (3 minutes)
1. Open ticket
2. Read resolution
3. Score criteria (30 seconds each)
4. Add feedback (1 minute)
5. Submit

### Weekly Report Generation
1. Set date range (last 7 days)
2. Apply team filter
3. Export to Excel
4. Add to shared drive
5. Send notification

---

*For more details on specific features, see the [Feature Guide](./Feature-Guide.md)*

