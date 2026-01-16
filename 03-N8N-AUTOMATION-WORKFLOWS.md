# N8N Automation Workflows for ComCreate

**Document Version:** 1.0
**Created:** January 16, 2026

---

## Overview

This document provides technical specifications for N8N automation workflows to power ComCreate's B2B outreach to local service businesses. Based on compliance research, the strategy prioritizes email as the primary automated channel, with LinkedIn handled manually (supported by N8N task management).

**Important Compliance Notes:**
- LinkedIn direct automation = HIGH RISK (account bans likely)
- Instagram cold DM automation = CRITICAL RISK (avoid)
- Cold SMS = ILLEGAL without consent (TCPA violations)
- Email cold outreach = COMPLIANT (follow CAN-SPAM)

---

## Tools Stack

| Tool | Purpose | N8N Integration | Monthly Cost |
|------|---------|-----------------|--------------|
| N8N | Workflow orchestration | Self-hosted or Cloud | $0-50 |
| Apollo.io | Lead data sourcing | Native node | $99 |
| Hunter.io | Email verification | HTTP Request | $49 |
| Lemlist | Email sequences | Webhook | $59 |
| HubSpot | CRM | Native node | Free-$45 |
| Calendly | Booking | Webhook | $10 |
| Slack | Notifications | Native node | Free |
| Google Sheets | Data tracking | Native node | Free |

**Total:** ~$270-315/month

---

## Workflow 1: Lead Acquisition Pipeline

### Purpose
Automatically pull new local service business leads from Apollo.io, verify emails, and add qualified leads to the outreach sequence.

### Trigger
Schedule: Daily at 8:00 AM

### Flow Diagram

```
[Schedule Trigger: Daily 8 AM]
            │
            ▼
[Apollo.io: Search for new leads]
    - Industry: Home Services, Construction, Dental, Legal
    - Employee count: 1-50
    - Location: Target cities
    - Title: Owner, President, CEO
    - Limit: 50 leads/day
            │
            ▼
[Loop: For each lead]
            │
            ▼
[HTTP Request: Hunter.io verify email]
            │
            ▼
[IF: Email valid?]
    │
    ├─→ YES: [Continue]
    │
    └─→ NO: [Skip lead]
            │
            ▼
[HubSpot: Create/Update Contact]
    - First name
    - Last name
    - Email
    - Company
    - Phone
    - Industry (custom property)
    - Lead source: "Apollo Outreach"
    - Lead status: "New"
            │
            ▼
[Google Sheets: Log lead]
    - Track all attempted leads
    - Track verification results
            │
            ▼
[HTTP Request: Add to Lemlist campaign]
    - Campaign: "Cold Outreach - [Industry]"
            │
            ▼
[Slack: Daily summary]
    - "Added X leads to outreach today"
    - "Y emails failed verification"
```

### Node Configurations

**Node 1: Schedule Trigger**
```json
{
  "rule": {
    "interval": [
      {
        "field": "hours",
        "triggerAtHour": 8
      }
    ]
  }
}
```

**Node 2: Apollo.io Search**
```json
{
  "operation": "search",
  "filters": {
    "person_titles": ["Owner", "President", "CEO", "Founder"],
    "organization_num_employees_ranges": ["1,10", "11,50"],
    "person_locations": ["California, United States", "Arizona, United States"],
    "organization_industry_tag_ids": ["home services", "construction", "hvac"]
  },
  "limit": 50
}
```

**Node 3: Hunter.io Verification**
```json
{
  "method": "GET",
  "url": "https://api.hunter.io/v2/email-verifier",
  "qs": {
    "email": "={{ $json.email }}",
    "api_key": "{{ $credentials.hunterApiKey }}"
  }
}
```

**Node 4: HubSpot Create Contact**
```json
{
  "operation": "create",
  "resource": "contact",
  "properties": {
    "firstname": "={{ $json.first_name }}",
    "lastname": "={{ $json.last_name }}",
    "email": "={{ $json.email }}",
    "company": "={{ $json.organization.name }}",
    "phone": "={{ $json.phone_numbers[0].number }}",
    "industry": "={{ $json.organization.industry }}",
    "lead_source": "Apollo Outreach",
    "hs_lead_status": "NEW"
  }
}
```

**Node 5: Add to Lemlist**
```json
{
  "method": "POST",
  "url": "https://api.lemlist.com/api/campaigns/{{ $env.LEMLIST_CAMPAIGN_ID }}/leads",
  "headers": {
    "Authorization": "Basic {{ $credentials.lemlistApiKey }}"
  },
  "body": {
    "email": "={{ $json.email }}",
    "firstName": "={{ $json.first_name }}",
    "lastName": "={{ $json.last_name }}",
    "companyName": "={{ $json.organization.name }}"
  }
}
```

---

## Workflow 2: Reply Handler

### Purpose
When someone replies to an outreach email, detect sentiment, update CRM, and notify the team for follow-up.

### Trigger
Webhook: Lemlist sends webhook on email reply

### Flow Diagram

```
[Webhook: Lemlist reply received]
            │
            ▼
[Set: Extract reply data]
    - Email
    - Reply text
    - Campaign name
    - Lead name
            │
            ▼
[IF: Contains negative keywords?]
    - "unsubscribe", "remove", "stop", "not interested"
    │
    ├─→ YES: [Mark as "Not Interested"]
    │         │
    │         ▼
    │   [HubSpot: Update contact]
    │   - Lead status: "Unqualified"
    │   - Reason: "Negative reply"
    │         │
    │         ▼
    │   [Lemlist: Remove from campaign]
    │
    └─→ NO: [Continue to positive flow]
            │
            ▼
[IF: Contains positive signals?]
    - "interested", "tell me more", "how much", "pricing", "call"
    │
    ├─→ YES: [Hot Lead!]
    │         │
    │         ▼
    │   [HubSpot: Update contact]
    │   - Lead status: "Engaged"
    │   - Deal: Create new deal
    │         │
    │         ▼
    │   [Slack: Urgent notification]
    │   - "@channel Hot lead! Reply from [Name]"
    │   - Include reply text
    │         │
    │         ▼
    │   [Lemlist: Pause sequence]
    │
    └─→ NEUTRAL: [Standard follow-up]
            │
            ▼
[HubSpot: Log activity]
    - Note: "Email reply received"
    - Include reply content
            │
            ▼
[Slack: Team notification]
    - "Reply from [Name] at [Company]"
    - Include reply preview
```

### Webhook Configuration

**Lemlist Webhook URL:**
`https://your-n8n-instance.com/webhook/lemlist-reply`

**Lemlist Settings:**
- Enable webhook for "Lead replied"
- Enable webhook for "Lead unsubscribed"

### Sentiment Detection Keywords

**Positive Indicators:**
```
interested, love to, sounds good, tell me more, how much,
pricing, cost, schedule, call me, set up a time,
available, let's talk, like to learn, perfect timing
```

**Negative Indicators:**
```
unsubscribe, remove, stop, not interested, no thanks,
take me off, don't contact, wrong person, not for us,
already have, not looking, delete
```

---

## Workflow 3: Meeting Booked Handler

### Purpose
When a prospect books a strategy call via Calendly, create deal in HubSpot, send confirmations, and prepare the sales team.

### Trigger
Webhook: Calendly sends booking notification

### Flow Diagram

```
[Webhook: Calendly booking received]
            │
            ▼
[Set: Parse booking data]
    - Name
    - Email
    - Company (from custom question)
    - Meeting time
    - Meeting type
            │
            ▼
[HubSpot: Search for existing contact]
            │
            ▼
[IF: Contact exists?]
    │
    ├─→ YES: [HubSpot: Update contact]
    │
    └─→ NO: [HubSpot: Create contact]
            │
            ▼
[HubSpot: Create Deal]
    - Deal name: "[Company] - Strategy Call"
    - Pipeline: Sales Pipeline
    - Stage: Appointment Scheduled
    - Amount: $6,594 (avg 6-month value)
    - Close date: Meeting date + 14 days
    - Associate with contact
            │
            ▼
[IF: Lead source = Apollo Outreach?]
    │
    ├─→ YES: [Lemlist: Mark as converted]
    │         │
    │         ▼
    │   [Lemlist: Remove from sequence]
    │
    └─→ NO: [Continue]
            │
            ▼
[Gmail: Send prep email to sales]
    - Meeting details
    - Link to HubSpot contact
    - Link to company website
    - Notes from previous interactions
            │
            ▼
[Google Calendar: Create prep task]
    - 1 hour before meeting
    - "Prep for call with [Company]"
            │
            ▼
[Slack: Notify team]
    - "New strategy call booked!"
    - Meeting time
    - Company name
    - Lead source
```

### Calendly Custom Questions

Add these to your Calendly booking form:
1. Company Name (required)
2. What services are you most interested in? (dropdown)
3. Current monthly marketing budget (dropdown: <$500, $500-1K, $1K-2K, $2K+)
4. How did you hear about us? (dropdown or text)

---

## Workflow 4: LinkedIn Task Generator

### Purpose
Generate daily manual LinkedIn outreach tasks based on leads in the pipeline.

### Trigger
Schedule: Daily at 9:00 AM

### Flow Diagram

```
[Schedule Trigger: Daily 9 AM]
            │
            ▼
[HubSpot: Get contacts]
    - Filter: Lead status = "New" OR "Contacted"
    - Filter: Has LinkedIn URL
    - Filter: Not yet connected on LinkedIn
    - Limit: 25
            │
            ▼
[Loop: For each contact]
            │
            ▼
[Set: Format task]
    - Name
    - Company
    - LinkedIn URL
    - Recommended message (personalized)
            │
            ▼
[Google Sheets: Add to daily task list]
    - Date
    - Contact name
    - LinkedIn URL
    - Suggested connection message
    - Status: Pending
            │
            ▼
[After loop completes]
            │
            ▼
[Slack: Send daily task list]
    - "Today's LinkedIn outreach (X contacts):"
    - List with links
    - Reminder of daily limits (25 connections)
```

### LinkedIn Message Templates (for manual sending)

**Connection Request (300 char limit):**

Template 1 - Neighborhood Focus:
```
Hi {{firstName}}, noticed you run {{companyName}} in {{city}}.
I help local service businesses grow their online presence.
Would love to connect!
```

Template 2 - Industry Focus:
```
Hi {{firstName}}, always looking to connect with successful
{{industry}} business owners. Saw {{companyName}} has great reviews.
Let's connect! -Josh
```

**First Message After Connection:**
```
Thanks for connecting, {{firstName}}!

Quick question - what's your biggest challenge right now
when it comes to getting new customers online?

I ask because we've been working with a lot of {{industry}}
businesses and keep hearing the same things.
```

---

## Workflow 5: Lead Scoring

### Purpose
Automatically score leads based on engagement and attributes to prioritize outreach.

### Trigger
On HubSpot contact update (webhook)

### Scoring Criteria

| Criteria | Points |
|----------|--------|
| Email opened (any) | +5 |
| Email clicked | +10 |
| Email replied (positive) | +25 |
| Website visit | +15 |
| Pricing page visit | +20 |
| Meeting booked | +50 |
| Company size 10-50 | +10 |
| Company size 1-10 | +5 |
| High-value industry (HVAC, dental) | +10 |
| Has phone number | +5 |
| Has LinkedIn | +5 |
| West Coast location | +5 |

### Score Thresholds

| Score | Classification | Action |
|-------|---------------|--------|
| 0-25 | Cold | Continue sequence |
| 26-50 | Warm | Add to LinkedIn task list |
| 51-75 | Hot | Priority follow-up, phone call |
| 76+ | Very Hot | Immediate sales outreach |

### Flow Diagram

```
[Webhook: HubSpot contact updated]
            │
            ▼
[HubSpot: Get full contact record]
            │
            ▼
[Function: Calculate score]
    - Sum all applicable criteria
            │
            ▼
[HubSpot: Update lead score property]
            │
            ▼
[IF: Score >= 51?]
    │
    ├─→ YES: [Slack: Hot lead alert]
    │
    └─→ NO: [End]
```

---

## Workflow 6: Weekly Reporting

### Purpose
Generate weekly outreach performance report for the team.

### Trigger
Schedule: Every Monday at 8:00 AM

### Flow Diagram

```
[Schedule Trigger: Monday 8 AM]
            │
            ▼
[Lemlist API: Get campaign stats]
    - Emails sent
    - Open rate
    - Reply rate
    - Bounces
            │
            ▼
[HubSpot: Get pipeline stats]
    - New leads this week
    - Meetings booked
    - Deals created
    - Deals closed
            │
            ▼
[Google Sheets: Get LinkedIn task completion]
    - Tasks assigned
    - Tasks completed
    - Connection acceptance rate
            │
            ▼
[Function: Compile report]
            │
            ▼
[Slack: Post weekly report]
            │
            ▼
[Gmail: Send to leadership]
```

### Report Template

```
## Weekly Outreach Report - Week of {{date}}

### Email Outreach (Lemlist)
- Emails Sent: {{emailsSent}}
- Open Rate: {{openRate}}%
- Reply Rate: {{replyRate}}%
- Positive Replies: {{positiveReplies}}
- Bounces: {{bounces}}

### Pipeline (HubSpot)
- New Leads: {{newLeads}}
- Meetings Booked: {{meetingsBooked}}
- Deals Created: {{dealsCreated}}
- Deals Closed: {{dealsClosed}}
- Pipeline Value: ${{pipelineValue}}

### LinkedIn (Manual)
- Connection Requests Sent: {{connectionsSent}}
- Connections Accepted: {{connectionsAccepted}}
- Messages Sent: {{messagesSent}}
- Responses: {{responses}}

### Key Metrics
- Cost Per Lead: ${{costPerLead}}
- Cost Per Meeting: ${{costPerMeeting}}
- Lead to Meeting Rate: {{leadToMeetingRate}}%
```

---

## Workflow 7: ManyChat Instagram Handler

### Purpose
Handle Instagram comment-to-DM automation for compliant lead capture.

### Trigger
Webhook: ManyChat sends trigger when user comments keyword

### Flow Diagram

```
[Webhook: ManyChat comment trigger]
    - User commented "GUIDE" on post
            │
            ▼
[ManyChat: Send automated DM sequence]
    - "Hey! Thanks for your interest in our marketing guide."
    - "Quick question - what type of service business do you run?"
    - [Button options: HVAC, Plumbing, Electrical, Other]
            │
            ▼
[ManyChat: Collect email]
    - "What's the best email to send the guide to?"
            │
            ▼
[Webhook: ManyChat sends data to N8N]
            │
            ▼
[HubSpot: Create contact]
    - Lead source: "Instagram ManyChat"
    - Include Instagram username
            │
            ▼
[Gmail: Send lead magnet]
            │
            ▼
[HubSpot: Enroll in email nurture sequence]
```

### ManyChat Flow Structure

1. **Trigger:** User comments keyword (e.g., "GUIDE", "FREE")
2. **Message 1:** Welcome + ask business type
3. **Message 2:** Based on selection, personalized response
4. **Message 3:** Ask for email
5. **Message 4:** Confirm + set expectation
6. **Webhook:** Send to N8N for CRM integration

---

## Environment Variables

Set these in N8N:

```
# Apollo.io
APOLLO_API_KEY=your_api_key

# Hunter.io
HUNTER_API_KEY=your_api_key

# Lemlist
LEMLIST_API_KEY=your_api_key
LEMLIST_CAMPAIGN_ID=your_campaign_id

# HubSpot
HUBSPOT_API_KEY=your_api_key

# Calendly
CALENDLY_WEBHOOK_SECRET=your_secret

# Slack
SLACK_WEBHOOK_URL=your_webhook_url

# ManyChat
MANYCHAT_API_KEY=your_api_key
```

---

## Error Handling

### All Workflows Should Include:

1. **Error Trigger Node**
   - Catches any workflow errors
   - Sends Slack notification with error details
   - Logs to Google Sheet for tracking

2. **Rate Limit Handling**
   - Add Wait nodes between API calls
   - Respect rate limits:
     - Apollo: 100 requests/min
     - Hunter: 100 requests/min
     - Lemlist: 50 requests/min
     - HubSpot: 100 requests/10 sec

3. **Retry Logic**
   - HTTP Request nodes: Enable retry on failure
   - Max retries: 3
   - Wait between retries: 60 seconds

### Error Notification Template

```
Workflow Error Alert

Workflow: {{workflowName}}
Node: {{nodeName}}
Error: {{errorMessage}}
Time: {{timestamp}}

Data:
{{JSON.stringify($json)}}
```

---

## Implementation Checklist

### Week 1: Foundation
- [ ] Set up N8N instance (cloud or self-hosted)
- [ ] Create Apollo.io account and generate API key
- [ ] Create Hunter.io account and generate API key
- [ ] Set up Lemlist with first campaign
- [ ] Configure HubSpot custom properties
- [ ] Create Slack workspace/channel for notifications
- [ ] Set up Google Sheet for tracking

### Week 2: Build Workflows
- [ ] Build Workflow 1: Lead Acquisition Pipeline
- [ ] Build Workflow 2: Reply Handler
- [ ] Build Workflow 3: Meeting Booked Handler
- [ ] Test each workflow with sample data

### Week 3: Enhance & Launch
- [ ] Build Workflow 4: LinkedIn Task Generator
- [ ] Build Workflow 5: Lead Scoring
- [ ] Build Workflow 6: Weekly Reporting
- [ ] Full system test with live data
- [ ] Launch with 50 leads/day

### Week 4: Optimize
- [ ] Monitor delivery rates and adjust
- [ ] Review reply sentiment accuracy
- [ ] Adjust scoring based on conversion data
- [ ] Scale to full volume (100+ leads/day)

---

## Compliance Checklist

### CAN-SPAM Requirements (Email)
- [ ] Physical address in every email
- [ ] Clear unsubscribe link
- [ ] Honor opt-outs within 10 days
- [ ] No deceptive subject lines
- [ ] Identify as advertisement where required

### GDPR Considerations (if targeting EU)
- [ ] Legitimate interest documented
- [ ] Right to erasure process
- [ ] Data processing records

### Platform Terms of Service
- [x] NO automated LinkedIn connections/messages
- [x] NO automated Instagram cold DMs
- [x] NO cold SMS outreach
- [x] ManyChat used only for responding to user-initiated contact

---

*Workflow specifications prepared January 2026. Test thoroughly before scaling.*
