# SleekFlow WhatsApp Enquiry Workflow Implementation Guide

## Overview
This package contains all necessary documentation and configuration files to implement automated WhatsApp enquiry workflows in SleekFlow Flow Builder, including both initial enquiry handling (UC1) and daily internal reminder system (UC2).

## Package Contents

### UC1: WhatsApp Enquiry Auto-Handler
- `implementation-guide.md` - Step-by-step implementation instructions for UC1
- `workflow-config.json` - Sample workflow configuration for UC1
- `custom-object-schema.json` - Custom object definition (shared by UC1 & UC2)
- `testing-checklist.md` - Testing and validation guide for UC1

### UC2: Daily Internal Reminder Workflow
- `uc2-implementation-guide.md` - Step-by-step implementation for daily reminders
- `uc2-daily-reminder-workflow.json` - Complete workflow configuration for UC2
- `uc2-testing-checklist.md` - Comprehensive testing guide for UC2
- `uc2-test-report.md` - Test report template for UC2 validation

## Quick Start

### UC1 Implementation (Prerequisite for UC2)
1. Follow `implementation-guide.md` to set up the base workflow
2. Create the custom object using `custom-object-schema.json`
3. Import or manually configure the workflow using `workflow-config.json`
4. Run through `testing-checklist.md`
5. Deploy and monitor

### UC2 Implementation (Daily Reminders)
1. Ensure UC1 is working and populating enquiry logs
2. Follow `uc2-implementation-guide.md` for scheduled reminder setup
3. Configure the workflow using `uc2-daily-reminder-workflow.json`
4. Run through `uc2-testing-checklist.md`
5. Use `uc2-test-report.md` to document test results
6. Deploy with 6 PM weekday schedule

## Workflow Summaries

### UC1: WhatsApp Enquiry Auto-Handler
- **Trigger**: New WhatsApp message received
- **Conditions**: New customer (conversation_count = 0) + enquiry keywords
- **Actions**: Auto-reply → Assign to support → Create enquiry log
- **Error Handling**: Admin notifications for failed assignments

### UC2: Daily Internal Reminder Workflow  
- **Trigger**: Scheduled daily at 6:00 PM (weekdays only)
- **Conditions**: Weekday check → Search for unresolved enquiries (>24h old)
- **Actions**: Build summary → Send internal reminder to support team
- **Error Handling**: Fallback notifications for search failures

## Support
Refer to troubleshooting.md for common issues or contact your SleekFlow administrator.