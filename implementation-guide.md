# SleekFlow WhatsApp Enquiry Workflow - Implementation Guide

## Prerequisites
- SleekFlow account with Flow Builder access
- WhatsApp Business API integrated with SleekFlow
- Admin permissions to create custom objects and workflows
- Support team/user groups configured

## Phase 1: Custom Object Setup

### Step 1: Create Enquiry Log Custom Object
1. Navigate to **Settings** → **Custom Objects**
2. Click **"Create New Object"**
3. Configure the object:
   - **Object Name**: "Enquiry Log"
   - **API Name**: "enquiry_log"
   - **Description**: "Logs customer enquiries for tracking and analysis"

### Step 2: Add Custom Object Properties
Add the following fields to the "Enquiry Log" object:

| Field Name | Field Type | API Name | Required | Default Value | Description |
|------------|------------|----------|----------|---------------|-------------|
| Enquiry ID | Text | enquiry_id | Yes | Auto-generated | Unique identifier |
| Customer Name | Text | customer_name | Yes | - | From contact.name |
| Phone Number | Text | phone_number | Yes | - | From contact.phone |
| Enquiry Details | Long Text | enquiry_details | Yes | - | Message content |
| Enquiry Type | Picklist | enquiry_type | No | - | Product Info, Shipping, Returns |
| Timestamp | DateTime | timestamp | Yes | Current Time | When enquiry was received |
| Status | Picklist | status | Yes | Open | Open, In Progress, Resolved |
| Assigned Staff | Text | assigned_staff | No | - | Staff member name |

### Step 3: Configure Picklist Values
**Enquiry Type Options:**
- Product Info
- Shipping
- Returns
- Technical Support
- General Enquiry

**Status Options:**
- Open
- In Progress
- Resolved
- Closed

## Phase 2: Workflow Configuration

### Step 1: Create New Flow
1. Navigate to **Flow Builder** → **Create New Flow**
2. Set flow name: "WhatsApp Enquiry Auto-Handler"
3. Set description: "Automatically handles new customer enquiries on WhatsApp"
4. Enable the flow

### Step 2: Configure Trigger
1. Add **"Message Received"** trigger
2. Configure trigger settings:
   - **Channel Filter**: WhatsApp only
   - **Direction**: Incoming messages
   - **Message Type**: Text messages

### Step 3: Add Condition Blocks

#### Condition 1: New Customer Check
1. Add **"If/Else"** condition block
2. Configure condition:
   - **Field**: `contact.conversation_count`
   - **Operator**: "equals"
   - **Value**: 0
   - **Label**: "Check if new customer"

#### Condition 2: Enquiry Keyword Check (nested under Condition 1 - Yes branch)
1. Add another **"If/Else"** condition block
2. Configure condition:
   - **Field**: `message.text`
   - **Operator**: "contains any of"
   - **Values**: "price,cost,how much,delivery,shipping,size,available,stock,order,buy,purchase,return,refund,help,support,info,information,details"
   - **Case Sensitive**: No
   - **Label**: "Check for enquiry keywords"

### Step 4: Add Action Blocks (in the "Yes" branch of both conditions)

#### Action 1: Send Auto-Reply
1. Add **"Send Message"** action
2. Configure message:
   - **Channel**: WhatsApp
   - **Message Type**: Text
   - **Template**: 
   ```
   Thanks for your enquiry! 🙏 We're reviewing your message and will respond soon. 
   
   For urgent matters, reply with 'URGENT' and we'll prioritize your request.
   
   - The [Company Name] Team
   ```

#### Action 2: Assign Conversation
1. Add **"Assign Conversation"** action
2. Configure assignment:
   - **Assignment Type**: Round Robin
   - **Team/User Group**: "Support Team"
   - **Fallback**: "Support Manager" (specific user)

#### Action 3: Create Enquiry Log
1. Add **"Create Custom Object"** action
2. Select **"Enquiry Log"** object
3. Map fields:
   - `enquiry_id`: Use formula `CONCAT("ENQ-", TEXT(NOW(), "YYYYMMDD-HHMMSS"))`
   - `customer_name`: `{{contact.name}}`
   - `phone_number`: `{{contact.phone}}`
   - `enquiry_details`: `{{message.text}}`
   - `timestamp`: `{{current_datetime}}`
   - `status`: "Open"
   - `assigned_staff`: `{{assigned_user.name}}`
   - Leave `enquiry_type` blank for manual classification

### Step 5: Add Error Handling

#### Error Handler for Assignment Failure
1. Add **"Error Handler"** block after the assignment action
2. Configure error condition: "Assignment Failed"
3. Add **"Send Internal Message"** action:
   - **Recipient**: Support Team channel or specific admin users
   - **Message**: 
   ```
   ⚠️ Assignment Failed
   
   Enquiry from: {{contact.name}} ({{contact.phone}})
   Message: {{message.text}}
   Time: {{current_datetime}}
   
   Please assign manually in SleekFlow.
   ```

### Step 6: Configure Flow Settings
1. Set **Flow Priority**: High
2. Enable **Flow Logging** for debugging
3. Set **Execution Timeout**: 30 seconds
4. Save and activate the flow

## Phase 3: Testing Setup

### Step 1: Create Test Contact
1. Add a test contact with `conversation_count = 0`
2. Use a real phone number you can test with

### Step 2: Prepare Test Messages
Create test messages containing keywords:
- "What's the price of your product?"
- "Do you have delivery to my area?"
- "I need help with an order"

### Step 3: Verify Flow Logic
1. Send test message
2. Check auto-reply is sent
3. Verify conversation assignment
4. Confirm custom object creation
5. Test error scenarios

## Phase 4: Deployment

### Step 1: Production Validation
1. Test with real team members
2. Verify all integrations work
3. Check custom object data accuracy

### Step 2: Go Live
1. Activate the flow
2. Monitor initial performance
3. Set up alerts for failures

### Step 3: Ongoing Maintenance
1. Review enquiry logs weekly
2. Update keywords based on common queries
3. Refine assignment rules as needed
4. Monitor response times and customer satisfaction

## Common Configuration Variations

### Alternative Keyword Matching
Instead of simple text matching, you can use regex patterns:
- Pattern: `\b(price|cost|how\s+much)\b`
- This matches whole words only

### Advanced Assignment Rules
- Time-based assignment (business hours vs after hours)
- Skill-based routing based on enquiry type
- Language-based assignment

### Enhanced Auto-Reply Templates
- Personalized with customer name
- Include estimated response time
- Provide FAQ links
- Offer callback option

## Next Steps
1. Complete the implementation following this guide
2. Run through the testing checklist
3. Deploy to production
4. Monitor and optimize based on performance metrics
