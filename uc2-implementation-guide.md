# UC2: Daily Internal Reminder Workflow - Implementation Guide

## Overview
This guide implements UC2, building on the existing UC1 WhatsApp Enquiry Auto-Handler. The Daily Internal Reminder Workflow sends scheduled reminders at 6 PM weekdays about unresolved enquiries to support staff.

## Prerequisites
- UC1 WhatsApp Enquiry Auto-Handler must be implemented and running
- `enquiry_log` custom object must exist (created in UC1)
- Support Team user group must be configured
- SleekFlow Flow Builder with scheduling capabilities
- Admin permissions to create scheduled workflows

## Phase 1: Verify UC1 Dependencies

### Step 1: Confirm Custom Object Exists
1. Navigate to **Settings** → **Custom Objects**
2. Verify "Enquiry Log" object exists with these key fields:
   - `status` (with values: Open, In Progress, Resolved, Closed)
   - `timestamp` (datetime field)
   - `customer_name`, `phone_number`, `enquiry_details`

### Step 2: Confirm Support Team Configuration
1. Navigate to **Settings** → **Teams & Users**
2. Verify "Support Team" user group exists
3. Ensure team has appropriate members assigned

## Phase 2: Create Daily Reminder Workflow

### Step 1: Create New Scheduled Flow
1. Navigate to **Flow Builder** → **Create New Flow**
2. Set flow name: "Daily Internal Enquiry Reminder"
3. Set description: "Sends daily reminders at 6 PM about unresolved enquiries to support staff"
4. Enable the flow

### Step 2: Configure Schedule Trigger
1. Add **"Schedule"** trigger
2. Configure trigger settings:
   - **Schedule Type**: Daily
   - **Time**: 18:00 (6:00 PM)
   - **Timezone**: UTC+8 (adjust to your company timezone)
   - **Days**: Monday through Friday only
   - **Label**: "Daily 6PM Weekday Trigger"

### Step 3: Add Weekday Condition (Optional Safeguard)
1. Add **"If/Else"** condition block
2. Configure condition:
   - **Field**: Current date/time
   - **Operator**: "day of week not in"
   - **Values**: Saturday, Sunday
   - **Label**: "Skip weekends"
   
*Note: This provides an extra safeguard even if trigger is configured for weekdays only*

### Step 4: Search for Unresolved Enquiries
1. Add **"Search Custom Objects"** action
2. Configure search:
   - **Object**: "enquiry_log"
   - **Filters**:
     ```
     status ≠ "resolved" AND
     status ≠ "closed" AND
     timestamp < (now - 24 hours)
     ```
   - **Sort**: timestamp descending
   - **Limit**: 50 records
   - **Label**: "Find unresolved enquiries"

### Step 5: Check Search Results
1. Add **"If/Else"** condition block
2. Configure condition:
   - **Field**: `search_results.count`
   - **Operator**: "greater than"
   - **Value**: 0
   - **Label**: "Check if enquiries found"

### Step 6A: Build Summary (if enquiries found)
1. Add **"Set Variables"** action
2. Configure variables:
   ```
   contact_count = {{search_results.count}}
   top_contacts = SLICE(SORT(search_results, 'timestamp', 'desc'), 0, 5)
   ```

### Step 6B: Send Reminder Message
1. Add **"Send Internal Message"** action
2. Configure message:
   - **Recipients**: Support Team
   - **Template**:
   ```
   📋 Daily Enquiry Reminder

   🔍 There are {{contact_count}} contacts with unresolved enquiries needing attention.

   📊 Top 5 by recency:
   {{#each top_contacts}}
   • {{customer_name}} ({{phone_number}}) - {{LEFT enquiry_details 50}}...
     Status: {{status}} | Age: {{TIME_AGO timestamp}}
   {{/each}}

   ⏰ Please review and address these enquiries ASAP.

   💡 Tip: Focus on enquiries older than 24 hours first.

   ---
   🤖 Automated reminder sent at {{current_datetime}}
   ```

### Step 7: Handle Zero Enquiries Case
1. Add **"Send Internal Message"** action (in "No" branch)
2. Configure message:
   - **Recipients**: Support Team
   - **Template**:
   ```
   ✅ Daily Enquiry Check

   Great news! All enquiries have been addressed - no unresolved items found.

   🎉 Keep up the excellent work!

   ---
   🤖 Automated check completed at {{current_datetime}}
   ```

### Step 8: Add Error Handling
1. Add **"Error Handler"** block after search action
2. Configure error condition: "Search Failed", "API Timeout", "Permission Denied"
3. Add **"Send Internal Message"** action:
   - **Recipients**: Support Team + System Admin
   - **Template**:
   ```
   ⚠️ Daily Reminder Error

   Unable to fetch enquiry data for daily reminder.

   Error: {{error.message}}
   Time: {{current_datetime}}

   🔧 Please check the system and run manual enquiry review.

   📋 Manual Action Required:
   1. Check SleekFlow enquiry logs
   2. Review unresolved items
   3. Contact system admin if issue persists
   ```

### Step 9: Configure Flow Settings
1. Set **Flow Priority**: Medium
2. Enable **Flow Logging** for debugging
3. Set **Execution Timeout**: 120 seconds (allows for larger searches)
4. Save and activate the flow

## Phase 3: Testing and Validation

### Step 1: Create Test Data
1. Manually create a few test enquiry logs:
   - Some with status "open" and timestamp > 24 hours ago
   - Some with status "resolved" (should be excluded)
   - Some with recent timestamps < 24 hours (should be excluded)

### Step 2: Manual Test Trigger
1. Use Flow Builder's "Test Flow" feature
2. Manually trigger the flow to simulate the 6 PM schedule
3. Verify correct message is sent based on test data

### Step 3: Test Edge Cases
- **Weekend Test**: Manually trigger on Saturday/Sunday (should skip)
- **No Data Test**: Ensure all enquiry_logs are resolved/closed, trigger flow
- **Error Test**: Temporarily remove permissions to trigger error handler

### Step 4: Schedule Validation
1. Enable the flow and wait for next 6 PM trigger
2. Verify message is received by Support Team
3. Check flow execution logs for any issues

## Phase 4: Advanced Configuration Options

### Alternative Time Configurations
```json
// For different timezones
"timezone": "America/New_York"  // EST/EDT
"timezone": "Europe/London"     // GMT/BST
"timezone": "Asia/Tokyo"        // JST

// For different times
"time": "17:30"  // 5:30 PM
"time": "09:00"  // 9:00 AM morning reminder
```

### Enhanced Filtering Options
```json
// Include only specific enquiry types
{
  "field": "enquiry_type",
  "operator": "in",
  "values": ["product_info", "technical_support"]
}

// Prioritize by assigned staff
{
  "field": "assigned_staff", 
  "operator": "is_not_null"
}
```

### Multiple Reminder Recipients
```json
"recipients": [
  "Support Team",
  "Support Manager", 
  "sales@company.com"
]
```

## Phase 5: Integration with UC1

### Shared Dependencies
- Both workflows use the same `enquiry_log` custom object
- Both reference "Support Team" user group
- UC2 depends on data created by UC1

### Monitoring Integration
- UC1 creates enquiry logs
- UC2 monitors and reports on unresolved logs
- Together they provide end-to-end enquiry lifecycle management

### Performance Considerations
- UC1 runs on every incoming WhatsApp message
- UC2 runs once daily at 6 PM
- Both should be monitored for execution time and success rates

## Troubleshooting Common Issues

### Issue 1: No Messages Received
**Symptoms**: Flow executes but no internal messages sent
**Solutions**:
- Check "Support Team" group membership
- Verify internal messaging permissions
- Review flow execution logs

### Issue 2: Incorrect Enquiry Count
**Symptoms**: Count doesn't match manual review
**Solutions**:
- Check filter conditions (24-hour threshold)
- Verify status field values match exactly
- Review timezone settings

### Issue 3: Weekend Messages
**Symptoms**: Reminders sent on weekends
**Solutions**:
- Verify trigger weekday configuration
- Check additional weekday condition logic
- Review timezone vs server time differences

### Issue 4: Search Timeout
**Symptoms**: Error handler triggered for large datasets
**Solutions**:
- Reduce search limit from 50 to 25
- Add pagination logic for large result sets
- Increase flow timeout setting

## Monitoring and Maintenance

### Weekly Review
- Check flow execution success rate
- Review feedback from support team on reminder usefulness
- Adjust time or frequency if needed

### Monthly Optimization
- Analyze enquiry age patterns
- Update message templates based on team feedback
- Review and update error handling

### Performance Metrics
- Flow execution time
- Search result accuracy
- Staff response to reminders
- Reduction in unresolved enquiry age

## Next Steps
1. Complete UC2 implementation following this guide
2. Run through testing checklist
3. Deploy to production
4. Monitor for first week
5. Gather feedback from support team
6. Consider additional reminder types (urgent enquiries, VIP customers)

---

**Document created**: 2025-01-20  
**Version**: 1.0  
**Dependencies**: UC1 WhatsApp Enquiry Auto-Handler  
**Integration**: SleekFlow Flow Builder Scheduling
