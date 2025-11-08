 # Frequently Asked Questions (FAQ)

## General Questions

### What is the Quality Compass?
The Quality Compass is a quality assurance evaluation system integrated directly into Jira. It allows teams to systematically evaluate ticket quality, track performance metrics, and generate comprehensive analytics reports.

### Who can use the Quality Compass?
Anyone with access to your Jira project can view analytics (with appropriate permissions). Creating evaluations typically requires "Edit Issues" permission, while administrative functions require project admin rights.

### Is training required to use the app?
Basic usage requires no formal training - the interface is intuitive and includes helpful tooltips. However, we recommend reviewing this documentation for best practices and advanced features.

---

## Evaluation Questions

### How do I create an evaluation?
1. Navigate to any Jira ticket
2. Click the "Evaluate" button
3. Score each criterion (0-10)
4. Add feedback
5. Submit the evaluation

### Can I edit an evaluation after submitting?
Yes. Evaluations can be edited at any time, provided the ticket is still open and you have the necessary permissions.

### What happens if I make a mistake in scoring?
You can edit the evaluation later. If the ticket is already closed or you lack permissions, contact an administrator for assistance.

### Why can't I see the Evaluate button?
Possible reasons:
- You don't have "Edit Issues" permission
- The app isn't installed for this project

### How many evaluations can one ticket have?
Tickets can have multiple evaluations; analytics use the most recent per ticket for charts. Raw tables may show full history where applicable.

---

## Scoring Questions

### What does each score mean?
- **0-4.9**: Poor performance, significant issues
- **5-7.9**: Average performance, room for improvement
- **8-10**: Good performance, minor issues only

### Should I use the full scale (0-10)?
Yes! Using the full scale provides better differentiation and more actionable data. Avoid clustering all scores around 7-8.

### How do I ensure consistent scoring?
- Use the scoring guide for each criterion
- Attend calibration sessions with your team
- Review the evaluator analytics for your scoring patterns
- Compare your scores with team averages

### What if I disagree with another evaluator's score?
Discuss it in your team's calibration meeting. The app tracks evaluator patterns to identify and address inconsistencies.

---

## Analytics Questions

### How often is analytics data updated?
- Real-time for new evaluations
- Cached results are shown instantly on revisit; background refresh updates automatically
- Manual refresh available via the Refresh/Apply Filters button

### What does the trend percentage mean?
The trend shows the change in average score:
- Positive percentage = improving quality
- Negative percentage = declining quality
- Based on recent vs. historical performance

### Why don't I see any data in analytics?
- Check your date range filter
- Ensure evaluations exist for the selected period
- Verify you have permission to view analytics
- Try refreshing the page

### How is the "Most Improved" agent determined?
Based on the highest positive trend percentage over the selected time period, with a minimum number of evaluations for statistical significance.

### Can I export analytics data?
Yes! Click the Export button and choose either CSV or Excel format. You can export filtered or complete datasets.

---

## Performance Questions

### How is agent performance calculated?
Agent performance is based on:
- Average evaluation score
- Number of evaluations
- Trend over time
- Individual criteria scores

### What does "bias" mean in evaluator analytics?
Bias indicates if an evaluator consistently scores higher (lenient) or lower (strict) than the team average by 2.5+ points.

### How can I improve my performance score?
- Review feedback from evaluations
- Focus on your weakest criteria
- Ask evaluators for specific improvement suggestions
- Track your progress over time

### Why is my trend showing as declining when I'm improving?
Trends are calculated based on recent performance vs. historical baseline. Short-term fluctuations are normal. Focus on long-term trends.

---

## Technical Questions

### Is my data secure?
Yes, all data is:
- Encrypted at rest and in transit
- Stored in Atlassian's secure Forge platform
- Access-controlled based on Jira permissions
- Regularly backed up

### Can I integrate this with other tools?
The app provides data export capabilities. Additional API endpoints and integrations are planned for future releases.

### What browsers are supported?
Modern versions of:
- Chrome (recommended)
- Firefox
- Safari
- Edge

### Does the app work on mobile devices?
Yes, the app is fully responsive and works on tablets and smartphones with optimized mobile views.

### What happens to the data if we uninstall the app?
Data remains in Forge storage until manually deleted or purged. Export before uninstall. Contact admin/support for immediate deletion if required.

---

## Troubleshooting Questions

### Why is the app running slowly?
- First load of complex tabs (Evaluator/Advanced) can take longer. Re-opens should be much faster due to caching.
- Reduce the date range for large datasets
- Click Refresh to fetch latest; otherwise the app may show cached data first and update in the background
- Check your internet connection / try a different browser
- Contact support if issues persist

### I'm getting an error message. What should I do?
1. Note the exact error message
2. Try refreshing the page
3. Clear browser cache and cookies
4. Check if others are experiencing the same issue
5. Contact support with error details

### Why can't I export data?
- Check if pop-ups are blocked
- Try a different export format
- Ensure the dataset isn't too large

### The charts aren't displaying correctly.
- Refresh the page
- Check browser compatibility
- Disable browser extensions temporarily
- Update your browser to the latest version

### I can't find a specific evaluation.
- Check your filters
- Expand the date range
- Search by ticket ID
- Verify the evaluation was submitted successfully

---

## Administrative Questions

### How do I add custom evaluation criteria?
Evaluation criteria are currently pre-configured and cannot be modified through the UI. Contact your development team if you need to change the criteria.

### Can I restrict who can create evaluations?
Yes, through Jira's permission schemes. Users need "Edit Issues" permission to create evaluations.

### How do I manage evaluator access?
Evaluator access is controlled through Jira project permissions. Administrators can manage project access and roles under Settings.

### Can I delete evaluations?
No, evaluations cannot be deleted once created. This ensures data integrity and maintains a complete evaluation history.

### How do I handle GDPR requests?
GDPR compliance features are partially implemented:
- Data export is available through the analytics export functionality
- For data deletion requests, contact your development team
- Note: The GDPR service code exists but requires manual intervention

---

## Best Practices Questions

### How often should we evaluate tickets?
Best practice is to evaluate tickets as soon as they're resolved, while context is fresh. Avoid batch evaluations.

### Should every ticket be evaluated?
Ideally yes, but at minimum:
- All customer-reported issues
- High-priority tickets
- Complex technical tasks
- A random sample of routine tickets

### How do we prevent gaming the system?
- Regular calibration sessions
- Monitor evaluator analytics for bias
- Rotate evaluators
- Track evaluation history
- Focus on trends, not individual scores

### What's the best way to provide feedback?
- Be specific and actionable
- Balance positive and constructive feedback
- Reference specific examples
- Suggest improvements
- Keep it professional

### How often should we review analytics?
- Team leads: Weekly
- Individuals: Bi-weekly
- Quarterly deep-dives for trends

---

## Getting Help

### Where can I find more documentation?
- [Getting Started Guide](./Getting-Started.md)
- [Feature Guide](./Feature-Guide.md)
- [Workflows Guide](./Workflows.md)
- [Technical Documentation](../Technical/README.md)

### How do I report a bug?
Raise a ticket in the Service Desk Jira project with:
- Description of the issue
- Steps to reproduce
- Screenshots if applicable
- Browser and version
- Time and date of occurrence

### Where can I suggest new features?
Submit feature requests to:
- Email: pedro.pinto@sportingtech.com
- Include use case and expected benefits

### Is there a user community?
Not yet, but there will be in the future.

### How do I get admin access?
Contact your Jira administrator or project lead. Admin access is granted based on role and responsibilities.

---

## Quick Troubleshooting Checklist

If something isn't working, try these steps in order:

1. ✅ Refresh the page
2. ✅ Clear browser cache
3. ✅ Check your permissions
4. ✅ Verify filters aren't hiding data
5. ✅ Try a different browser
6. ✅ Check if others have the same issue
7. ✅ Contact support if unresolved

---

*Can't find your answer? Contact pedro.pinto@sportingtech.com*
