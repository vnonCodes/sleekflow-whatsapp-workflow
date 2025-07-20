# UC2 Daily Internal Reminder Workflow - Troubleshooting Guide

## Common Issues and Solutions

### Issue 1: No Reminder Messages Received

**Symptoms:**
- Flow shows as executing successfully in logs
- Support team reports no reminder messages received
- No error messages in flow logs

**Possible Causes & Solutions:**

#### Cause A: Support Team Configuration
- **Check**: Navigate to Settings → Teams & Users
- **Verify**: "Support Team" group exists and has members
- **Solution**: Add team members or verify group name matches exactly

#### Cause B: Internal Messaging Permissions
- **Check**: Team members have permission to receive internal messages
- **Verify**: Test sending a manual internal message to the team
- **Solution**: Update user permissions or group settings

#### Cause C: Weekend/Holiday Execution
- **Check**: Flow logs for "Skipped daily reminder - weekend" message
- **Verify**: Current day is Monday-Friday in configured timezone
- **Solution**: Normal behavior - flow should skip weekends

### Issue 2: Wrong Enquiry Count in Reminders

**Symptoms:**
- Reminder shows different count than manual review
- Old resolved enquiries appear in results
- Recent enquiries missing from results

**Possible Causes & Solutions:**

#### Cause A: Status Field Mismatch
- **Check**: Enquiry log status field values exactly match filter
- **Common Issue**: "Resolved" vs "resolved" (case sensitivity)
- **Solution**: Update filter values to match exact case, or use case-insensitive operators

#### Cause B: Timestamp Format Issues  
- **Check**: timestamp field format in enquiry_log records
- **Common Issue**: Timezone differences causing incorrect age calculation
- **Solution**: Verify timezone settings match between UC1 and UC2

#### Cause C: 24-Hour Threshold Calculation
- **Check**: Flow execution time vs enquiry timestamps
- **Debug**: Manually calculate 24 hours ago from flow execution time
- **Solution**: Adjust threshold or use relative time functions

### Issue 3: Flow Execution Timeout

**Symptoms:**
- Error message: "Flow execution timed out"
- Flow stops at search custom objects step
- Large number of enquiry logs in database

**Solutions:**
1. **Reduce Search Limit**: Change from 50 to 25 records
2. **Increase Timeout**: Set flow timeout to 180 seconds
3. **Add Pagination**: Implement multiple smaller searches
4. **Optimize Filters**: Add more specific filters to reduce result set

### Issue 4: Schedule Not Triggering

**Symptoms:**
- No flow execution logs at scheduled time
- Flow appears enabled but doesn't run
- Manual trigger works fine

**Possible Causes & Solutions:**

#### Cause A: Schedule Configuration
- **Check**: Schedule trigger settings
- **Common Issues**:
  - Wrong timezone selected
  - Incorrect time format (use 24-hour format)
  - Days of week not properly configured
- **Solution**: Reconfigure schedule with correct settings

#### Cause B: Flow Status
- **Check**: Flow is enabled and active
- **Verify**: No conflicting workflows with same schedule
- **Solution**: Enable flow or resolve scheduling conflicts

#### Cause C: System Maintenance
- **Check**: SleekFlow system status during scheduled time
- **Verify**: Other scheduled flows are working
- **Solution**: Contact SleekFlow support if system-wide issue

### Issue 5: Error Handler Triggering Frequently

**Symptoms:**
- Frequent error notifications to Support Team + Admin
- Error messages about search failures
- Inconsistent flow execution

**Diagnostic Steps:**
1. **Review Error Messages**: Check specific error details
2. **Check Permissions**: Verify flow has access to enquiry_log object
3. **Database Status**: Ensure custom object is accessible
4. **Load Testing**: Check if system is under high load

**Solutions:**
- **Permission Fix**: Grant flow appropriate object access
- **Retry Logic**: Add retry mechanism with delays
- **Rate Limiting**: Space out search operations
- **Fallback Data**: Use cached results when possible

### Issue 6: Message Formatting Issues

**Symptoms:**
- Variable placeholders showing as {{variable}} in messages
- Truncated names or phone numbers
- Special characters not displaying correctly

**Solutions:**

#### Variable Substitution Problems
```
Problem: {{contact_count}} displays literally
Solution: Check variable name matches exactly in template
```

#### Text Truncation Issues
```
Problem: Long names get cut off
Solution: Adjust LEFT() function parameter: LEFT(enquiry_details, 75)
```

#### Special Character Encoding
```
Problem: Emojis not displaying
Solution: Ensure UTF-8 encoding in message templates
```

### Issue 7: Integration Issues with UC1

**Symptoms:**
- UC2 not finding enquiries created by UC1
- Data inconsistency between workflows
- Status updates not reflected

**Diagnostic Steps:**
1. **Verify UC1 Operation**: Ensure UC1 is creating enquiry logs
2. **Check Data Format**: Compare field formats between UC1 and UC2
3. **Test Data Flow**: Create test enquiry via UC1, verify UC2 can find it

**Solutions:**
- **Field Mapping**: Ensure UC1 and UC2 use same field names
- **Status Synchronization**: Verify status values match exactly
- **Timestamp Compatibility**: Use consistent datetime formats

## Performance Optimization

### Large Dataset Handling

**For organizations with >100 enquiries/day:**

```json
// Optimized search configuration
{
  "limit": 25,
  "filters": [
    {"field": "status", "operator": "in", "values": ["open", "in_progress"]},
    {"field": "timestamp", "operator": "older_than", "value": "48_hours"},
    {"field": "priority", "operator": "equals", "value": "high"}
  ]
}
```

### Memory Usage Optimization

**For flows timing out:**

1. **Process in Batches**: 
   - Search 25 records at a time
   - Use multiple search actions if needed
   - Combine results in variables

2. **Selective Fields**:
   - Only fetch required fields
   - Avoid loading large text fields unnecessarily

3. **Conditional Processing**:
   - Skip processing if count is 0
   - Early exit conditions

## Monitoring and Alerts

### Setting Up Monitoring

**Key Metrics to Track:**
- Daily execution success rate (target: 100%)
- Average execution time (target: <10 seconds) 
- Error rate (target: 0%)
- Message delivery time (target: <30 seconds)

**Recommended Alerts:**
1. **Flow Failure Alert**: If flow fails 2 days in a row
2. **Performance Alert**: If execution time >60 seconds
3. **Data Alert**: If enquiry count changes by >50% day-over-day

### Flow Health Check

**Weekly Review Checklist:**
- [ ] Flow execution logs show consistent success
- [ ] No error notifications received
- [ ] Support team feedback is positive
- [ ] Enquiry counts align with business volume
- [ ] Performance metrics within targets

## Emergency Procedures

### If Flow Completely Fails
1. **Immediate**: Disable the flow to prevent error spam
2. **Notify**: Alert Support Team via alternative channel
3. **Manual Process**: Generate manual enquiry report
4. **Debug**: Review logs and identify root cause
5. **Fix**: Implement fix and test thoroughly
6. **Re-enable**: Only after confirming fix works

### If Wrong Data is Sent
1. **Send Correction**: Manual message to Support Team with correct information
2. **Investigate**: Review search filters and data quality
3. **Prevent**: Fix root cause to avoid recurrence
4. **Document**: Record issue and resolution for future reference

## Getting Additional Help

### SleekFlow Support Channels
- **Documentation**: SleekFlow Flow Builder help center
- **Support Ticket**: For system-level issues
- **Community Forum**: For configuration questions

### Escalation Process
1. **Level 1**: Check this troubleshooting guide
2. **Level 2**: Review flow logs and configuration  
3. **Level 3**: Contact system administrator
4. **Level 4**: Open SleekFlow support ticket

### Information to Provide When Seeking Help
- Flow configuration export
- Recent execution logs
- Error messages (exact text)
- Screenshots of issue
- Steps to reproduce
- Business impact assessment

---

**Last Updated**: 2025-01-20  
**Version**: 1.0  
**Related Workflows**: UC1 WhatsApp Enquiry Auto-Handler
