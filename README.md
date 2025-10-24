# AI-Powered Nonprofit Fundraising Automation

Quick demo and how we got here: [https://youtu.be/Gt6_sMTtrDw](https://youtu.be/Gt6_sMTtrDw)

## Overview

This repository contains n8n workflow automation for nonprofit fundraising using AI-driven strategy discovery, A/B testing, and multi-channel outreach. Built for the **LongBio Nonprofit Fundraising Agent** challenge at HackAging.ai 2025.

![Avenue Discovery & AB Message Generation](https://github.com/asverlov/longbio-agent/blob/main/longbio.jpg)

## System Architecture

The system consists of 3 core workflows that work together to automate fundraising campaign generation, lead sourcing, and A/B tested outreach.

### Data Model

All workflows operate on a **single Google Sheet per nonprofit** with dynamic tabs:
- **Brief tab**: Input (mission, projects, location)
- **Avenue tabs**: One per fundraising strategy (e.g., `parents-email`, `foundations-grants`)
- **Leads tabs**: Contact lists per avenue (e.g., `parents-email-Leads`)

---

## WF-1: Avenue Discovery & AB Message Generation

**Purpose:** Autonomously discover optimal fundraising strategies and generate A/B tested messaging.

**Input:**
- Google Sheet ID
- Brief tab with: `mission`, `projects`, `location`

**Process:**
1. AI analyzes the nonprofit's mission and autonomously identifies 2-4 fundraising "avenues"
2. Each avenue defines:
   - Target donor segment (WHO)
   - Outreach channel (HOW)
   - Value proposition (WHY they care)
   - Search hints for lead generation
   - Two message angles for A/B testing
3. Creates a tab for each avenue with metadata
4. Generates two email variants (A/B) per avenue using different psychological appeals
5. Creates a Leads tab for each avenue with tracking columns

**Output:**
- Avenue tabs with A/B messages ready for review/editing
- Empty Leads tabs ready for WF-2
- Summary with next steps

**Key Features:**
- AI discovers audiences—no predefined donor segments required
- Preserves manual edits on re-runs
- Validates all AI outputs before creating tabs

---

## WF-2: Lead Generation per Avenue

**Purpose:** Find and populate donor contacts for each fundraising avenue.

**Input:**
- Google Sheet ID
- Avenue slug (e.g., `parents-email`)

**Process:**
1. Reads the avenue's `searchHints` field
2. Uses AI/web scraping/APIs to find potential donor contacts
3. Normalizes contact data (name, email, org, role, city, state)
4. Deduplicates by email across all avenues
5. Appends leads to the `{avenue}-Leads` tab with `status: ready`

**Output:**
- Populated Leads tab with contacts ready to send
- Deduplicated across all campaigns
- Status tracking for each lead

**Key Features:**
- Repeatable—run multiple times to add more leads
- Never overwrites already-sent leads
- Cross-avenue deduplication prevents spam

---

## WF-3: AB Send (Campaign Execution)

**Purpose:** Execute A/B tested email campaigns with deterministic variant assignment.

**Input:**
- Google Sheet ID
- Avenue slug (e.g., `parents-email`)
- Run tag (unique campaign ID, e.g., `2025-10-22-v1`)
- Optional: Batch size

**Process:**
1. Reads A/B message variants from avenue tab
2. Pulls eligible leads (`status: ready`, not sent in this run tag)
3. **Assigns A/B variant deterministically** using hash(email + run tag) for 50/50 split
4. Sends emails via Gmail with personalization
5. Updates each lead row with:
   - `status: sent`
   - `variant: A` or `B`
   - `runTag`, `sentAt`, `messageId` (for tracking)

**Output:**
- Emails sent to donors
- Complete send log in Leads tab
- Ready for engagement/donation tracking

**Key Features:**
- **Idempotent**: Same run tag won't resend to same leads
- **Deterministic A/B split**: Same lead always gets same variant per campaign
- **Trackable**: Gmail message IDs enable reply/engagement tracking
- Manual trigger = human approval gate

## Installation

1. Import `wf1-avenue-discovery.json` into n8n
2. Configure credentials:
   - Google Sheets OAuth2
   - OpenAI API
3. Create a Google Sheet with a "Brief" tab containing your nonprofit's mission, projects, and location
4. Run WF-1 with the spreadsheet ID

---

## Use Case Example: "Living is Smart" Nonprofit

**Input:**
- Mission: "Promote longevity science and anti-aging education"
- Projects: "YouTube/TikTok content, high school longevity clubs, aging science exhibition"
- Location: "Boston, MA"

**WF-1 Output (AI-discovered avenues):**
1. `parents-email`: Target parents of high school students, emphasize youth health education
2. `schools-grants`: Target school administrators, focus on volunteer programs and curriculum
3. `localgovt-inperson`: Target city officials, highlight community health impact
4. `foundations-grants`: Target private foundations, data-driven ROI on aging research

Each avenue gets custom A/B messages, lead generation strategy, and tracking infrastructure.

---

## Roadmap

- [x] WF-1: Avenue Discovery & AB Message Generation 
- [ ] WF-2: Lead Generation per Avenue
- [ ] WF-3: AB Send (Campaign Execution)
- [ ] WF-4: Engagement & Donation Tracking
- [ ] Analytics Dashboard

---

## License & Credits

Built for the **HackAging.ai 2025 Rapid Adoption Track** challenge.

**Author:** Alex Sverlov (@Living is Smart Inc 501c3)

Designed for nonprofit fundraising automation with AI-driven strategy and continuous optimization.


