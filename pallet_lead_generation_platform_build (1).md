# Pallet Lead Generation & Route CRM Platform — Build Specification

**Working name:** Pallet Growth OS  
**Business:** Atlanta-area pallet company that buys, supplies, collects, repairs/refurbishes, builds, and sells pallets  
**Primary service area:** Atlanta, Georgia with configurable expansion up to a **3-hour drive-time radius**  
**Database/backend:** Supabase  
**Outbound email provider:** SendGrid  
**Document version:** v1.0  
**Date:** 2026-05-24

---

## 1. Executive Summary

Build a lead generation, routing, and lightweight CRM platform for a pallet business based in Atlanta, Georgia.

The platform will discover potential pallet **buyers**, **pickup sources**, **suppliers**, and **partners** within Atlanta and up to a 3-hour drive-time service area. It will gather company details, contact information, contact channels, estimated pallet opportunity, distance/route fit, and outreach readiness. Owners will review and approve leads before any email outreach is sent through SendGrid.

The system must support the pallet company’s three core revenue/supply motions:

1. **Collect / buy / pickup pallets**
   - Find businesses with used, excess, broken, or unwanted pallets.
   - Identify companies that allow recurring pallet pickup.
   - Support one-time cleanup and recurring pickup relationships.

2. **Refurbish and resell pallets**
   - Track source quality, pallet volume, and pallet condition.
   - Convert pickup sources into supply accounts.
   - Match refurbished pallet inventory to buyer demand.

3. **Build and sell new pallets**
   - Find businesses that purchase standard or custom new pallets.
   - Track pallet size, grade, quantity, delivery cadence, and buyer type.

The final product should feel like a focused CRM plus lead intelligence system, not a generic spreadsheet.

---

## 2. Product Goals

### 2.1 Business Goals

- Generate a steady list of qualified pallet leads in and around Atlanta.
- Categorize leads by likely value:
  - pallet pickup source
  - used pallet supplier
  - refurbished pallet buyer
  - new pallet buyer
  - logistics/transport partner
  - recycling/waste partner
- Build owner-ready contact lists with:
  - company name
  - address
  - website
  - phone
  - email/contact form
  - possible decision-maker role
  - industry/category
  - distance from base
  - drive time
  - estimated pallet volume
  - recommended next action
- Route leads and pickup opportunities based on demand, distance, and truck efficiency.
- Track all follow-up activity in a lightweight CRM.
- Prevent unapproved or non-compliant outreach.

### 2.2 Product Goals

- Search for and enrich potential leads using authorized data sources.
- Score each lead for pallet relevance and route fit.
- Present leads to owners for approval before contact.
- Send approved email outreach using SendGrid.
- Record email events, replies, bounces, suppressions, unsubscribes, and follow-up tasks.
- Provide dashboards for lead generation, sales pipeline, pickup pipeline, routing efficiency, and campaign performance.
- Create agent workflows that can run semi-autonomously with human review gates.

---

## 3. Non-Negotiable Constraints

1. **No outbound email without owner approval.**
2. **No outreach to suppressed, unsubscribed, bounced, or do-not-contact contacts.**
3. **All commercial email must include compliant sender identity, physical mailing address, and unsubscribe mechanism.**
4. **No scraping behind login walls or violating platform terms.**
5. **No unsupported claims in emails.**
6. **Every lead must show data provenance.**
7. **Every AI-generated recommendation must be auditable.**
8. **Owners must be able to override lead scores, statuses, categories, and follow-up tasks.**
9. **All route calculations must use drive time, not simple radius distance.**
10. **The 3-hour service range must be configurable.**

---

## 4. Recommended Tech Stack

### 4.1 Frontend

Recommended:

- **Next.js**
- **TypeScript**
- **Tailwind CSS**
- **shadcn/ui**
- **Mapbox, Google Maps JavaScript API, or equivalent mapping UI**
- **React Query / TanStack Query** for server state
- **Recharts** for charts

### 4.2 Backend

Recommended:

- **Supabase Postgres** as primary database
- **Supabase Auth** for owner/admin/user login
- **Supabase Row Level Security**
- **Supabase Edge Functions** for secure API operations
- **Supabase Realtime** for dashboards and job status updates
- **Supabase Storage** for uploaded CSVs, exports, and generated reports
- **Optional Supabase Vector / pgvector** for semantic lead deduplication and company similarity

### 4.3 Email

- **SendGrid Email API**
- **SendGrid domain authentication**
- **SendGrid Advanced Suppression Management**
- **SendGrid Event Webhook**
- **SendGrid Inbound Parse Webhook** if reply handling is needed through a monitored domain/subdomain

### 4.4 Lead Discovery / Enrichment Sources

Use only authorized or licensed sources.

Recommended source categories:

- Google Places API / Places Nearby Search
- Google Business Profile-derived public place details through approved APIs
- Company websites
- Public contact pages
- Public directories with permitted use
- Georgia business records where appropriate
- County/city business license datasets where available
- Optional paid providers:
  - Apollo
  - ZoomInfo
  - Clearbit-style enrichment
  - People Data Labs
  - Data Axle
  - Clay
  - Hunter.io
  - BuiltWith
  - Similarweb-style traffic/company data

Do **not** rely on a single lead source. The system should store source confidence and provenance per field.

### 4.5 Routing

Recommended:

- Google Routes API / Distance Matrix style service
- OR Mapbox Optimization API
- OR OpenRouteService
- OR self-hosted OSRM for cost control later

Routing must calculate:

- drive time from Atlanta base
- drive distance
- pickup/delivery time window
- number of stops
- truck capacity
- pallet quantity
- route cluster density
- route profitability

---

## 5. Target Lead Segments

### 5.1 Pickup Source Leads

Businesses likely to have excess, broken, used, or unwanted pallets:

- warehouses
- distribution centers
- 3PLs
- freight terminals
- fulfillment centers
- manufacturers
- food distributors
- beverage distributors
- grocery wholesalers
- produce distributors
- appliance stores
- furniture stores
- building material suppliers
- flooring/tile suppliers
- nurseries/garden centers
- auto parts distributors
- packaging companies
- print shops
- moving and storage companies
- event rental companies
- construction suppliers
- recycling centers
- waste management companies
- industrial parks
- cold storage facilities

### 5.2 Buyer Leads

Businesses likely to purchase used, refurbished, or new pallets:

- manufacturers
- exporters
- warehouses
- distributors
- e-commerce fulfillment centers
- logistics companies
- food and beverage operations
- agricultural suppliers
- packaging companies
- building material distributors
- chemical/non-food industrial suppliers
- retail distribution operations
- pallet brokers
- shipping departments
- custom crating/shipping businesses

### 5.3 Partner Leads

Potential operational or referral partners:

- trucking companies
- local freight carriers
- recycling companies
- junk removal businesses
- warehouse cleanout companies
- property managers for industrial parks
- logistics consultants
- packaging suppliers
- industrial real estate brokers
- waste brokers

---

## 6. Lead Lifecycle / CRM Pipeline

Use CRM lifecycle stages so every company and contact has a clear place in the funnel.

### 6.1 Company Lifecycle

```text
DISCOVERED
→ ENRICHING
→ ENRICHED
→ SCORED
→ OWNER_REVIEW
→ APPROVED
→ CONTACTED
→ RESPONDED
→ QUALIFIED
→ OPPORTUNITY
→ CUSTOMER
→ RECURRING_ACCOUNT
→ INACTIVE
→ DO_NOT_CONTACT
```

### 6.2 Contact Lifecycle

```text
DISCOVERED
→ VERIFIED
→ OWNER_APPROVED
→ CONTACTED
→ REPLIED
→ MEETING_SET
→ NOT_INTERESTED
→ BAD_CONTACT
→ UNSUBSCRIBED
→ DO_NOT_CONTACT
```

### 6.3 Opportunity Types

```text
PALLET_PICKUP
USED_PALLET_SUPPLY
REFURBISHED_PALLET_SALE
NEW_PALLET_SALE
CUSTOM_PALLET_BUILD
RECURRING_PICKUP
RECURRING_DELIVERY
PARTNERSHIP
```

### 6.4 CRM Follow-Up Rules

Default follow-up sequence after owner approval:

| Stage | Timing | Action |
|---|---:|---|
| Approved | Day 0 | Send first email or assign call task |
| Contacted, no reply | Day 3 | Follow-up email |
| Still no reply | Day 7 | Phone call task |
| Still no reply | Day 14 | Final light follow-up |
| No engagement | Day 21 | Move to nurture or inactive |
| Positive reply | Immediate | Assign owner task |
| Pickup interest | Immediate | Create route assessment |
| Buyer interest | Immediate | Create sales opportunity |

---

## 7. Dashboard Requirements

### 7.1 Main Owner Dashboard

Show:

- total leads discovered
- leads awaiting owner review
- approved leads
- contacted leads
- responses
- qualified opportunities
- pickup opportunities
- sales opportunities
- recurring accounts
- email bounce rate
- unsubscribe rate
- next follow-ups due
- top target industries
- route-fit leads within current truck schedule
- estimated pallets available
- estimated pallet demand
- estimated value of pipeline

### 7.2 Lead Review Dashboard

Owners should see a queue with:

- lead name
- lead type
- source confidence
- score
- distance
- drive time
- likely pallet use case
- contact info
- data sources
- recommended outreach angle
- risk/compliance flags
- approve/reject/edit buttons

Actions:

- approve for email
- approve for call only
- reject
- mark as duplicate
- mark as do-not-contact
- request more research
- assign to owner/staff
- schedule follow-up

### 7.3 CRM Kanban

Columns:

- New
- Researching
- Needs Owner Review
- Approved
- Contacted
- Responded
- Qualified
- Pickup Scheduled
- Quote Sent
- Won
- Lost
- Nurture
- Do Not Contact

Cards should display:

- company name
- city/state
- lead type
- estimated pallets/month
- next task
- last contact date
- owner assigned
- urgency
- route fit

### 7.4 Map & Routing Dashboard

Show:

- leads on map
- route clusters
- current customers/sources
- drive-time bands
- 30/60/90/120/180-minute zones
- route profitability
- stops by truck/day
- pickup opportunities by pallet quantity
- suggested weekly service loops

Recommended views:

- Atlanta metro
- North corridor: I-75 / I-575 / GA-400
- Northeast corridor: I-85
- East corridor: I-20
- South corridor: I-75 / I-85
- West corridor: I-20
- Outbound 3-hour expansion zones

### 7.5 Email Campaign Dashboard

Show:

- emails approved
- emails sent
- delivered
- opened
- clicked
- replied
- bounced
- unsubscribed
- spam reports
- positive replies
- negative replies
- follow-ups due
- campaign performance by lead segment

---

## 8. Agent System

The platform should use specialized agents. Agents do not send emails directly unless the owner approval gate has passed.

### 8.1 Agent Control Policy

Every agent must follow these rules:

- Store all outputs in Supabase.
- Add source URLs or source names for each claim.
- Mark confidence level for each field.
- Never overwrite owner-verified fields without creating a proposed update.
- Never contact a lead directly.
- Never send outreach unless the lead, contact, and campaign are approved.
- Respect do-not-contact and suppression flags.
- Log every action in `audit_logs`.

### 8.2 Agent List

| Agent | Purpose | Human Approval Required? |
|---|---|---|
| Territory Agent | Builds search areas and service zones using drive-time logic | No |
| Lead Discovery Agent | Finds potential companies by industry/location | No |
| Enrichment Agent | Finds websites, phone numbers, contact pages, emails, roles | No |
| Deduplication Agent | Merges likely duplicate companies/contacts | Yes for destructive merge |
| Lead Scoring Agent | Scores fit, value, urgency, contactability, route fit | No |
| Compliance Agent | Checks outreach readiness, opt-outs, risky data, missing address/unsubscribe | No |
| Outreach Drafting Agent | Creates email drafts and call notes | Yes before sending |
| Owner Review Agent | Prepares clean approval summaries for owners | Yes |
| SendGrid Campaign Agent | Sends approved emails and records SendGrid IDs | Yes gate before send |
| Inbox / Reply Agent | Classifies replies and creates tasks | No, but owner handles replies |
| CRM Follow-Up Agent | Creates next tasks based on stage rules | No |
| Routing Agent | Creates pickup/sales route suggestions | Yes before dispatch |
| Reporting Agent | Builds owner reports and exports | No |

---

## 9. Agent Prompt Contracts

Use these prompt contracts in the agent implementation. Each agent should receive structured context and return strict JSON.

### 9.1 Territory Agent Prompt

```text
You are the Territory Agent for an Atlanta pallet company.

Goal:
Create drive-time-based lead search territories for pallet pickup, supply, and sales expansion.

Context:
- Business base location: {{base_address}}
- Maximum service range: {{max_drive_minutes}} minutes
- Current customers/sources: {{current_accounts}}
- Target lead categories: {{target_categories}}
- Routing provider output: {{drive_time_data}}

Rules:
- Use drive time, not straight-line distance.
- Prioritize industrial corridors, logistics clusters, warehouse areas, and manufacturing-heavy zones.
- Create territories that are useful for lead search and truck routing.
- Return confidence notes and assumptions.

Output JSON:
{
  "territories": [
    {
      "name": "...",
      "corridor": "...",
      "cities_or_areas": ["..."],
      "drive_time_min": 0,
      "drive_time_max": 0,
      "priority": "high|medium|low",
      "reason": "...",
      "search_keywords": ["..."]
    }
  ],
  "assumptions": ["..."],
  "warnings": ["..."]
}
```

### 9.2 Lead Discovery Agent Prompt

```text
You are the Lead Discovery Agent for a pallet company.

Goal:
Find businesses likely to either have unwanted pallets, need pallet pickup, buy refurbished pallets, or buy new/custom pallets.

Inputs:
- Territory: {{territory}}
- Target industry categories: {{categories}}
- Search source results: {{source_results}}
- Existing database companies: {{existing_companies}}

Rules:
- Only use authorized source data.
- Do not invent companies.
- Mark each lead type: PICKUP_SOURCE, BUYER, BOTH, or PARTNER.
- Include source provenance for every company.
- Flag duplicates.

Output JSON:
{
  "companies": [
    {
      "name": "...",
      "lead_type": "PICKUP_SOURCE|BUYER|BOTH|PARTNER",
      "industry": "...",
      "address": "...",
      "city": "...",
      "state": "GA",
      "website": "...",
      "phone": "...",
      "source": "...",
      "source_url": "...",
      "why_relevant": "...",
      "duplicate_candidate": true,
      "confidence": 0.0
    }
  ]
}
```

### 9.3 Enrichment Agent Prompt

```text
You are the Enrichment Agent.

Goal:
Enrich a company record with contact details and outreach readiness.

Inputs:
- Company record: {{company}}
- Website pages or snippets: {{website_data}}
- Public directory data: {{directory_data}}

Rules:
- Prefer company-owned sources over third-party directories.
- Do not guess personal emails.
- Generic emails like info@, sales@, shipping@, receiving@, warehouse@ are acceptable if sourced.
- Contact forms are valid contact channels.
- Return source and confidence per field.

Output JSON:
{
  "company_updates": {
    "website": {"value": "...", "source": "...", "confidence": 0.0},
    "phone": {"value": "...", "source": "...", "confidence": 0.0},
    "contact_form_url": {"value": "...", "source": "...", "confidence": 0.0}
  },
  "contacts": [
    {
      "name": "...",
      "title": "...",
      "email": "...",
      "phone": "...",
      "source": "...",
      "confidence": 0.0,
      "is_generic": false
    }
  ],
  "warnings": ["..."]
}
```

### 9.4 Lead Scoring Agent Prompt

```text
You are the Lead Scoring Agent.

Goal:
Score a company for pallet business opportunity.

Inputs:
- Company: {{company}}
- Contacts: {{contacts}}
- Industry: {{industry}}
- Distance/drive time: {{route_data}}
- Existing route clusters: {{route_clusters}}
- Owner preferences: {{owner_preferences}}

Scoring dimensions:
- ICP fit
- likely pallet volume
- buyer/source fit
- route fit
- contactability
- urgency signals
- source confidence

Output JSON:
{
  "score_total": 0,
  "score_breakdown": {
    "icp_fit": 0,
    "pallet_volume": 0,
    "buyer_source_fit": 0,
    "route_fit": 0,
    "contactability": 0,
    "urgency": 0,
    "source_confidence": 0
  },
  "lead_type": "PICKUP_SOURCE|BUYER|BOTH|PARTNER",
  "estimated_pallets_month": 0,
  "recommended_next_action": "...",
  "reasoning_summary": "...",
  "confidence": 0.0
}
```

### 9.5 Compliance Agent Prompt

```text
You are the Compliance Agent.

Goal:
Determine whether a lead/contact/campaign is ready for compliant outreach.

Inputs:
- Company: {{company}}
- Contact: {{contact}}
- Campaign: {{campaign}}
- Suppression records: {{suppression_records}}
- Owner approval status: {{approval_status}}
- Email template: {{email_template}}

Rules:
- No outreach without owner approval.
- No outreach if contact is unsubscribed, suppressed, bounced, or do-not-contact.
- Commercial email must include sender identity, accurate subject, physical address, and unsubscribe mechanism.
- Flag risky data sources or missing provenance.
- Do not provide legal advice; provide platform compliance checks.

Output JSON:
{
  "ready_to_send": false,
  "blockers": ["..."],
  "warnings": ["..."],
  "required_fixes": ["..."],
  "compliance_notes": ["..."]
}
```

### 9.6 Outreach Drafting Agent Prompt

```text
You are the Outreach Drafting Agent for a pallet company.

Goal:
Draft a short, professional B2B email for approved leads.

Inputs:
- Company: {{company}}
- Lead type: {{lead_type}}
- Contact: {{contact}}
- Pallet service offerings: {{offerings}}
- Owner-approved claims: {{approved_claims}}
- Compliance footer: {{footer}}

Rules:
- Keep the email concise.
- Do not make unsupported claims.
- Do not mention scraped or private data.
- Use a practical local business tone.
- Include a simple call to action.
- Include required compliance footer and unsubscribe language.

Output JSON:
{
  "subject": "...",
  "preview": "...",
  "body_text": "...",
  "body_html": "...",
  "personalization_used": ["..."],
  "warnings": ["..."]
}
```

### 9.7 Routing Agent Prompt

```text
You are the Routing Agent for a pallet company.

Goal:
Recommend pickup and sales routes based on demand, location, capacity, and profitability.

Inputs:
- Base address: {{base_address}}
- Truck capacity: {{truck_capacity}}
- Available driver hours: {{driver_hours}}
- Approved stops: {{approved_stops}}
- Drive-time matrix: {{drive_time_matrix}}
- Pallet quantities: {{pallet_quantities}}
- Service windows: {{service_windows}}

Rules:
- Prioritize safe, realistic routes.
- Do not exceed truck capacity.
- Use drive-time data, not straight-line distance.
- Group nearby stops where possible.
- Flag stops that are too far unless value justifies the trip.

Output JSON:
{
  "route_name": "...",
  "recommended_date": "...",
  "stops": [
    {
      "company_id": "...",
      "sequence": 1,
      "reason": "...",
      "estimated_arrival": "...",
      "estimated_service_minutes": 0,
      "estimated_pallets": 0
    }
  ],
  "total_drive_minutes": 0,
  "total_service_minutes": 0,
  "total_estimated_pallets": 0,
  "capacity_warning": false,
  "profitability_summary": "...",
  "risks": ["..."]
}
```

---

## 10. Lead Scoring Model

### 10.1 Score Formula

Total score: 0–100

```text
score_total =
  (icp_fit * 0.25) +
  (pallet_volume * 0.20) +
  (buyer_source_fit * 0.15) +
  (route_fit * 0.15) +
  (contactability * 0.10) +
  (urgency * 0.10) +
  (source_confidence * 0.05)
```

Each sub-score is 0–100.

### 10.2 Sub-Score Definitions

| Field | Meaning |
|---|---|
| `icp_fit` | How closely the business matches target industries |
| `pallet_volume` | Estimated number of pallets generated or needed |
| `buyer_source_fit` | Whether the company likely buys, supplies, or does both |
| `route_fit` | Drive-time and cluster value relative to current service routes |
| `contactability` | Quality of phone/email/contact-form data |
| `urgency` | Signals such as warehouse cleanup, shipping volume, expansion, hiring, reviews mentioning pallets/loading dock |
| `source_confidence` | Reliability of the source data |

### 10.3 Priority Levels

| Score | Priority |
|---:|---|
| 85–100 | A — immediate owner review |
| 70–84 | B — strong lead |
| 50–69 | C — nurture/research |
| 30–49 | D — low priority |
| 0–29 | Reject or archive |

---

## 11. Routing Logic

### 11.1 Service Area

Base rule:

```text
lead is serviceable if drive_time_from_base <= max_drive_minutes
```

Default:

```text
max_drive_minutes = 180
```

This must be configurable in settings.

### 11.2 Route Fit

A lead can be high-value even if far away when:

- estimated pallet volume is high
- lead can become recurring
- lead is near other leads/customers
- lead is on a planned route corridor
- lead is a buyer requiring delivery
- lead has both pickup supply and purchase demand

### 11.3 Route Priority Formula

```text
route_priority =
  estimated_gross_value
  + recurring_account_bonus
  + cluster_density_bonus
  - estimated_travel_cost
  - estimated_service_cost
  - capacity_penalty
```

### 11.4 Suggested Route Types

| Route Type | Purpose |
|---|---|
| Pickup loop | Collect used/broken/excess pallets |
| Sales delivery loop | Deliver refurbished or new pallets |
| Mixed loop | Pickup and delivery on same route |
| Expansion scouting route | Visit several high-score leads in a new corridor |
| Emergency cleanup route | One-time large pallet removal |
| Recurring account route | Scheduled service for established customers |

---

## 12. SendGrid Email Requirements

### 12.1 SendGrid Setup

Required:

- authenticate sending domain
- configure SPF/DKIM through SendGrid domain authentication
- configure DMARC at domain level
- configure branded link tracking if link tracking is used
- create unsubscribe groups through Advanced Suppression Management
- configure Event Webhook
- configure Inbound Parse Webhook if replies are routed through SendGrid
- use categories/custom args for campaign tracking

### 12.2 Required SendGrid Suppression Groups

Create suppression groups:

1. `Pallet Pickup Outreach`
2. `Pallet Supply Sales Outreach`
3. `New Pallet Sales Outreach`
4. `Partner Outreach`
5. `General Company Updates`

All marketing/outreach emails must include:

- unsubscribe link
- physical mailing address
- accurate sender name
- accurate subject line
- company identity
- one clear call to action

### 12.3 SendGrid Event Mapping

Map SendGrid events into `email_events`.

| SendGrid Event | CRM Effect |
|---|---|
| processed | mark message accepted by SendGrid |
| delivered | mark delivered |
| open | increment opens |
| click | increment clicks |
| bounce | mark contact bounced, suppress if hard bounce |
| dropped | investigate/suppress |
| spamreport | mark do-not-contact |
| unsubscribe | mark unsubscribed and suppress |
| group_unsubscribe | suppress for group |
| group_resubscribe | record but require owner confirmation before future campaigns |

### 12.4 Sending Gate

Before sending, the system must verify:

```text
lead.owner_approved = true
contact.owner_approved = true
campaign.owner_approved = true
contact.email_verified_status in ('verified', 'generic_company_email', 'unknown_allowed_by_owner')
contact.unsubscribed = false
contact.do_not_contact = false
contact.suppressed = false
company.do_not_contact = false
email_template.compliance_passed = true
```

If any check fails, block send and write to `audit_logs`.

---

## 13. Compliance & Data Governance

### 13.1 CAN-SPAM Controls

System controls:

- accurate From/Reply-To identity
- non-deceptive subject line
- commercial email identification where required
- physical mailing address in footer
- clear unsubscribe mechanism
- opt-out processing within required timeframe
- suppression list sync with SendGrid
- audit trail for every send
- source provenance for contact data
- owner approval before outreach

### 13.2 TCPA / Calls / Texts

For calls and SMS:

- Treat automated texts/robocalls as restricted.
- Do not send automated marketing texts unless prior express written consent is captured and stored.
- Support manual call tasks, but track do-not-call preferences.
- If SMS is added later, build consent capture and STOP handling before launch.

### 13.3 Data Source Rules

Allowed:

- authorized APIs
- company websites
- public contact pages
- manually entered owner notes
- owner-provided customer/source lists
- licensed lead databases

Not allowed:

- scraping private systems
- bypassing logins
- ignoring robots.txt or terms
- harvesting emails from sources that prohibit it
- using purchased lists without source and opt-out risk review
- guessing personal email addresses without verification and owner approval

### 13.4 Audit Requirements

Every material action must be logged:

- lead discovered
- contact enriched
- score calculated
- owner approved
- owner rejected
- email drafted
- email sent
- email event received
- unsubscribe received
- route generated
- route approved
- record merged
- field manually changed

---

## 14. Supabase Data Model

This is an implementation-ready starting schema. Adjust naming conventions as needed.

```sql
-- Extensions
create extension if not exists "uuid-ossp";
create extension if not exists pgcrypto;

-- Optional if using embeddings
-- create extension if not exists vector;

-- Enums
create type lead_type as enum (
  'PICKUP_SOURCE',
  'BUYER',
  'BOTH',
  'PARTNER'
);

create type company_status as enum (
  'DISCOVERED',
  'ENRICHING',
  'ENRICHED',
  'SCORED',
  'OWNER_REVIEW',
  'APPROVED',
  'CONTACTED',
  'RESPONDED',
  'QUALIFIED',
  'OPPORTUNITY',
  'CUSTOMER',
  'RECURRING_ACCOUNT',
  'INACTIVE',
  'DO_NOT_CONTACT'
);

create type contact_status as enum (
  'DISCOVERED',
  'VERIFIED',
  'OWNER_APPROVED',
  'CONTACTED',
  'REPLIED',
  'MEETING_SET',
  'NOT_INTERESTED',
  'BAD_CONTACT',
  'UNSUBSCRIBED',
  'DO_NOT_CONTACT'
);

create type opportunity_type as enum (
  'PALLET_PICKUP',
  'USED_PALLET_SUPPLY',
  'REFURBISHED_PALLET_SALE',
  'NEW_PALLET_SALE',
  'CUSTOM_PALLET_BUILD',
  'RECURRING_PICKUP',
  'RECURRING_DELIVERY',
  'PARTNERSHIP'
);

create type task_status as enum (
  'OPEN',
  'IN_PROGRESS',
  'DONE',
  'CANCELLED'
);

create type task_type as enum (
  'CALL',
  'EMAIL',
  'OWNER_REVIEW',
  'ROUTE_REVIEW',
  'FOLLOW_UP',
  'QUOTE',
  'SITE_VISIT',
  'DATA_RESEARCH'
);

-- Tenant/company account
create table tenants (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  base_address text,
  base_lat numeric,
  base_lng numeric,
  max_drive_minutes integer not null default 180,
  physical_mailing_address text,
  created_at timestamptz not null default now()
);

-- User profile
create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  tenant_id uuid not null references tenants(id) on delete cascade,
  full_name text,
  role text not null check (role in ('owner', 'admin', 'sales', 'ops', 'viewer')),
  created_at timestamptz not null default now()
);

-- Companies
create table companies (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  name text not null,
  normalized_name text,
  lead_type lead_type,
  status company_status not null default 'DISCOVERED',
  industry text,
  website text,
  phone text,
  email text,
  contact_form_url text,
  address_line1 text,
  address_line2 text,
  city text,
  state text,
  postal_code text,
  country text default 'US',
  lat numeric,
  lng numeric,
  google_place_id text,
  drive_minutes_from_base integer,
  drive_miles_from_base numeric,
  estimated_pallets_month integer,
  estimated_pallet_condition text,
  estimated_pallet_demand text,
  source_confidence numeric check (source_confidence >= 0 and source_confidence <= 1),
  owner_approved boolean not null default false,
  do_not_contact boolean not null default false,
  duplicate_of uuid references companies(id),
  notes text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index companies_tenant_idx on companies(tenant_id);
create index companies_status_idx on companies(status);
create index companies_lead_type_idx on companies(lead_type);
create index companies_geo_idx on companies(lat, lng);

-- Company source provenance
create table company_sources (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid not null references companies(id) on delete cascade,
  source_name text not null,
  source_url text,
  source_type text,
  raw_payload jsonb,
  confidence numeric check (confidence >= 0 and confidence <= 1),
  created_at timestamptz not null default now()
);

-- Contacts
create table contacts (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid not null references companies(id) on delete cascade,
  first_name text,
  last_name text,
  full_name text,
  title text,
  department text,
  email text,
  phone text,
  linkedin_url text,
  is_generic boolean not null default false,
  status contact_status not null default 'DISCOVERED',
  email_verified_status text check (email_verified_status in ('verified', 'invalid', 'catch_all', 'unknown', 'generic_company_email')),
  owner_approved boolean not null default false,
  unsubscribed boolean not null default false,
  suppressed boolean not null default false,
  do_not_contact boolean not null default false,
  source_name text,
  source_url text,
  confidence numeric check (confidence >= 0 and confidence <= 1),
  last_contacted_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create index contacts_tenant_idx on contacts(tenant_id);
create index contacts_company_idx on contacts(company_id);
create index contacts_email_idx on contacts(lower(email));

-- Lead scores
create table lead_scores (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid not null references companies(id) on delete cascade,
  score_total integer not null check (score_total >= 0 and score_total <= 100),
  icp_fit integer check (icp_fit >= 0 and icp_fit <= 100),
  pallet_volume integer check (pallet_volume >= 0 and pallet_volume <= 100),
  buyer_source_fit integer check (buyer_source_fit >= 0 and buyer_source_fit <= 100),
  route_fit integer check (route_fit >= 0 and route_fit <= 100),
  contactability integer check (contactability >= 0 and contactability <= 100),
  urgency integer check (urgency >= 0 and urgency <= 100),
  source_confidence_score integer check (source_confidence_score >= 0 and source_confidence_score <= 100),
  recommended_next_action text,
  reasoning_summary text,
  model_version text,
  created_at timestamptz not null default now()
);

-- Owner reviews
create table owner_reviews (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid references companies(id) on delete cascade,
  contact_id uuid references contacts(id) on delete set null,
  reviewer_id uuid references profiles(id),
  decision text not null check (decision in ('approved', 'rejected', 'needs_more_research', 'call_only', 'do_not_contact')),
  notes text,
  created_at timestamptz not null default now()
);

-- Opportunities
create table opportunities (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid not null references companies(id) on delete cascade,
  contact_id uuid references contacts(id) on delete set null,
  opportunity_type opportunity_type not null,
  stage text not null default 'new',
  estimated_value numeric,
  estimated_pallets integer,
  expected_close_date date,
  recurring boolean not null default false,
  win_probability numeric,
  notes text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

-- Campaigns
create table campaigns (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  name text not null,
  campaign_type text not null,
  sendgrid_asm_group_id text,
  status text not null default 'draft' check (status in ('draft', 'owner_review', 'approved', 'sending', 'paused', 'completed', 'archived')),
  owner_approved boolean not null default false,
  approved_by uuid references profiles(id),
  approved_at timestamptz,
  created_at timestamptz not null default now()
);

-- Email templates
create table email_templates (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  campaign_id uuid references campaigns(id) on delete cascade,
  name text not null,
  subject text not null,
  body_text text not null,
  body_html text,
  compliance_passed boolean not null default false,
  created_at timestamptz not null default now()
);

-- Email messages
create table email_messages (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  campaign_id uuid references campaigns(id) on delete set null,
  company_id uuid references companies(id) on delete set null,
  contact_id uuid references contacts(id) on delete set null,
  template_id uuid references email_templates(id) on delete set null,
  sendgrid_message_id text,
  to_email text not null,
  from_email text not null,
  subject text not null,
  status text not null default 'queued',
  blocked_reason text,
  sent_at timestamptz,
  delivered_at timestamptz,
  opened_at timestamptz,
  clicked_at timestamptz,
  bounced_at timestamptz,
  unsubscribed_at timestamptz,
  created_at timestamptz not null default now()
);

-- Email events from SendGrid
create table email_events (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  email_message_id uuid references email_messages(id) on delete cascade,
  sendgrid_message_id text,
  event_type text not null,
  event_payload jsonb,
  event_at timestamptz,
  created_at timestamptz not null default now()
);

-- Suppressions
create table suppressions (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  email text not null,
  suppression_type text not null check (suppression_type in ('unsubscribe', 'bounce', 'spamreport', 'manual', 'global', 'group')),
  sendgrid_group_id text,
  reason text,
  source text,
  created_at timestamptz not null default now(),
  unique(tenant_id, email, suppression_type)
);

-- Tasks
create table tasks (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid references companies(id) on delete cascade,
  contact_id uuid references contacts(id) on delete set null,
  opportunity_id uuid references opportunities(id) on delete cascade,
  assigned_to uuid references profiles(id),
  task_type task_type not null,
  status task_status not null default 'OPEN',
  title text not null,
  description text,
  due_at timestamptz,
  completed_at timestamptz,
  created_at timestamptz not null default now()
);

-- Notes
create table notes (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  company_id uuid references companies(id) on delete cascade,
  contact_id uuid references contacts(id) on delete set null,
  opportunity_id uuid references opportunities(id) on delete cascade,
  author_id uuid references profiles(id),
  body text not null,
  created_at timestamptz not null default now()
);

-- Route plans
create table route_plans (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  name text not null,
  route_type text not null check (route_type in ('pickup', 'delivery', 'mixed', 'scouting', 'recurring')),
  status text not null default 'draft' check (status in ('draft', 'owner_review', 'approved', 'completed', 'cancelled')),
  scheduled_date date,
  total_drive_minutes integer,
  total_service_minutes integer,
  total_miles numeric,
  total_estimated_pallets integer,
  capacity_warning boolean default false,
  profitability_summary text,
  created_at timestamptz not null default now()
);

-- Route stops
create table route_stops (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  route_plan_id uuid not null references route_plans(id) on delete cascade,
  company_id uuid not null references companies(id) on delete cascade,
  sequence integer not null,
  stop_type text not null check (stop_type in ('pickup', 'delivery', 'sales_visit', 'scouting')),
  estimated_arrival timestamptz,
  estimated_service_minutes integer,
  estimated_pallets integer,
  notes text,
  created_at timestamptz not null default now()
);

-- Agent runs
create table agent_runs (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  agent_name text not null,
  status text not null check (status in ('queued', 'running', 'succeeded', 'failed', 'cancelled')),
  input jsonb,
  output jsonb,
  error text,
  started_at timestamptz,
  finished_at timestamptz,
  created_at timestamptz not null default now()
);

-- Audit logs
create table audit_logs (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references tenants(id) on delete cascade,
  actor_user_id uuid references profiles(id),
  actor_type text not null check (actor_type in ('user', 'agent', 'system')),
  action text not null,
  entity_type text not null,
  entity_id uuid,
  before jsonb,
  after jsonb,
  metadata jsonb,
  created_at timestamptz not null default now()
);
```

---

## 15. Supabase RLS Requirements

Enable RLS on all tenant-owned tables.

```sql
alter table companies enable row level security;
alter table contacts enable row level security;
alter table campaigns enable row level security;
alter table email_messages enable row level security;
alter table tasks enable row level security;
alter table opportunities enable row level security;
alter table route_plans enable row level security;
alter table route_stops enable row level security;
alter table audit_logs enable row level security;
```

Example tenant isolation policy:

```sql
create policy "tenant members can read companies"
on companies for select
using (
  tenant_id in (
    select tenant_id from profiles where id = auth.uid()
  )
);

create policy "admins and owners can update companies"
on companies for update
using (
  tenant_id in (
    select tenant_id from profiles
    where id = auth.uid()
    and role in ('owner', 'admin', 'sales', 'ops')
  )
);
```

Recommended roles:

| Role | Capabilities |
|---|---|
| owner | full access, approve campaigns/routes/leads |
| admin | manage users/settings, approve if allowed |
| sales | manage CRM, propose campaigns, contact follow-ups |
| ops | manage routes, pickups, pallet quantities |
| viewer | read-only dashboard |

---

## 16. API / Edge Function Requirements

### 16.1 Lead Search

`POST /functions/v1/search-leads`

Input:

```json
{
  "territory_id": "uuid",
  "categories": ["warehouse", "manufacturer", "distribution_center"],
  "max_results": 100
}
```

Output:

```json
{
  "agent_run_id": "uuid",
  "companies_created": 20,
  "duplicates_found": 8,
  "needs_review": 12
}
```

### 16.2 Enrich Company

`POST /functions/v1/enrich-company`

Input:

```json
{
  "company_id": "uuid"
}
```

### 16.3 Score Company

`POST /functions/v1/score-company`

Input:

```json
{
  "company_id": "uuid"
}
```

### 16.4 Owner Approval

`POST /functions/v1/owner-review`

Input:

```json
{
  "company_id": "uuid",
  "contact_id": "uuid",
  "decision": "approved",
  "notes": "Good pickup source. Start with email."
}
```

### 16.5 Draft Outreach

`POST /functions/v1/draft-outreach`

Input:

```json
{
  "company_id": "uuid",
  "contact_id": "uuid",
  "campaign_id": "uuid"
}
```

### 16.6 Send Approved Email

`POST /functions/v1/send-approved-email`

Input:

```json
{
  "email_message_id": "uuid"
}
```

This endpoint must run all compliance and suppression checks before calling SendGrid.

### 16.7 SendGrid Event Webhook

`POST /functions/v1/sendgrid-events`

Responsibilities:

- verify webhook signature if configured
- map SendGrid message ID to `email_messages`
- insert `email_events`
- update message/contact/campaign status
- create tasks for replies or important engagement
- write suppressions for unsubscribes, spam reports, and hard bounces

### 16.8 Route Generation

`POST /functions/v1/generate-route`

Input:

```json
{
  "route_type": "pickup",
  "candidate_company_ids": ["uuid"],
  "truck_capacity": 500,
  "scheduled_date": "2026-06-01"
}
```

---

## 17. UI Pages

### 17.1 `/dashboard`

Owner KPI homepage.

Widgets:

- lead totals
- review queue
- email performance
- route opportunities
- pipeline value
- top industries
- tasks due today
- recent replies

### 17.2 `/leads/search`

Lead discovery interface.

Inputs:

- territory
- industry
- lead type
- radius/drive-time
- keywords
- max results
- data source

Actions:

- run search
- preview results
- import selected
- enrich selected
- score selected

### 17.3 `/leads/review`

Owner approval queue.

Actions:

- approve
- reject
- approve call only
- request more research
- mark duplicate
- do not contact

### 17.4 `/companies/[id]`

Company intelligence profile.

Sections:

- company summary
- lead score
- contacts
- source history
- CRM timeline
- email history
- opportunities
- route notes
- owner notes
- audit log

### 17.5 `/crm`

Kanban CRM.

Features:

- drag/drop stage updates
- filters by owner, status, lead type, city, score
- task creation
- notes
- opportunity creation

### 17.6 `/campaigns`

Campaign management.

Features:

- draft templates
- owner approval
- audience filters
- suppression preview
- SendGrid status
- event analytics

### 17.7 `/routes`

Routing and map interface.

Features:

- candidate stops
- route builder
- route score
- map preview
- capacity warning
- owner approval
- printable driver sheet

### 17.8 `/settings`

Settings:

- company profile
- base address
- physical mailing address
- max drive minutes
- SendGrid API key status
- SendGrid ASM group mapping
- domain authentication checklist
- user roles
- lead scoring weights
- service offerings
- email footer
- compliance settings

---

## 18. Email Templates

### 18.1 Pickup Source Email

Subject options:

```text
Quick question about pallets at {{company_name}}
Pallet pickup around {{city}}
Do you have used or broken pallets to clear out?
```

Body:

```text
Hi {{first_name_or_team}},

I’m reaching out from {{pallet_company_name}} here in the Atlanta area.

We help local warehouses, distributors, and businesses clear out used, broken, or excess pallets. Depending on the condition and quantity, we can discuss pickup options and recurring service.

Is pallet removal or pallet cleanup something your team handles at {{company_name}}?

Best,
{{sender_name}}
{{sender_title}}
{{pallet_company_name}}
{{phone}}
{{physical_address}}

You can unsubscribe from future emails here: {{unsubscribe_link}}
```

### 18.2 Buyer Email

Subject options:

```text
Pallet supply for {{company_name}}
Used and new pallets near {{city}}
Quick pallet supply question
```

Body:

```text
Hi {{first_name_or_team}},

I’m with {{pallet_company_name}}, an Atlanta-area pallet supplier.

We provide refurbished pallets, new pallets, and custom pallet builds for local businesses. I wanted to ask whether {{company_name}} currently buys pallets or has periodic pallet needs for shipping, warehousing, or distribution.

Would it be worth sending over basic pricing and availability?

Best,
{{sender_name}}
{{sender_title}}
{{pallet_company_name}}
{{phone}}
{{physical_address}}

You can unsubscribe from future emails here: {{unsubscribe_link}}
```

### 18.3 Partner Email

Subject options:

```text
Potential pallet pickup partnership
Partnering on pallet cleanouts
Pallet recycling / pickup referral question
```

Body:

```text
Hi {{first_name_or_team}},

I’m reaching out from {{pallet_company_name}} in the Atlanta area.

We work with businesses that need pallet pickup, pallet recycling, refurbished pallets, and new pallet supply. I thought there may be a partnership fit if your team comes across customers with excess or broken pallets.

Would you be open to a quick conversation?

Best,
{{sender_name}}
{{sender_title}}
{{pallet_company_name}}
{{phone}}
{{physical_address}}

You can unsubscribe from future emails here: {{unsubscribe_link}}
```

---

## 19. MVP Build Phases

### Phase 0 — Project Setup

Deliverables:

- Supabase project
- Next.js app
- auth and roles
- base tenant settings
- SendGrid account setup
- domain authentication checklist
- environment variables
- initial RLS policies

Acceptance criteria:

- owner can log in
- owner can update base address and service radius
- SendGrid API key stored securely
- tenant data is isolated

### Phase 1 — CRM Foundation

Deliverables:

- companies table
- contacts table
- tasks
- notes
- opportunities
- CRM kanban
- company profile page
- owner review status

Acceptance criteria:

- owner can manually add companies and contacts
- owner can move lead through lifecycle
- owner can create tasks and notes
- audit log captures changes

### Phase 2 — Lead Discovery and Enrichment

Deliverables:

- territory configuration
- Google Places / approved source integration
- lead import
- dedupe
- enrichment agent
- source provenance

Acceptance criteria:

- system can discover target companies in selected territory
- lead source is recorded
- duplicate candidates are flagged
- owner can approve/reject imported leads

### Phase 3 — Lead Scoring

Deliverables:

- lead scoring agent
- scoring dashboard
- route drive-time scoring
- priority queue

Acceptance criteria:

- each enriched company gets a score
- score breakdown is visible
- route fit is calculated using drive time
- owners can override score or priority

### Phase 4 — SendGrid Outreach

Deliverables:

- campaign builder
- email templates
- owner approval
- compliance gate
- SendGrid send function
- SendGrid event webhook
- suppressions table
- unsubscribe handling

Acceptance criteria:

- email cannot send without approval
- unsubscribe link is included
- SendGrid events update CRM
- unsubscribed contacts cannot be emailed again
- bounced contacts are suppressed

### Phase 5 — Routing

Deliverables:

- map view
- drive-time service zones
- route generation
- route approval
- route stop list
- route performance reporting

Acceptance criteria:

- system creates route suggestions from approved stops
- route respects truck capacity and drive-time limits
- owner can approve or edit route
- completed routes update company notes and opportunities

### Phase 6 — Agent Automation and Reporting

Deliverables:

- scheduled lead searches
- scheduled enrichment
- weekly owner report
- CRM follow-up automation
- reply classification
- opportunity reporting

Acceptance criteria:

- owner receives weekly lead report
- tasks are auto-created after replies or no-reply windows
- all agent actions are auditable
- dashboard reflects current pipeline

---

## 20. Environment Variables

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# SendGrid
SENDGRID_API_KEY=
SENDGRID_FROM_EMAIL=
SENDGRID_FROM_NAME=
SENDGRID_WEBHOOK_PUBLIC_KEY=
SENDGRID_WEBHOOK_VERIFICATION_KEY=
SENDGRID_PICKUP_ASM_GROUP_ID=
SENDGRID_SALES_ASM_GROUP_ID=
SENDGRID_PARTNER_ASM_GROUP_ID=

# Maps / Routing
GOOGLE_MAPS_API_KEY=
GOOGLE_PLACES_API_KEY=
GOOGLE_ROUTES_API_KEY=

# App
APP_BASE_URL=
TENANT_DEFAULT_BASE_ADDRESS=
MAX_DRIVE_MINUTES=180

# AI / Agents
AI_PROVIDER_API_KEY=
AGENT_MODEL=
```

---

## 21. Owner Approval Workflow

### 21.1 Lead Approval

```text
Lead discovered
→ Enriched
→ Scored
→ Compliance checked
→ Added to Owner Review Queue
→ Owner approves/rejects
→ If approved, contact can enter campaign audience
```

### 21.2 Email Approval

```text
Campaign drafted
→ Audience selected
→ Suppression preview generated
→ Compliance Agent checks
→ Owner approves campaign
→ Email messages are queued
→ SendGrid send function sends only passing messages
```

### 21.3 Route Approval

```text
Pickup/sales opportunities selected
→ Routing Agent creates draft route
→ Owner reviews route
→ Owner approves
→ Route stop tasks created
→ Route completed
→ CRM records updated
```

---

## 22. Reports and Exports

### 22.1 Owner Weekly Report

Include:

- new leads discovered
- top 20 leads by score
- leads needing review
- contacts missing emails
- approved but not contacted
- email performance
- replies requiring action
- pickup opportunities
- sales opportunities
- suggested routes
- do-not-contact updates
- recommended next steps

### 22.2 CSV Exports

Export options:

- owner-approved contact list
- pickup source list
- buyer list
- partner list
- campaign audience
- route stops
- opportunities
- follow-up tasks

Each export must include:

- company name
- lead type
- address
- city/state
- phone
- email/contact form
- website
- score
- source
- owner approval status
- last contact date
- CRM stage
- do-not-contact status

---

## 23. Quality, Testing, and Acceptance Criteria

### 23.1 Unit Tests

Test:

- scoring formula
- suppression checks
- owner approval gates
- SendGrid event mapping
- lifecycle transitions
- route capacity calculations
- duplicate detection
- tenant isolation

### 23.2 Integration Tests

Test:

- lead search to import
- import to enrichment
- enrichment to scoring
- scoring to owner review
- approval to campaign
- campaign to SendGrid
- SendGrid webhook to CRM update
- route generation to route approval

### 23.3 Compliance Tests

Test:

- cannot email without owner approval
- cannot email unsubscribed contact
- cannot email suppressed contact
- cannot send template missing physical address
- cannot send template missing unsubscribe link
- bounce creates suppression
- spam report creates do-not-contact
- unsubscribe creates suppression
- manual do-not-contact blocks all outreach

### 23.4 Dashboard Acceptance

The owner should be able to answer these questions in under 30 seconds:

- How many leads need my review?
- Which leads are most valuable?
- Who should we contact today?
- Which companies may have pallets to pick up?
- Which companies may buy pallets?
- Which routes are worth running?
- Which emails received replies?
- Which contacts are suppressed?
- What is the value of the current pipeline?

---

## 24. Recommended Industry Standards Applied

The platform should follow these B2B lead generation standards:

1. Define target customer profiles before searching.
2. Segment leads by use case and buyer/source type.
3. Use multiple data sources and store provenance.
4. Score leads before outreach.
5. Deduplicate companies and contacts.
6. Verify or confidence-score contact information.
7. Route all leads through CRM stages.
8. Require human approval before outbound outreach.
9. Use domain authentication and sender reputation controls.
10. Include unsubscribe and suppression management.
11. Track email events and replies.
12. Convert engagement into follow-up tasks.
13. Measure source quality and pipeline conversion.
14. Continuously refine scoring based on won/lost outcomes.
15. Use routing economics, not just lead count, for field operations.

---

## 25. Build Priorities

### Must Have

- Supabase auth/data/RLS
- company/contact CRM
- lead search/import
- enrichment
- scoring
- owner approval queue
- SendGrid approved sending
- SendGrid webhook events
- unsubscribe/suppression handling
- dashboard
- routing drive-time fields
- map view
- audit logs

### Should Have

- route optimization
- reply classification
- weekly reports
- CSV exports
- AI-generated outreach drafts
- automated follow-up tasks
- duplicate merge workflow

### Could Have

- inventory matching
- quote generation
- customer portal
- SMS with explicit consent
- truck driver mobile view
- automated pricing recommendations
- predictive route planning
- third-party CRM sync

---

## 26. Definition of Done

The build is done when:

- owners can log in securely
- owners can search for leads around Atlanta and within the 3-hour drive-time expansion zone
- companies and contacts are stored in Supabase
- leads are enriched, deduped, scored, and presented for review
- owners can approve/reject leads
- approved leads can receive compliant SendGrid emails
- SendGrid events flow back into the CRM
- unsubscribes/bounces/spam reports block future sends
- owners can manage follow-up tasks
- owners can view a dashboard of pipeline and lead activity
- owners can generate route suggestions based on demand and drive time
- all agent actions and owner decisions are auditable

---

## 27. Implementation Notes for Developers

### Recommended First Sprint

1. Create Supabase project.
2. Implement schema and RLS.
3. Create Next.js shell with auth.
4. Build company/contact CRUD.
5. Build owner review queue.
6. Add manual CSV import.
7. Add lead scoring function.
8. Add dashboard basics.

### Recommended Second Sprint

1. Add Places/API lead discovery.
2. Add enrichment workflow.
3. Add dedupe.
4. Add route drive-time lookup.
5. Add map page.
6. Add campaign builder.
7. Add SendGrid suppression groups.
8. Add SendGrid email send function.

### Recommended Third Sprint

1. Add SendGrid webhooks.
2. Add CRM automations.
3. Add route generation.
4. Add reports.
5. Add agent run logs.
6. Add reply classification.
7. Harden compliance tests.
8. Run owner acceptance testing.

---

## 28. Reference Checklist

### Business Setup Needed From Owner

- legal business name
- preferred sending domain
- physical mailing address
- base pickup/delivery address
- phone number
- service offerings
- pallet types and sizes
- truck capacity
- pickup pricing rules
- sales pricing rules
- preferred service days
- existing customers/sources CSV, if available
- do-not-contact list, if available
- approved email signature
- approved claims and wording

### Technical Setup Needed

- Supabase project credentials
- SendGrid account
- authenticated sending domain
- SendGrid API key
- SendGrid ASM group IDs
- maps/routing API key
- app domain
- privacy policy page
- unsubscribe landing page
- webhook endpoints

---

## 29. Future Enhancements

- AI-powered quote builder
- pallet inventory forecast
- recurring pickup schedule optimization
- customer portal for pickup requests
- automated lead source quality scoring
- win/loss learning loop
- geofenced expansion planning
- driver mobile route app
- invoice/payment integration
- QuickBooks integration
- inbound website form that creates CRM leads
- referral partner portal
- pallet condition photo upload and AI classification
