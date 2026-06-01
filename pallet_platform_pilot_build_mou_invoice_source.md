# Pallet Lead Generation Platform — Pilot Build Scope, MOU Source, and Invoice Source

## 1. Project Overview

This document defines the pilot build for a lead generation and lightweight CRM platform for a pallet business serving Atlanta, Georgia and surrounding areas within approximately a three-hour drive.

The business buys, supplies, collects, refurbishes, and sells pallets. The platform will help the owners discover potential business leads, identify pallet pickup opportunities, organize contact information, route opportunities geographically, approve outreach, and manage follow-up activity through a simple CRM-style dashboard.

This document is intended to support:

- Build planning
- Developer handoff
- MOU preparation
- Invoice preparation
- Scope control
- Pilot acceptance testing

---

## 2. Pilot Objective

The objective of the pilot is to produce a working lead generation platform that helps the pallet company:

1. Discover local and regional businesses that may use, discard, buy, or supply pallets.
2. Classify those businesses as possible pallet buyers, pallet sources, pickup opportunities, or low-priority leads.
3. Store leads, contacts, notes, routing information, and outreach status in a structured database.
4. Give owners a dashboard to review and approve leads before outreach.
5. Use SendGrid to send approved email outreach.
6. Track follow-ups in a lightweight CRM pipeline.
7. Support future expansion to a larger lead-data stack only after the pilot proves what is missing.

The pilot should focus on a practical, working workflow rather than enterprise-grade automation.

---

## 3. Core Business Use Case

The pallet company operates in and around Atlanta, Georgia. It serves businesses that need new, refurbished, recycled, or replacement pallets, and it collects pallets from companies that have excess, broken, or unused pallets.

The business may work with:

- Warehouses
- Manufacturers
- Distribution centers
- Food and beverage distributors
- Grocery distributors
- Retail distribution operations
- Building supply companies
- Construction suppliers
- Logistics companies
- Freight and shipping companies
- Industrial parks
- Recycling yards
- Wholesale suppliers
- Produce distributors
- Packaging companies
- Local businesses with recurring pallet buildup

The platform should help identify businesses that may fit one or more of these categories.

---

## 4. Pilot Lead Source Policy

The pilot will use a limited and controlled set of lead sources.

### Included Lead Sources

1. **Google Places API**
   - Used to discover businesses by location, category, search term, and proximity.
   - Used to collect basic public business information such as name, address, phone number, website, rating, business type, and place metadata where available.

2. **Manual CSV Import**
   - Used for owner-provided lists, manually researched leads, purchased lists, or spreadsheet-based contacts.
   - CSV import should support basic validation and duplicate detection.

### Deferred Lead Sources

The following third-party B2B lead-data platforms are intentionally excluded from the pilot:

- Apollo
- ZoomInfo
- Clearbit
- People Data Labs / PDL
- Data Axle
- Clay
- Hunter

These providers may be reviewed in a later phase only after the pilot identifies actual gaps in lead volume, contact completeness, email quality, conversion rate, or data enrichment needs.

### Reason for Deferral

The pilot should avoid unnecessary cost and integration complexity. Google Places plus CSV import is sufficient to validate the business workflow for Atlanta-area pallet lead discovery.

---

## 5. Technology Stack

### Database and Backend

**Supabase**

Supabase will be used for:

- PostgreSQL database
- Authentication
- Role-based access policies
- API access
- File storage if needed
- Realtime dashboard updates if needed
- Audit logging where appropriate

### Email Delivery

**SendGrid**

SendGrid will be used for:

- Owner-approved outbound emails
- Email templates
- Delivery tracking
- Bounce tracking
- Suppression handling
- Unsubscribe handling
- Sender/domain authentication

### AI / Agent Layer

**Gemini 2.5 Agents**

Gemini 2.5 agents will be used for:

- Lead classification
- Lead summarization
- Contact research assistance
- Outreach draft generation
- Routing recommendations
- Follow-up recommendations
- Data cleanup support

Gemini agents should not send emails automatically. Agents prepare, classify, summarize, and recommend. Owners approve before outreach.

### Maps / Location

**Google Places API and Google Maps-related services**

Used for:

- Business discovery
- Address capture
- Geocoding
- Distance grouping
- Regional search
- Route planning support

For the pilot, routing can be approximate and should prioritize business usefulness over complex fleet logistics.

### Frontend

Suggested frontend options:

- Next.js
- React
- Tailwind CSS
- Supabase client libraries

The exact frontend framework may be adjusted by the build team, but the dashboard must be web-based and owner-accessible.

---

## 6. User Roles

### Owner / Admin

The owner can:

- View all leads
- Approve or reject leads
- Edit lead details
- Add notes
- Assign status
- Approve email outreach
- View follow-up tasks
- View routing groups
- Export lead/contact lists
- Review email activity
- Manage suppression or do-not-contact status

### Operator / Staff

Staff can:

- Add leads
- Import CSV files
- Research leads
- Update CRM statuses
- Add notes
- Prepare leads for owner review

Staff should not be able to send outreach unless granted permission.

### System / Agent

Agents can:

- Suggest classifications
- Draft outreach
- Score leads
- Flag missing data
- Summarize company fit
- Recommend next action

Agents cannot:

- Send emails without approval
- Delete records without human confirmation
- Override do-not-contact status
- Bypass compliance rules

---

## 7. Core Platform Modules

## 7.1 Lead Discovery Module

### Purpose

Find potential pallet buyers, pallet suppliers, and pallet pickup sources.

### MVP Inputs

- Search city or ZIP code
- Search radius or route zone
- Business category
- Search phrase
- Target lead type
- Optional notes from owner

### Example Search Terms

- Warehouse near Atlanta GA
- Distribution center near Atlanta GA
- Food distributor near Atlanta GA
- Manufacturer near Atlanta GA
- Logistics company near Atlanta GA
- Grocery distribution near Atlanta GA
- Building supply company near Atlanta GA
- Freight company near Atlanta GA
- Packaging company near Atlanta GA
- Produce distributor near Atlanta GA
- Industrial supplier near Atlanta GA

### Lead Output Fields

Each discovered lead should include:

- Business name
- Business category
- Address
- City
- State
- ZIP code
- Phone number
- Website
- Google Places ID
- Latitude
- Longitude
- Source
- Discovery search term
- Date discovered
- Initial lead type suggestion
- Confidence score
- Agent notes
- Owner review status

---

## 7.2 CSV Import Module

### Purpose

Allow the owner or staff to upload lead/contact spreadsheets.

### Supported Fields

The CSV importer should support the following columns where available:

- Company name
- Contact name
- Contact title
- Email
- Phone
- Website
- Address
- City
- State
- ZIP
- Industry
- Lead type
- Notes
- Source
- Existing relationship
- Do not contact flag

### Import Requirements

The importer should:

- Preview records before import
- Detect possible duplicates
- Validate email format
- Validate required fields
- Allow mapping of custom columns
- Tag imported leads by batch
- Save import history

---

## 7.3 Lead Classification Agent

### Purpose

Classify each lead based on likelihood of pallet relevance.

### Classification Categories

A lead may be classified as one or more of the following:

- Pallet buyer
- Pallet source
- Broken pallet pickup opportunity
- Refurbished pallet buyer
- New pallet buyer
- Recycled pallet buyer
- Recurring pickup opportunity
- One-time pickup opportunity
- Partner opportunity
- Low priority
- Not relevant

### Classification Inputs

- Business name
- Business category
- Website
- Google Places type
- Location
- Search term used
- Notes
- Imported CSV data

### Classification Output

The agent should output structured JSON including:

```json
{
  "lead_type": "pallet_source",
  "secondary_types": ["broken_pallet_pickup_opportunity"],
  "confidence_score": 0.82,
  "reasoning_summary": "The business appears to operate warehouse or distribution activity and may accumulate pallets.",
  "recommended_next_action": "Owner review before contact",
  "risk_flags": []
}
```

---

## 7.4 Contact Research Module

### Purpose

Collect public contact details for each lead.

### Contact Fields

- Contact name
- Title
- Email
- Phone
- Contact page URL
- General business email
- Website form URL
- Source URL
- Contact confidence score
- Last verified date

### Pilot Limitation

The pilot should focus on public contact data available from:

- Google Places
- Business website
- Contact page
- Owner-provided CSV files
- Manually entered information

The pilot should not rely on paid contact databases.

---

## 7.5 Owner Approval Workflow

### Purpose

Prevent unapproved outreach and give the owners control.

### Lead Review Statuses

- New
- Researching
- Ready for owner review
- Approved for outreach
- Rejected
- Needs more information
- Do not contact

### Approval Rules

- No outbound email may be sent until the lead is approved.
- No email may be sent to a contact marked do-not-contact.
- Rejected leads should remain stored but excluded from outreach.
- Owners should be able to approve leads individually or in batches.
- Owners should be able to edit the lead before approval.

---

## 7.6 Outreach Drafting Agent

### Purpose

Generate email drafts for approved leads.

### Email Types

- Pallet pickup inquiry
- Broken pallet removal inquiry
- Refurbished pallet sales introduction
- New pallet sales introduction
- General partnership introduction
- Follow-up email
- Re-engagement email

### Email Draft Requirements

Each email draft should include:

- Subject line
- Email body
- Personalization note
- Suggested call to action
- Lead type
- Tone
- Compliance check result

### Required Tone

Professional, simple, local, practical, and business-to-business.

### Sample Outreach Angle

For a pallet source:

> We help local businesses remove excess, broken, or unused pallets and can evaluate whether there is an ongoing pickup opportunity.

For a pallet buyer:

> We supply new and refurbished pallets to local businesses and may be able to support recurring pallet needs.

### Sending Rule

The agent drafts the message. The owner approves the final message. SendGrid sends the approved message.

---

## 7.7 SendGrid Email Module

### Purpose

Send approved outbound email and track status.

### Required SendGrid Features

- API-based email sending
- Sender authentication
- Template support
- Unsubscribe link
- Suppression group handling
- Bounce tracking
- Open/click tracking if enabled
- Webhook event ingestion

### Email Statuses

- Drafted
- Pending approval
- Approved
- Queued
- Sent
- Delivered
- Opened
- Clicked
- Bounced
- Deferred
- Unsubscribed
- Spam reported
- Failed

### Compliance Requirements

Each commercial email should include:

- Accurate sender name
- Accurate reply-to address
- Non-misleading subject line
- Business physical mailing address
- Clear unsubscribe option
- Suppression check before send

### Send Restrictions

The system must not send to:

- Unsubscribed contacts
- Suppressed contacts
- Do-not-contact records
- Rejected leads
- Leads not approved by owner

---

## 7.8 Lightweight CRM Module

### Purpose

Track relationship status and follow-up activity.

### CRM Lead Stages

- New
- Researching
- Needs owner review
- Approved for outreach
- Contacted
- Follow-up needed
- Interested
- Pickup scheduled
- Quote requested
- Negotiating
- Won
- Lost
- Do not contact

### Activity Types

- Note added
- Call placed
- Email drafted
- Email sent
- Email opened
- Email clicked
- Reply received
- Follow-up scheduled
- Owner approved
- Owner rejected
- Pickup scheduled
- Quote requested
- Deal won
- Deal lost

### Follow-up Fields

- Follow-up date
- Assigned user
- Follow-up type
- Follow-up note
- Priority
- Related email
- Related lead
- Status

---

## 7.9 Routing and Territory Module

### Purpose

Help owners understand where opportunities are located and group leads by practical drive routes.

### Pilot Routing Goals

The routing module should help answer:

- Which leads are near each other?
- Which leads are within the Atlanta core market?
- Which leads are within a three-hour drive of Atlanta?
- Which leads could be combined into a pickup or sales route?
- Which areas have enough lead density to justify outreach?

### Route Grouping Fields

- Route zone name
- Origin point
- Estimated drive distance
- Estimated drive time
- Lead count
- Pickup opportunity count
- Buyer opportunity count
- Priority score
- Suggested visit order
- Notes

### Suggested Route Zones

- Atlanta core
- North Atlanta
- South Atlanta
- East Atlanta
- West Atlanta
- Northwest Georgia
- Northeast Georgia
- Middle Georgia
- Chattanooga-adjacent region
- Alabama-border region
- South Carolina-border region

The exact route zones can be adjusted based on actual lead density.

---

## 7.10 Dashboard Module

### Purpose

Give owners a clear view of leads, approvals, outreach, and follow-up work.

### Dashboard Views

1. **Lead Overview**
   - Total leads
   - New leads
   - Leads ready for review
   - Approved leads
   - Contacted leads
   - Interested leads
   - Won opportunities

2. **Owner Review Queue**
   - Leads needing approval
   - Missing information
   - Agent summary
   - Suggested next action
   - Approve/reject controls

3. **Lead Map / Territory View**
   - Leads by location
   - Lead type filters
   - Route group filters
   - Three-hour drive zone indicator where possible

4. **CRM Pipeline**
   - Leads by stage
   - Follow-up tasks
   - Overdue follow-ups
   - Recent activity

5. **Email Activity**
   - Drafted emails
   - Approved emails
   - Sent emails
   - Delivered emails
   - Bounced emails
   - Unsubscribes
   - Replies if integrated

6. **Import History**
   - CSV import batches
   - Record count
   - Duplicates found
   - Errors
   - Imported by
   - Date imported

---

## 8. Database Design

Supabase PostgreSQL should include the following core tables.

## 8.1 organizations

Stores company account information for the pallet business.

Suggested fields:

- id
- name
- business_address
- city
- state
- zip
- phone
- website
- created_at
- updated_at

---

## 8.2 users

Managed through Supabase Auth with profile data stored separately if needed.

Suggested fields:

- id
- organization_id
- full_name
- email
- role
- is_active
- created_at
- updated_at

---

## 8.3 leads

Stores business leads.

Suggested fields:

- id
- organization_id
- business_name
- business_category
- lead_type
- secondary_lead_types
- status
- owner_review_status
- confidence_score
- priority_score
- source
- source_detail
- google_place_id
- website
- phone
- address_line_1
- address_line_2
- city
- state
- zip
- latitude
- longitude
- distance_from_atlanta
- estimated_drive_time
- route_zone
- agent_summary
- recommended_next_action
- do_not_contact
- created_at
- updated_at

---

## 8.4 contacts

Stores people or general contact points related to leads.

Suggested fields:

- id
- organization_id
- lead_id
- first_name
- last_name
- title
- email
- phone
- contact_type
- source
- source_url
- confidence_score
- email_status
- do_not_contact
- last_verified_at
- created_at
- updated_at

---

## 8.5 lead_notes

Stores notes from owners, staff, or agents.

Suggested fields:

- id
- organization_id
- lead_id
- user_id
- note_type
- note_body
- created_at

---

## 8.6 activities

Stores CRM timeline activity.

Suggested fields:

- id
- organization_id
- lead_id
- contact_id
- user_id
- activity_type
- activity_summary
- activity_metadata
- created_at

---

## 8.7 follow_ups

Stores follow-up tasks.

Suggested fields:

- id
- organization_id
- lead_id
- contact_id
- assigned_to
- due_date
- priority
- status
- follow_up_type
- notes
- completed_at
- created_at
- updated_at

---

## 8.8 email_drafts

Stores AI-generated and human-edited email drafts.

Suggested fields:

- id
- organization_id
- lead_id
- contact_id
- draft_type
- subject
- body
- personalization_notes
- compliance_status
- approval_status
- approved_by
- approved_at
- created_by_agent
- created_at
- updated_at

---

## 8.9 email_events

Stores SendGrid delivery and engagement events.

Suggested fields:

- id
- organization_id
- lead_id
- contact_id
- email_draft_id
- sendgrid_message_id
- event_type
- event_timestamp
- event_metadata
- created_at

---

## 8.10 imports

Stores CSV import batch history.

Suggested fields:

- id
- organization_id
- file_name
- imported_by
- total_rows
- imported_rows
- duplicate_rows
- error_rows
- import_status
- created_at

---

## 8.11 route_groups

Stores route and territory groupings.

Suggested fields:

- id
- organization_id
- name
- origin_address
- route_zone
- estimated_drive_time
- estimated_distance
- lead_count
- pickup_count
- buyer_count
- priority_score
- notes
- created_at
- updated_at

---

## 8.12 compliance_suppressions

Stores suppression and do-not-contact records.

Suggested fields:

- id
- organization_id
- email
- phone
- suppression_type
- source
- reason
- created_at

Suppression types may include:

- unsubscribe
- bounce
- spam_report
- manual_do_not_contact
- owner_rejected
- invalid_email

---

## 9. Agent Specifications

## 9.1 Lead Discovery Agent

### Role

Find potential pallet-related businesses using approved sources.

### Inputs

- Location
- Search terms
- Business categories
- Distance or route zone
- Target lead type
- Current database records for duplicate checking

### Outputs

- Candidate lead records
- Search source
- Discovery reason
- Confidence score
- Suggested lead category
- Missing fields

### Guardrails

- Use only approved pilot sources.
- Do not create duplicate records when a likely duplicate exists.
- Do not mark a lead as approved.
- Do not send outreach.

---

## 9.2 Lead Classification Agent

### Role

Classify leads by pallet business relevance.

### Inputs

- Business profile
- Google Places data
- Website text if available
- Existing notes

### Outputs

- Lead type
- Secondary lead types
- Confidence score
- Reasoning summary
- Recommended next action
- Risk flags

### Guardrails

- Do not overstate certainty.
- Mark unclear leads as needing owner review.
- Avoid fabricating details that are not present.

---

## 9.3 Contact Research Agent

### Role

Find public contact information and summarize available communication paths.

### Inputs

- Lead name
- Website
- Phone
- Address
- Existing contact fields

### Outputs

- Contact page URL
- General email if public
- Phone number
- Contact notes
- Missing contact fields
- Confidence score

### Guardrails

- Use public data only.
- Do not scrape behind logins.
- Do not use excluded paid lead databases during the pilot.
- Do not guess personal emails.

---

## 9.4 Outreach Drafting Agent

### Role

Draft useful outreach emails for approved leads.

### Inputs

- Lead type
- Business name
- Contact details
- Owner-approved offer
- Desired call to action
- Compliance rules

### Outputs

- Subject line
- Email body
- Personalization notes
- Compliance checklist
- Recommended follow-up timing

### Guardrails

- Do not send email.
- Do not use misleading claims.
- Do not promise pricing or service terms unless provided by owner.
- Include unsubscribe language and business address placeholder if required by template.

---

## 9.5 Routing Agent

### Role

Group leads into useful geographic clusters and suggest practical route opportunities.

### Inputs

- Lead addresses
- Coordinates
- Drive-time data if available
- Lead types
- Priority score
- Owner route preferences

### Outputs

- Route group suggestions
- Cluster summaries
- Priority notes
- Suggested route order
- Drive-time estimate

### Guardrails

- Treat routing as advisory in the pilot.
- Do not guarantee exact travel times.
- Flag records missing accurate address or coordinates.

---

## 9.6 CRM Follow-up Agent

### Role

Recommend follow-up actions and help maintain pipeline momentum.

### Inputs

- Lead status
- Last activity
- Email status
- Owner notes
- Follow-up schedule

### Outputs

- Recommended next action
- Suggested follow-up date
- Suggested note
- Priority level

### Guardrails

- Do not change a deal to won or lost without human action.
- Do not contact suppressed leads.
- Do not send reminders externally unless enabled.

---

## 9.7 Compliance Agent

### Role

Check outreach and contact records against pilot compliance rules.

### Inputs

- Email draft
- Contact record
- Suppression table
- Lead approval status
- Sender information

### Outputs

- Pass/fail compliance status
- Missing compliance items
- Suppression warnings
- Recommended correction

### Guardrails

- Block sending when suppression or do-not-contact status exists.
- Block sending when owner approval is missing.
- Block sending when unsubscribe language is missing.
- Block sending when sender identity or business address is missing.

---

## 10. Required Workflows

## 10.1 Discover Leads Workflow

1. User selects a location, search term, and category.
2. System queries Google Places.
3. System normalizes results.
4. System checks for duplicates in Supabase.
5. Lead Discovery Agent prepares candidate records.
6. Lead Classification Agent classifies lead type.
7. Leads are saved as `New` or `Ready for owner review`.
8. Owners review leads in dashboard.

---

## 10.2 CSV Import Workflow

1. User uploads CSV.
2. System previews rows.
3. User maps columns if needed.
4. System validates records.
5. System checks duplicates.
6. User confirms import.
7. System saves records to Supabase.
8. Agents classify and score imported leads.
9. Imported leads appear in the review queue.

---

## 10.3 Owner Approval Workflow

1. Owner opens review queue.
2. Owner reviews lead details, contact info, agent summary, and recommended next action.
3. Owner chooses:
   - Approve for outreach
   - Reject
   - Mark do-not-contact
   - Request more information
   - Edit lead
4. System logs owner action.
5. Approved leads become eligible for outreach drafting.

---

## 10.4 Email Outreach Workflow

1. Approved lead is selected for email drafting.
2. Outreach Drafting Agent generates draft.
3. Compliance Agent checks draft.
4. Owner reviews and edits draft.
5. Owner approves final message.
6. SendGrid sends email.
7. SendGrid event webhooks update Supabase.
8. CRM activity timeline is updated.
9. Follow-up task is created.

---

## 10.5 CRM Follow-up Workflow

1. Lead enters contacted stage.
2. System creates follow-up task.
3. Owner or staff logs call, reply, or note.
4. Lead status is updated.
5. Follow-up Agent recommends next action.
6. Lead is advanced, closed, or marked do-not-contact.

---

## 11. Dashboard Requirements

## 11.1 Main Dashboard

The main dashboard should show:

- Total leads
- Leads by status
- Leads by type
- Leads needing owner review
- Approved leads
- Recently contacted leads
- Follow-ups due
- Overdue follow-ups
- Email delivery summary
- Route group summary

---

## 11.2 Lead Table

The lead table should support:

- Search
- Filter by status
- Filter by lead type
- Filter by city
- Filter by route zone
- Filter by source
- Filter by owner review status
- Sort by priority score
- Sort by date added
- Bulk approval where appropriate
- Export to CSV

---

## 11.3 Lead Detail Page

Each lead detail page should show:

- Business profile
- Contact records
- Address and map
- Agent summary
- Lead classification
- Owner approval status
- Notes
- Email drafts
- Email events
- CRM activity timeline
- Follow-up tasks
- Route group

---

## 11.4 Owner Review Queue

The owner review queue should show:

- Lead name
- Lead type
- Confidence score
- Business category
- Location
- Contact completeness
- Agent summary
- Recommended next action
- Approve button
- Reject button
- Needs more info button
- Do-not-contact button

---

## 11.5 CRM Pipeline

The CRM pipeline should show leads grouped by stage:

- New
- Researching
- Needs owner review
- Approved for outreach
- Contacted
- Follow-up needed
- Interested
- Pickup scheduled
- Quote requested
- Won
- Lost
- Do not contact

---

## 12. Security and Permissions

### Authentication

Supabase Auth should be used for login and account management.

### Authorization

Role-based access should be enforced.

### Row Level Security

Supabase Row Level Security policies should be enabled for organization-specific data separation.

### Audit Trail

The system should log important actions, including:

- Lead creation
- Lead update
- Owner approval
- Rejection
- Do-not-contact marking
- Email approval
- Email send event
- CSV import
- User role change

---

## 13. Compliance Requirements

The platform should include basic compliance controls for outbound commercial email.

### Email Compliance

The platform should require:

- Accurate sender identity
- Accurate reply-to email
- Valid physical business mailing address in templates
- Clear unsubscribe language
- Suppression checks before sending
- No misleading subject lines
- Owner approval before sending

### Suppression Handling

The platform should store and honor:

- Unsubscribes
- Bounces
- Spam reports
- Manual do-not-contact flags
- Owner-rejected leads
- Invalid emails

### Contact Data Compliance

The pilot should only use approved sources:

- Google Places
- Public business websites
- Owner-provided CSV data
- Manually entered data

The pilot should not use unauthorized scraping, private datasets, or paid B2B contact databases unless later approved as a separate scope.

---

## 14. Acceptance Criteria

The pilot is considered complete when the following are working:

### Lead Discovery

- User can run a Google Places-based lead search.
- Results are saved into Supabase.
- Duplicate checks are performed.
- Leads include business name, address, phone, website where available, source, and location data.

### CSV Import

- User can upload a CSV file.
- System can map and preview fields.
- System imports valid records.
- System identifies possible duplicates.

### Lead Classification

- Agents classify leads into pallet-relevant categories.
- Each classified lead includes a confidence score and summary.
- Unclear leads are flagged for owner review.

### Owner Dashboard

- Owner can view all leads.
- Owner can approve, reject, or request more information.
- Owner can mark leads as do-not-contact.
- Owner can view lead detail pages and activity.

### Outreach

- System can generate email drafts.
- Owner can approve drafts.
- SendGrid can send approved emails.
- Suppressed or unapproved contacts are blocked from sending.

### CRM

- Leads can move through CRM stages.
- Notes and activities can be logged.
- Follow-up tasks can be created and completed.

### Routing

- Leads can be grouped by city, zone, or distance.
- System can identify leads within the target Atlanta service area and broader three-hour expansion zone.
- Route grouping is available at a basic pilot level.

---

## 15. Out of Scope for Pilot

The following are not included in the pilot unless added by separate agreement:

- Apollo integration
- ZoomInfo integration
- Clearbit integration
- People Data Labs integration
- Data Axle integration
- Clay integration
- Hunter integration
- Full enterprise CRM replacement
- Automated SMS outreach
- Automated phone calling
- AI voice agents
- Payment processing
- Inventory management
- Pallet manufacturing workflow management
- Driver dispatch system
- Fleet tracking
- Advanced route optimization
- Complex quote generation
- Contract management
- Customer portal
- Vendor portal
- Mobile app
- Multi-branch enterprise support
- Automated email sending without owner approval
- Data scraping behind logins or paywalls
- Legal advice or legal compliance certification

---

## 16. Future Phase Options

Future phases may include:

- Paid lead-data provider evaluation
- Email verification provider
- Reply parsing
- Advanced routing optimization
- Quote generation
- Pickup scheduling
- Inventory tracking
- Customer account portal
- Driver route mobile view
- Integration with accounting software
- Integration with existing CRM
- Advanced analytics
- Multi-user assignment workflows
- Call tracking
- SMS outreach with proper consent controls
- Website lead capture forms
- Landing pages by service area
- SEO content pages for pallet pickup and pallet sales
- Automated reporting to owners

---

## 17. MOU Source Terms

The following section can be used to prepare a memorandum of understanding.

### Project Name

Pallet Lead Generation Platform Pilot

### Client

[Client Legal Name]

### Service Provider

[Service Provider Legal Name]

### Project Purpose

To design and build a pilot lead generation, owner approval, email outreach, routing, and lightweight CRM platform for a pallet business operating in Atlanta, Georgia and surrounding markets.

### Scope Summary

The service provider will build a web-based pilot platform using Supabase, SendGrid, Google Places API, manual CSV import, and Gemini 2.5 agent workflows to support discovery, classification, owner review, approved outreach, and CRM follow-up for pallet-related business leads.

### Included Deliverables

- Supabase database schema
- User authentication and roles
- Lead discovery module using Google Places
- Manual CSV import module
- Lead classification agent workflow
- Contact research support workflow
- Owner approval dashboard
- Email drafting workflow
- SendGrid sending integration
- Basic email event tracking
- Lightweight CRM pipeline
- Follow-up task tracking
- Basic routing or territory grouping
- Compliance suppression checks
- Deployment-ready project handoff
- Basic usage documentation

### Excluded Deliverables

- Paid lead-provider integrations
- Full CRM replacement
- Automated calling or texting
- Advanced fleet routing
- Payment processing
- Inventory management
- Mobile app
- Legal compliance certification
- Ongoing list purchasing
- Ongoing outreach campaign management unless separately agreed

### Client Responsibilities

The client should provide:

- Business name and contact information
- Approved sender email address
- Physical business mailing address for emails
- Brand/logo if available
- Service descriptions
- Pallet pickup and pallet sales offer details
- Target service areas
- Preferred lead categories
- Any existing lead spreadsheets
- SendGrid account access or API key
- Supabase project access or approval to create project
- Google API key or billing project access
- Gemini API access or approval to create/configure access
- Review and approval of email templates
- Timely feedback on pilot testing

### Service Provider Responsibilities

The service provider should:

- Build the platform according to the agreed pilot scope
- Configure core database tables
- Implement lead discovery and CSV import workflows
- Implement owner approval before outreach
- Configure SendGrid integration
- Implement basic CRM follow-up tracking
- Implement basic routing/territory grouping
- Provide documentation for use and handoff
- Identify risks, limitations, and recommended future improvements

### Acceptance Criteria

The project will be accepted when the pilot platform demonstrates the core workflows listed in the Acceptance Criteria section of this document.

### Change Control

Any feature not included in the pilot scope should be treated as a change request or future phase.

Examples of change requests include:

- Additional third-party lead provider integrations
- Advanced routing optimization
- SMS or phone integrations
- Payment tools
- Inventory tools
- New dashboards beyond agreed scope
- Complex reporting
- Additional compliance workflows
- Major UI redesign after approval

### Ownership

Final ownership terms should be defined in the MOU.

Suggested topics to define:

- Code ownership
- Database ownership
- Client data ownership
- API account ownership
- Hosting account ownership
- Ongoing maintenance responsibility
- Third-party subscription responsibility

### Confidentiality

Both parties should treat business data, lead lists, customer lists, credentials, and platform access as confidential.

### Third-Party Costs

The client should be responsible for third-party usage costs unless otherwise agreed.

Possible third-party costs include:

- Supabase
- SendGrid
- Google Places / Maps APIs
- Gemini API
- Domain or email authentication services
- Hosting
- Any future paid lead-data providers

### Maintenance

Ongoing maintenance, monitoring, email deliverability management, data cleanup, campaign management, and future enhancements should be handled under a separate agreement unless explicitly included.

---

## 18. Invoice Source Line Items

The following line items can be used to prepare an invoice. Pricing may be added separately.

| Line Item | Description | Qty | Unit |
|---|---|---:|---|
| Discovery and Technical Planning | Requirements review, pilot scope definition, platform architecture, workflow planning | 1 | Project |
| Supabase Backend Setup | Database schema, auth configuration, roles, RLS planning, core tables | 1 | Project |
| Lead Discovery Module | Google Places lead search workflow and lead record creation | 1 | Project |
| CSV Import Module | CSV upload, field mapping, preview, validation, duplicate detection | 1 | Project |
| AI Agent Workflows | Gemini-based lead classification, summarization, outreach draft support, follow-up recommendations | 1 | Project |
| Owner Dashboard | Lead review queue, approvals, lead table, lead detail views, pipeline visibility | 1 | Project |
| SendGrid Integration | Approved outbound email sending, template structure, event tracking, suppression handling | 1 | Project |
| Lightweight CRM | Lead stages, notes, activity timeline, follow-up task tracking | 1 | Project |
| Routing / Territory Grouping | Basic geographic grouping and route-zone support for Atlanta and surrounding service areas | 1 | Project |
| Compliance Controls | Owner approval gate, suppression checks, unsubscribe/do-not-contact handling | 1 | Project |
| Testing and Handoff | Pilot testing, acceptance review, deployment support, basic documentation | 1 | Project |

---

## 19. Environment Variables and Credentials

The build may require the following environment variables:

```bash
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
SENDGRID_API_KEY=
SENDGRID_FROM_EMAIL=
SENDGRID_FROM_NAME=
SENDGRID_UNSUBSCRIBE_GROUP_ID=
GOOGLE_PLACES_API_KEY=
GOOGLE_MAPS_API_KEY=
GEMINI_API_KEY=
APP_BASE_URL=
BUSINESS_PHYSICAL_ADDRESS=
```

Credentials should not be committed to source control.

---

## 20. Pilot Success Metrics

The pilot should be evaluated using:

- Number of leads discovered
- Percentage of leads classified as relevant
- Percentage of leads approved by owner
- Percentage of leads with usable contact information
- Email bounce rate
- Email reply rate
- Number of interested businesses
- Number of pickup opportunities identified
- Number of pallet buyer opportunities identified
- Follow-up completion rate
- Owner satisfaction with lead quality
- Lead quality by category and route zone

---

## 21. Recommended Pilot Build Order

1. Confirm final scope and required accounts.
2. Set up Supabase project and schema.
3. Build authentication and role structure.
4. Build lead table and lead detail views.
5. Build Google Places lead discovery workflow.
6. Build CSV import workflow.
7. Add Gemini lead classification workflow.
8. Add owner approval queue.
9. Add outreach draft generation.
10. Add SendGrid sending after approval.
11. Add email event tracking.
12. Add CRM statuses, notes, and follow-ups.
13. Add basic routing/territory grouping.
14. Test complete workflow.
15. Deliver documentation and handoff.

---

## 22. Key Build Principles

- Keep the pilot focused.
- Avoid unnecessary paid lead-data integrations.
- Use owner approval before sending outreach.
- Store everything in Supabase.
- Use SendGrid only for approved messages.
- Treat AI agents as assistants, not autonomous decision-makers.
- Track compliance status.
- Track do-not-contact and unsubscribe status.
- Prefer clear workflows over complex automation.
- Build for future expansion without overbuilding the pilot.

---

## 23. Final Pilot Summary

This pilot will create a practical lead generation and CRM workflow for a pallet business in Atlanta, Georgia.

The platform will discover potential pallet-related business opportunities using Google Places and manual CSV import, classify and summarize leads using Gemini 2.5 agents, store structured data in Supabase, allow owners to review and approve opportunities, send approved outreach through SendGrid, and track follow-ups in a lightweight CRM dashboard.

The pilot intentionally excludes paid B2B lead-data platforms and advanced enterprise features so the business can first validate lead quality, owner workflow, routing usefulness, and outreach performance.
