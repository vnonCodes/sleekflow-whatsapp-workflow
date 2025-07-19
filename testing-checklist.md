# Testing and Validation Guide for WhatsApp Enquiry Workflow

## Objective
Ensure that the WhatsApp Enquiry Workflow functions as expected and handles all scenarios gracefully.

## Test Environment Setup
- **Test Account**: Configure a test WhatsApp Business account connected to SleekFlow.
- **Test Contacts**: Create test contacts with different `conversation_count` values.
- **Test Message Templates**: Prepare messages that match expected keywords and edge cases.

## Test Cases

### Test Case 1: New Customer with Enquiry Keywords
- **Steps**:
  1. Send a message from a contact with `conversation_count = 0`.
  2. Ensure the message contains one of the pre-defined keywords.
- **Expected Results**:
  - Auto-reply is sent.
  - Conversation is assigned to support.
  - Custom object "Enquiry Log" is created with mapped fields.

### Test Case 2: Existing Customer Enquiry
- **Steps**:
  1. Send a message from a contact with `conversation_count > 0`.
  2. Include a keyword in the message.
- **Expected Results**:
  - Workflow ends silently.

### Test Case 3: New Customer Without Keywords
- **Steps**:
  1. Send a message from a new contact without keywords.
- **Expected Results**:
  - Workflow ends silently.

### Test Case 4: Assignment Failure
- **Steps**:
  1. Simulate scenario where no staff are available for assignment.
  2. Send a valid enquiry message.
- **Expected Results**:
  - Internal message is sent to admins notifying of assignment failure.

### Test Case 5: Log Creation Verification
- **Steps**:
  1. Trigger workflow with valid conditions.
  2. Verify log details in SleekFlow against test message content.
- **Expected Results**:
  - Fields are accurately logged as per the mapping.

## Post-Test Validation
- Review logs and verify all actions executed correctly.
- Check if any errors were reported during testing.
- Ensure all test cases pass successfully.

## Deployment Validation
- Conduct a full simulation with the live environment but within a limited scope.
- Monitor feedback and performance; adjust configurations if necessary.

## Performance Testing
- Measure the flow execution time under normal and stress conditions.
- Verify response times and system behavior under load.

## Final Steps
- Update documentation with test results and observations.
- Communicate findings and prepare for full deployment.

---

Document created on 2025-07-19.
Prepared by: SleekFlow Automation Author
Version 1.0
