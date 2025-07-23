# UC4: Zoho CRM Lead Logging and Monthly Monitoring Workflow - Implementation Guide

## Overview
This guide implements UC4, building on the existing UC1 (WhatsApp Enquiry Auto-Handler), UC2 (Daily Reminder), and UC3 (Shopify Order Confirmation) workflows. The Zoho CRM Lead Logging Workflow automatically logs new SleekFlow contacts (excluding Shopify customers) as leads in Zoho CRM and monitors monthly lead counts for business analytics.

## Prerequisites
- UC1, UC2, UC3 workflows implemented and operational
- SleekFlow account with Flow Builder access and custom object capabilities
- Zoho CRM account with API access
- Zoho OAuth setup or API token configured
- Make.com account (leveraging existing UC3 setup for advanced monitoring)
- Admin permissions to create custom objects and workflows

## Architecture Overview

### Primary Workflow Path: SleekFlow → Zoho CRM
```
Contact Created → Lead Source Check → Zoho CRM API → Lead Log Update → Monthly Monitoring
```

### Enhanced Monitoring Path (using existing UC3 Make.com setup):
```
SleekFlow → Make.com → Google Sheets → Monthly Report Generation → Email Alerts
```

## Phase 1: Custom Object Extension

### Step 1: Create Lead Log Custom Object
1. Navigate to **Settings** → **Custom Objects**
2. Click **"Create New Object"**
3. Configure the object:
   - **Object Name**: "Lead Log"
   - **API Name**: "lead_log"
   - **Description**: "Monthly tracking of lead generation and CRM synchronization"

### Step 2: Add Lead Log Properties
Add the following fields to the "Lead Log" object:

| Field Name | Field Type | API Name | Required | Default Value | Description |
|------------|------------|----------|----------|---------------|-------------|
| Month Year | Text | month_year | Yes | Current Month | Format: "2025-01" |
| Lead Count | Number | lead_count | Yes | 1 | Monthly lead counter |
| Lead Source | Picklist | lead_source | Yes | - | WhatsApp, Website, Social |
| Last Updated | DateTime | last_updated | Yes | Current Time | Timestamp of last update |
| Sync Status | Picklist | sync_status | Yes | Pending | Pending, Synced, Failed |
| Failed Contacts | Long Text | failed_contacts | No | - | JSON array of failed sync attempts |

### Step 3: Configure Lead Source Picklist Values
**Lead Source Options:**
- WhatsApp
- Website Form
- Social Media
- Direct Contact
- Referral
- Other

**Sync Status Options:**
- Pending
- Synced
- Failed
- Retry Required

### Step 4: Extend Contact Custom Properties
Add lead source tracking to contacts:
1. Navigate to **Settings** → **Contact Fields**
2. Add custom field:
   - **Field Name**: "Lead Source"
   - **API Name**: "lead_source"
   - **Field Type**: Picklist (same options as above)
   - **Default**: "WhatsApp" (for contacts from UC1)

## Phase 2: Zoho CRM API Setup

### Step 1: Zoho OAuth Configuration
1. **Register Application in Zoho Developer Console**:
   - Go to https://api-console.zoho.com/
   - Create new application: "SleekFlow Lead Integration"
   - Add scopes: `ZohoCRM.modules.ALL`, `ZohoCRM.users.READ`
   - Note Client ID and Client Secret

2. **Generate Access Token**:
   ```bash
   # Authorization URL (replace CLIENT_ID)
   https://accounts.zoho.com/oauth/v2/auth?scope=ZohoCRM.modules.ALL&client_id=CLIENT_ID&response_type=code&redirect_uri=https://httpbin.org/get
   
   # Exchange code for tokens (use Postman or curl)
   POST https://accounts.zoho.com/oauth/v2/token
   Content-Type: application/x-www-form-urlencoded
   
   grant_type=authorization_code&
   client_id=YOUR_CLIENT_ID&
   client_secret=YOUR_CLIENT_SECRET&
   redirect_uri=https://httpbin.org/get&
   code=AUTHORIZATION_CODE
   ```

3. **Set Up Token Refresh**:
   - Store refresh_token securely
   - Implement token refresh logic in Make.com or SleekFlow

### Step 2: Test Zoho CRM API Connection
Test payload for contact creation:
```json
{
  "data": [
    {
      "First_Name": "John",
      "Last_Name": "Doe",
      "Phone": "+1234567890",
      "Email": "john.doe@example.com",
      "Lead_Source": "WhatsApp",
      "Lead_Status": "Not Contacted",
      "Description": "Lead generated from SleekFlow contact creation",
      "Custom_Fields": {
        "SleekFlow_Contact_ID": "contact_123",
        "Created_Date": "2025-01-21T10:30:00Z"
      }
    }
  ]
}
```

## Phase 3: SleekFlow Workflow Configuration

### Step 1: Create Primary Lead Logging Flow
1. Navigate to **Flow Builder** → **Create New Flow**
2. Set flow name: "Zoho CRM Lead Logger"
3. Set description: "Automatically logs non-Shopify contacts to Zoho CRM"
4. Enable the flow

### Step 2: Configure Trigger
1. Add **"Contact Created"** trigger
2. Configure trigger settings:
   - **Contact Source**: All channels
   - **Contact Type**: All contact types
   - **Execution**: Real-time

### Step 3: Add Condition Blocks

#### Condition 1: Lead Source Filter
1. Add **"If/Else"** condition block
2. Configure condition:
   - **Field**: `contact.lead_source` OR `contact.custom_fields.lead_source`
   - **Operator**: "not equals"
   - **Value**: "Shopify"
   - **Label**: "Exclude Shopify Customers"

#### Condition 2: Required Fields Check (nested under Condition 1 - Yes branch)
1. Add another **"If/Else"** condition block
2. Configure condition:
   - **Field**: `contact.name`
   - **Operator**: "is not empty"
   - **Additional conditions**: `contact.phone` is not empty
   - **Label**: "Validate Required Contact Data"

### Step 4: Add Action Blocks

#### Action 1: Create Zoho CRM Lead
1. Add **"HTTP Request"** action
2. Configure API call:
   - **URL**: `https://www.zohoapis.com/crm/v2/Leads`
   - **Method**: POST
   - **Headers**:
     ```json
     {
       "Authorization": "Zoho-oauthtoken {{ZOHO_ACCESS_TOKEN}}",
       "Content-Type": "application/json"
     }
     ```
   - **Request Body**:
     ```json
     {
       "data": [
         {
           "First_Name": "{{split(contact.name, ' ')[0]}}",
           "Last_Name": "{{split(contact.name, ' ')[1] || split(contact.name, ' ')[0]}}",
           "Phone": "{{contact.phone}}",
           "Email": "{{contact.email}}",
           "Lead_Source": "{{contact.lead_source || 'WhatsApp'}}",
           "Lead_Status": "Not Contacted",
           "Description": "Lead generated from SleekFlow - {{contact.lead_source || 'WhatsApp'}} channel",
           "SleekFlow_Contact_ID": "{{contact.id}}",
           "Created_Date": "{{current_datetime}}"
         }
       ]
     }
     ```

#### Action 2: Update/Create Lead Log Entry
1. Add **"Search Custom Object"** action
2. Search for existing Lead Log:
   - **Object**: "Lead Log"
   - **Criteria**: `month_year` equals `{{format(current_datetime, 'YYYY-MM')}}`
   - **Lead Source**: `{{contact.lead_source || 'WhatsApp'}}`

3. Add **"If/Else"** condition for found/not found
4. **If Found** - Add **"Update Custom Object"**:
   ```json
   {
     "lead_count": "{{add(found_record.lead_count, 1)}}",
     "last_updated": "{{current_datetime}}",
     "sync_status": "{{if(previous_action.success, 'Synced', 'Failed')}}"
   }
   ```

5. **If Not Found** - Add **"Create Custom Object"**:
   ```json
   {
     "month_year": "{{format(current_datetime, 'YYYY-MM')}}",
     "lead_count": 1,
     "lead_source": "{{contact.lead_source || 'WhatsApp'}}",
     "last_updated": "{{current_datetime}}",
     "sync_status": "{{if(previous_action.success, 'Synced', 'Failed')}}"
   }
   ```

#### Action 3: Tag Contact with Sync Status
1. Add **"Add Tag to Contact"** action
2. Configure tagging:
   - **Tag**: "CRM_Synced" (if successful) or "Sync_Pending" (if failed)
   - **Contact**: `{{contact.id}}`

### Step 5: Error Handling Configuration

#### Error Handler for Zoho API Failure
1. Add **"Error Handler"** block after Zoho API action
2. Configure error condition: "HTTP Request Failed"
3. Add **"Update Lead Log"** action:
   ```json
   {
     "sync_status": "Failed",
     "failed_contacts": "{{append(existing_failed_contacts, contact.id)}}"
   }
   ```

4. Add **"Send Internal Notification"** action:
   ```
   🚨 Zoho CRM Sync Failed
   
   Contact Details:
   - Name: {{contact.name}}
   - Phone: {{contact.phone}}
   - Email: {{contact.email}}
   - Lead Source: {{contact.lead_source}}
   - Error: {{error.message}}
   
   Please check Zoho API status and retry manually.
   ```

## Phase 4: Monthly Monitoring Enhancement

### Step 1: Create Monthly Report Flow (Optional - Advanced)
1. Create new flow: "Monthly Lead Report Generator"
2. **Trigger**: Scheduled (Last day of month, 11:59 PM)
3. **Actions**:
   - Search all Lead Log entries for current month
   - Aggregate counts by lead source
   - Generate summary report
   - Send to stakeholders

### Step 2: Integrate with Existing UC3 Make.com Setup
Extend the existing UC3 Make.com scenario for enhanced monitoring:

1. **Add New Webhook Module** in existing Make.com scenario
2. **Configure SleekFlow Integration**:
   - Webhook URL: Point to Make.com from SleekFlow lead logging flow
   - Payload: Include lead data and monthly aggregation

3. **Google Sheets Integration** (leveraging UC3 setup):
   - Create new sheet: "Lead Tracking"
   - Columns: Date, Lead Source, Contact Name, Phone, Email, Zoho Status
   - Monthly summary calculations

4. **Monthly Report Automation**:
   - Scheduled scenario: First day of month
   - Generate summary report from Google Sheets
   - Email monthly metrics to stakeholders

## Phase 5: Advanced Monitoring Dashboard

### Step 1: Lead Analytics Custom Object
Create additional custom object for detailed analytics:
```json
{
  "name": "Lead Analytics",
  "fields": [
    {
      "name": "Period",
      "api_name": "period",
      "type": "text",
      "description": "YYYY-MM format"
    },
    {
      "name": "Total Leads",
      "api_name": "total_leads",
      "type": "number"
    },
    {
      "name": "WhatsApp Leads",
      "api_name": "whatsapp_leads",
      "type": "number"
    },
    {
      "name": "Website Leads",
      "api_name": "website_leads",
      "type": "number"
    },
    {
      "name": "Sync Success Rate",
      "api_name": "sync_success_rate",
      "type": "number",
      "format": "percentage"
    },
    {
      "name": "Failed Sync Count",
      "api_name": "failed_sync_count",
      "type": "number"
    }
  ]
}
```

### Step 2: Reporting and Dashboards
1. **SleekFlow Built-in Reports**:
   - Lead Log summary views
   - Monthly trend analysis
   - Source performance comparison

2. **External Dashboard** (using existing UC3 Google Sheets):
   - Real-time lead tracking
   - Monthly goal vs. actual charts
   - Conversion funnel analysis

## Phase 6: Integration with Existing UC1-UC3 Workflows

### Enhanced Contact Flow (UC1 Integration)
1. **Modify UC1 Enquiry Flow**:
   - Set `lead_source = "WhatsApp"` for new contacts
   - Trigger UC4 flow after enquiry log creation

2. **Cross-Reference Data**:
   - Link enquiry logs to lead logs
   - Track conversion from lead to customer

### Shopify Customer Exclusion (UC3 Integration)
1. **Contact Tagging in UC3**:
   - Tag Shopify contacts with `lead_source = "Shopify"`
   - Ensure UC4 exclusion logic works correctly

2. **Customer Journey Tracking**:
   - Monitor lead → customer progression
   - Identify high-value lead sources

### Enhanced Daily Reminders (UC2 Integration)
1. **Include Lead Metrics** in daily reports:
   ```
   📊 Daily Summary Report
   
   💬 Enquiries: {{enquiry_count}}
   🎯 New Leads: {{lead_count}}
   🛒 Orders: {{order_count}}
   ⚠️  Sync Failures: {{failed_sync_count}}
   ```

2. **Action Items**:
   - Alert for unusual lead volume changes
   - Notify about sync failures requiring attention

## Phase 7: Testing and Validation

### Step 1: Unit Testing
1. **Create Test Contacts**:
   - Various lead sources (WhatsApp, Website, Social)
   - Include edge cases (missing fields, special characters)

2. **Zoho CRM Verification**:
   - Confirm leads appear in Zoho CRM
   - Verify field mapping accuracy
   - Check custom field population

3. **Lead Log Testing**:
   - Verify monthly counters increment correctly
   - Test rollover to new month
   - Validate source-specific tracking

### Step 2: Integration Testing
1. **End-to-End Flow**:
   - Contact created → Zoho lead → Lead log updated → Tag applied
   - Test error scenarios and recovery

2. **Cross-Workflow Integration**:
   - UC1 enquiry → UC4 lead logging
   - UC3 Shopify customer exclusion
   - UC2 enhanced reporting

### Step 3: Volume Testing
1. **Bulk Contact Creation**:
   - Test with multiple simultaneous contacts
   - Verify API rate limiting handling
   - Monitor performance impact

2. **Monthly Rollover**:
   - Test month-end boundary conditions
   - Verify new month initialization
   - Check historical data preservation

## Phase 8: Production Deployment

### Go-Live Checklist
- [ ] Zoho CRM API credentials configured and tested
- [ ] SleekFlow workflows activated
- [ ] Lead Log custom object deployed
- [ ] Contact lead_source field populated
- [ ] Error handling and notifications set up
- [ ] Monthly monitoring configured
- [ ] Integration with UC1-UC3 validated
- [ ] Team trained on new lead tracking

### Monitoring Setup
1. **SleekFlow Monitoring**:
   - Flow execution success rates
   - Lead log creation accuracy
   - Error notification responsiveness

2. **Zoho CRM Monitoring**:
   - Lead creation success rates
   - Data quality validation
   - API usage tracking

3. **Business Metrics**:
   - Monthly lead volume trends
   - Source performance analysis
   - Conversion rate tracking

## Phase 9: Success Metrics and KPIs

### Technical Metrics
- **Sync Success Rate**: > 98%
- **API Response Time**: < 5 seconds
- **Monthly Counter Accuracy**: 100%
- **Error Recovery Rate**: > 95%

### Business Metrics
- **Lead Tracking Coverage**: 100% of non-Shopify contacts
- **Monthly Reporting Accuracy**: < 2% variance
- **Data Quality Score**: > 95%
- **Cross-Workflow Integration**: 100% compatibility

### Integration Health
- **UC1 + UC4**: Lead generation from enquiries
- **UC3 + UC4**: Customer exclusion accuracy
- **UC2 + UC4**: Enhanced reporting completeness
- **System Performance**: < 10% impact on existing workflows

## Troubleshooting Guide

### Common Issues
1. **Zoho API Authentication Failures**:
   - Check access token expiration
   - Verify OAuth refresh token validity
   - Confirm API scope permissions

2. **Lead Source Assignment**:
   - Validate contact field population
   - Check UC1/UC3 integration points
   - Verify default value logic

3. **Monthly Counter Discrepancies**:
   - Audit Lead Log creation/updates
   - Check timezone handling
   - Verify search criteria accuracy

4. **Performance Impact**:
   - Monitor API rate limits
   - Optimize batch processing
   - Implement queueing if needed

### Error Recovery Procedures
1. **Failed Sync Recovery**:
   - Identify contacts with "Sync_Pending" tags
   - Run manual sync process
   - Update Lead Log counters

2. **Data Consistency Checks**:
   - Monthly audit of Lead Log vs. actual contacts
   - Zoho CRM data validation
   - Cross-reference with UC1/UC3 data

---

**Document Created**: 2025-01-21  
**Version**: 1.0  
**Dependencies**: UC1, UC2, UC3, Zoho CRM API, Make.com (optional)  
**Integration Platforms**: SleekFlow Flow Builder, Zoho CRM, Make.com
