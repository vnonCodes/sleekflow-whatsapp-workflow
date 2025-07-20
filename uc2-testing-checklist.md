# UC2 Daily Internal Reminder Workflow - Testing Checklist

## Overview
Comprehensive testing guide for the Daily Internal Reminder Workflow that integrates with UC1 to send scheduled reminders about unresolved enquiries.

## Pre-Testing Setup

### ✅ Environment Verification
- [ ] UC1 WhatsApp Enquiry Auto-Handler is active and working
- [ ] `enquiry_log` custom object exists and is populated
- [ ] "Support Team" user group is configured with test members
- [ ] Admin access to Flow Builder and Custom Objects
- [ ] Test internal messaging channels are set up

### ✅ Test Data Preparation
Create the following test enquiry log entries:

#### Test Entry 1: Unresolved Old Enquiry
- [ ] Status: "open" 
- [ ] Timestamp: 48 hours ago
- [ ] Customer Name: "Test Customer A"
- [ ] Phone: "+1234567890"
- [ ] Enquiry Details: "Need help with product pricing and availability"

#### Test Entry 2: Recent Open Enquiry (Should be excluded)
- [ ] Status: "open"
- [ ] Timestamp: 12 hours ago  
- [ ] Customer Name: "Test Customer B"
- [ ] Phone: "+1234567891"
- [ ] Enquiry Details: "Question about shipping times"

#### Test Entry 3: Resolved Old Enquiry (Should be excluded)
- [ ] Status: "resolved"
- [ ] Timestamp: 72 hours ago
- [ ] Customer Name: "Test Customer C" 
- [ ] Phone: "+1234567892"
- [ ] Enquiry Details: "Return policy question - resolved"

#### Test Entry 4: In Progress Old Enquiry
- [ ] Status: "in_progress"
- [ ] Timestamp: 36 hours ago
- [ ] Customer Name: "Test Customer D"
- [ ] Phone: "+1234567893"
- [ ] Enquiry Details: "Technical support for app installation"

#### Test Entry 5: Closed Old Enquiry (Should be excluded)
- [ ] Status: "closed"
- [ ] Timestamp: 60 hours ago
- [ ] Customer Name: "Test Customer E"
- [ ] Phone: "+1234567894"
- [ ] Enquiry Details: "General inquiry - closed"

## Core Functionality Testing

### Test Case 1: Manual Flow Execution with Data
**Objective**: Verify flow executes correctly with unresolved enquiries

**Steps**:
1. [ ] Navigate to Flow Builder → Daily Internal Enquiry Reminder
2. [ ] Click "Test Flow" or "Manual Trigger"
3. [ ] Execute the flow during weekday hours

**Expected Results**:
- [ ] Flow executes successfully
- [ ] Search finds 2 unresolved enquiries (Test Entry 1 & 4)
- [ ] Internal message is sent to Support Team
- [ ] Message contains correct count: "There are 2 contacts with unresolved enquiries"
- [ ] Message shows top contacts with names and phone numbers
- [ ] Message includes enquiry age ("2 days ago", "1 day ago")

**Actual Results**:
```
Count Found: _____
Message Received: Yes/No
Message Content Correct: Yes/No
Notes: ___________
```

### Test Case 2: No Unresolved Enquiries
**Objective**: Test behavior when all enquiries are resolved

**Setup**:
1. [ ] Change all test enquiry statuses to "resolved" or "closed"
2. [ ] Execute manual flow trigger

**Expected Results**:
- [ ] Flow executes successfully
- [ ] Search returns 0 results
- [ ] "All enquiries addressed" message is sent
- [ ] Message contains congratulatory text

**Actual Results**:
```
Message Type: Success/Error
Message Content: ___________
Notes: ___________
```

### Test Case 3: Weekend Skip Logic
**Objective**: Verify flow skips execution on weekends

**Steps**:
1. [ ] If testing on weekday, temporarily modify weekday condition to exclude current day
2. [ ] Execute manual flow trigger
3. [ ] Check flow logs

**Expected Results**:
- [ ] Flow starts but ends silently
- [ ] No internal messages sent
- [ ] Flow log shows "Skipped daily reminder - weekend"

**Actual Results**:
```
Flow Outcome: Skipped/Executed
Log Message: ___________
```

### Test Case 4: Error Handling - Search Failure
**Objective**: Test error handling when search fails

**Setup**:
1. [ ] Temporarily remove permissions for flow to access custom objects
2. [ ] Execute manual flow trigger

**Expected Results**:
- [ ] Search action fails
- [ ] Error handler triggers
- [ ] Error notification sent to Support Team + System Admin
- [ ] Message includes error details and manual action steps

**Actual Results**:
```
Error Caught: Yes/No
Error Message Sent: Yes/No
Recipients Correct: Yes/No
```

## Schedule Testing

### Test Case 5: Schedule Trigger Verification
**Objective**: Verify scheduled execution works correctly

**Note**: This test requires waiting for actual scheduled time or temporarily modifying schedule

**Steps**:
1. [ ] Set flow schedule to trigger in next 10 minutes (temporary test)
2. [ ] Wait for scheduled execution
3. [ ] Check for internal message receipt
4. [ ] Review flow execution logs

**Expected Results**:
- [ ] Flow triggers automatically at scheduled time
- [ ] Appropriate message is sent based on current enquiry data
- [ ] Flow logs show successful execution

**Actual Results**:
```
Triggered on Time: Yes/No
Message Received: Yes/No  
Execution Time: ___________
```

## Advanced Testing

### Test Case 6: Large Dataset Performance
**Objective**: Test performance with many enquiries

**Setup**:
1. [ ] Create 20+ test enquiry logs (mix of old unresolved and resolved)
2. [ ] Execute flow and measure execution time

**Expected Results**:
- [ ] Flow completes within 120 seconds timeout
- [ ] Correct count of unresolved enquiries
- [ ] Top 5 contacts displayed correctly
- [ ] No timeout errors

**Actual Results**:
```
Execution Time: _____ seconds
Enquiries Found: _____
Top 5 Correct: Yes/No
```

### Test Case 7: Timezone Handling
**Objective**: Verify timezone configuration works correctly

**Steps**:
1. [ ] Note current system time and configured timezone
2. [ ] Trigger flow and check timestamp in message
3. [ ] Verify times align with expected timezone

**Expected Results**:
- [ ] Message timestamp matches expected timezone
- [ ] Enquiry ages calculated correctly relative to timezone

**Actual Results**:
```
Expected Time: ___________
Actual Time: ___________
Timezone Correct: Yes/No
```

### Test Case 8: Message Template Formatting
**Objective**: Verify message formatting and variables

**Steps**:
1. [ ] Create enquiries with various name lengths and special characters
2. [ ] Execute flow and examine message formatting

**Expected Results**:
- [ ] Names display correctly without truncation issues
- [ ] Phone numbers format properly
- [ ] Enquiry details truncate at 50 characters with "..."
- [ ] Emojis and formatting display correctly
- [ ] No variable substitution errors (no {{}} visible)

**Actual Results**:
```
Formatting Issues: Yes/No
Variable Errors: Yes/No
Special Characters: OK/Issues
```

## Integration Testing with UC1

### Test Case 9: End-to-End Integration
**Objective**: Test complete workflow from UC1 creation to UC2 reminder

**Steps**:
1. [ ] Use UC1 to create a new enquiry (send WhatsApp message)
2. [ ] Wait 25+ hours or modify enquiry timestamp
3. [ ] Execute UC2 flow
4. [ ] Verify enquiry appears in reminder

**Expected Results**:
- [ ] UC1 creates enquiry log correctly
- [ ] UC2 finds enquiry after aging threshold
- [ ] Enquiry details match between UC1 creation and UC2 display

**Actual Results**:
```
UC1 Creation: Success/Fail
UC2 Detection: Success/Fail
Data Consistency: Good/Issues
```

## Edge Case Testing

### Test Case 10: Empty Database
**Objective**: Test behavior with no enquiry logs

**Setup**:
1. [ ] Remove all enquiry_log records (backup first!)
2. [ ] Execute flow

**Expected Results**:
- [ ] Flow executes without errors
- [ ] "All enquiries addressed" message sent
- [ ] No system errors

### Test Case 11: Malformed Data
**Objective**: Test resilience with corrupted enquiry data

**Setup**:
1. [ ] Create enquiry with missing customer_name
2. [ ] Create enquiry with null timestamp
3. [ ] Execute flow

**Expected Results**:
- [ ] Flow handles missing data gracefully
- [ ] Error handler catches data issues if needed
- [ ] No flow crashes

### Test Case 12: Permission Changes
**Objective**: Test behavior when permissions change

**Setup**:
1. [ ] Remove Support Team from internal messaging
2. [ ] Execute flow
3. [ ] Restore permissions

**Expected Results**:
- [ ] Appropriate error handling
- [ ] Fallback notification or graceful failure
- [ ] System admin notification if configured

## Performance and Monitoring

### Test Case 13: Resource Usage
**Objective**: Monitor resource consumption during execution

**Metrics to Track**:
- [ ] Flow execution time: _____ seconds
- [ ] Memory usage: Normal/High
- [ ] API calls made: _____ 
- [ ] Search query performance: _____ ms

### Test Case 14: Concurrent Execution
**Objective**: Test behavior if multiple instances run

**Setup**:
1. [ ] Manually trigger flow while it's already running
2. [ ] Check for race conditions or conflicts

**Expected Results**:
- [ ] No duplicate messages sent
- [ ] No data corruption
- [ ] Appropriate queuing or collision handling

## Final Validation

### ✅ Production Readiness Checklist
- [ ] All test cases pass
- [ ] Error scenarios handled appropriately  
- [ ] Performance within acceptable limits
- [ ] Support team trained on receiving reminders
- [ ] Monitoring and alerting configured
- [ ] Documentation complete
- [ ] Rollback plan prepared

### ✅ Go-Live Verification
- [ ] Schedule configured for production timing (6:00 PM weekdays)
- [ ] Recipients set to production Support Team
- [ ] Flow enabled and active
- [ ] First scheduled execution monitored
- [ ] Success metrics baseline established

## Post-Deployment Monitoring (First Week)

### Daily Checks
- [ ] Day 1: Flow executed on schedule, message received
- [ ] Day 2: Correct enquiry count and details
- [ ] Day 3: Weekend skip worked correctly  
- [ ] Day 4: Staff feedback on message usefulness
- [ ] Day 5: Performance and timing validation

### Success Criteria
- [ ] 100% successful scheduled executions
- [ ] <5% false positive enquiries (actually resolved but flagged)
- [ ] <2 second execution time for normal data volumes
- [ ] Positive feedback from support team
- [ ] No error messages or system issues

## Issue Tracking

| Test Case | Status | Issues Found | Resolution | Retest Required |
|-----------|--------|--------------|------------|-----------------|
| TC1 | ⬜ | | | ⬜ |
| TC2 | ⬜ | | | ⬜ |
| TC3 | ⬜ | | | ⬜ |
| TC4 | ⬜ | | | ⬜ |
| TC5 | ⬜ | | | ⬜ |

---

**Tester**: ________________  
**Test Date**: _______________  
**Environment**: Dev/Staging/Prod  
**Version**: UC2 v1.0  
**Dependencies**: UC1 WhatsApp Enquiry Auto-Handler v1.0
