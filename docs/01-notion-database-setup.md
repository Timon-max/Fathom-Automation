# Notion Database Setup Guide

**Complete database property configurations for all three databases**

---

## Overview

This system uses three interconnected Notion databases:

1. **Call Intelligence Hub** - Central repository of all call recordings with AI analysis
2. **Content Ideas Bank** - Library of content ideas extracted from calls
3. **Client Intelligence** - Strategic client health tracking and relationship management

---

## Database 1: Call Intelligence Hub

**Purpose:** Master record of every call with transcript, summary, and processing metadata

### Properties Configuration

| Property Name | Type | Configuration | Notes |
|--------------|------|---------------|-------|
| **Call Title** | Title | Default title property | Auto-filled from Fathom recording title |
| **Call Date** | Date | Include time: Yes | When the recording was created |
| **Recording URL** | URL | - | Direct link to Fathom recording |
| **Duration** | Number | Format: Number<br>Show as: Minutes | Call length in minutes |
| **Raw Transcript** | Text | Long text | Full transcript from Fathom |
| **Fathom Summary** | Text | Long text | Fathom's AI-generated summary |
| **Executive Summary** | Text | Long text | Your AI-generated executive summary (Markdown formatted) |
| **Processing Status** | Select | Options:<br>- 🟡 Queued<br>- 🔵 Processing<br>- 🟢 Complete<br>- 🔴 Failed | Tracks automation workflow progress |
| **Sentiment Score** | Select | Options:<br>- 🟢 Improving<br>- 🔵 Stable<br>- 🟡 Watch<br>- 🔴 At Risk | Client health indicator |
| **Content Ideas Generated** | Rollup | Relation: Content Ideas Bank → Source Call<br>Property: (Count all)<br>Calculate: Count values | Auto-counts linked content ideas |
| **Client Name** | Text | Short text | For filtering and grouping (optional) |
| **Created** | Created time | Auto-populated | System field |

### Sample Entry

```
Call Title: Q1 Strategy Session - Sarah Johnson
Call Date: January 11, 2024 2:00 PM
Recording URL: https://app.fathom.video/recording/...
Duration: 42
Processing Status: 🟢 Complete
Sentiment Score: 🟢 Improving
Content Ideas Generated: 8
```

---

## Database 2: Content Ideas Bank

**Purpose:** Searchable library of all content ideas with drafts and publication tracking

### Properties Configuration

| Property Name | Type | Configuration | Notes |
|--------------|------|---------------|-------|
| **Idea Title** | Title | Default title property | Attention-grabbing content title |
| **Content Type** | Select | Options:<br>- 📝 LinkedIn Post<br>- 📧 Newsletter<br>- 🎨 Carousel<br>- 🧵 Thread<br>- 🎥 Video Script | Content format |
| **Source Call** | Relation | Related to: Call Intelligence Hub<br>Allow multiple: No | Links back to originating call |
| **Client Name** | Rollup | Relation: Source Call<br>Property: Client Name<br>Calculate: Show original | Auto-pulls from call record |
| **Raw Idea** | Text | Long text | Core concept and why it resonates (AI output) |
| **Polished Draft** | Text | Long text | Complete, ready-to-edit content (only for priority 8+) |
| **Priority Score** | Number | Format: Number<br>Range: 1-10 | AI-assigned virality/relevance score |
| **Hook/Opening** | Text | Short text | First line / opening hook |
| **Key Points** | Text | Long text | Bullet points to cover |
| **Platform Fit** | Multi-select | Options:<br>- LinkedIn<br>- Email<br>- Twitter<br>- Instagram<br>- Facebook<br>- Blog | Best platforms for this content |
| **Status** | Select | Options:<br>- 🆕 New Idea<br>- ✍️ Ready to Draft<br>- 📝 Drafted<br>- 📅 Scheduled<br>- ✅ Published | Content workflow stage |
| **Published URL** | URL | - | Link once published (manual) |
| **Created** | Created time | Auto-populated | When idea was generated |

### Formula Properties (Optional)

**Priority Indicator**
```
Property: Priority Indicator (Formula)
Formula: if(prop("Priority Score") >= 8, "⭐ High", if(prop("Priority Score") >= 6, "🔶 Medium", "🔵 Low"))
```

**Days Since Created**
```
Property: Age (Formula)
Formula: dateBetween(now(), prop("Created"), "days")
```

### Sample Entry

```
Idea Title: Why Most Coaches Fail at Time Management
Content Type: 📝 LinkedIn Post
Source Call: → [Q1 Strategy Session - Sarah Johnson]
Client Name: Sarah Johnson (rollup)
Priority Score: 9
Hook/Opening: I asked 47 coaches about their biggest challenge...
Platform Fit: LinkedIn, Newsletter
Status: ✍️ Ready to Draft
Created: January 11, 2024
```

---

## Database 3: Client Intelligence

**Purpose:** Strategic client health monitoring and relationship tracking

### Properties Configuration

| Property Name | Type | Configuration | Notes |
|--------------|------|---------------|-------|
| **Client Name** | Title | Default title property | Primary client identifier |
| **Last Call Date** | Date | Include time: No | Most recent call |
| **Total Calls** | Rollup | Relation: Call Intelligence Hub → Client Name<br>Property: Call Title<br>Calculate: Count values | Auto-counts calls (if you add relation) |
| **Current Sentiment** | Select | Options:<br>- 🟢 Improving<br>- 🔵 Stable<br>- 🟡 Watch<br>- 🔴 At Risk | Latest sentiment from most recent call |
| **Pain Points** | Multi-select | Dynamic options<br>(AI adds new ones automatically) | Ongoing challenges and frustrations |
| **Recent Wins** | Text | Long text | Successes and progress (chronological, newest first) |
| **Upsell Opportunities** | Text | Long text | Expansion possibilities and wish-list items |
| **Red Flags** | Text | Long text | Warning signs and concerns |
| **Next Call Date** | Date | Include time: Yes | Scheduled future call (manual entry) |
| **Account Status** | Select | Options:<br>- 🟢 Active<br>- 🟡 Onboarding<br>- 🔵 Stable<br>- 🟠 At Risk<br>- 🔴 Churned | Manual override of status |
| **Notes** | Text | Long text | Free-form strategic notes |
| **Created** | Created time | Auto-populated | When client was added |

### Formula Properties (Optional)

**Relationship Score**
```
Property: Health Score (Formula)
Formula:
if(prop("Current Sentiment") == "🟢 Improving", 100,
  if(prop("Current Sentiment") == "🔵 Stable", 75,
    if(prop("Current Sentiment") == "🟡 Watch", 50, 25)))
```

**Days Since Last Call**
```
Property: Days Since Contact (Formula)
Formula: dateBetween(now(), prop("Last Call Date"), "days")
```

**Needs Attention**
```
Property: Alert (Formula)
Formula:
if(prop("Days Since Contact") > 30, "⚠️ No contact 30+ days",
  if(contains(prop("Current Sentiment"), "At Risk"), "🚨 At Risk Client",
    if(contains(prop("Current Sentiment"), "Watch"), "👀 Watch Closely", "")))
```

### Sample Entry

```
Client Name: Sarah Johnson
Last Call Date: January 11, 2024
Current Sentiment: 🟢 Improving
Pain Points: Team Onboarding, Notion Navigation, ROI Tracking
Recent Wins:
  - 01/11/2024 Call: Successfully automated client intake process
  - 01/11/2024 Call: Closed 3 new clients using generated content
  - 01/11/2024 Call: Team adoption went from 30% to 85%
Upsell Opportunities:
  - Podcast workflow automation
  - Client-facing dashboard
  - White-label solution for their clients
Red Flags: None detected
Account Status: 🟢 Active
```

---

## Database Relations & Rollups

### Setting Up Relations

**Content Ideas Bank → Call Intelligence Hub**

1. Open Content Ideas Bank database
2. Add property: "Source Call"
3. Type: Relation
4. Related database: Call Intelligence Hub
5. Relation type: Single relation (each idea links to one call)
6. Show on Call Intelligence Hub: Yes
7. Name in related database: "Content Ideas" (auto-created)

**Result:**
- Content ideas can link to their source call
- Call records show all related content ideas
- Enables rollup to count ideas per call

### Setting Up Rollups

**Call Intelligence Hub: Count Content Ideas**

1. Requires: Relation from Content Ideas Bank set up first
2. Add property: "Content Ideas Generated"
3. Type: Rollup
4. Relation: Content Ideas (the reverse relation)
5. Property: Any (e.g., Idea Title)
6. Calculate: Count all

**Content Ideas Bank: Pull Client Name**

1. Requires: Relation to Call Intelligence Hub set up
2. Requires: Client Name property exists in Call Intelligence Hub
3. Add property: "Client Name"
4. Type: Rollup
5. Relation: Source Call
6. Property: Client Name
7. Calculate: Show original

---

## Notion Integration Setup

### Create Integration

1. Go to: https://www.notion.so/my-integrations
2. Click **"+ New integration"**
3. Configure:
   - **Name:** Fathom Automation
   - **Associated workspace:** [Your workspace]
   - **Logo:** (optional)
4. Click **"Submit"**
5. Copy **"Internal Integration Token"** (starts with `secret_`)
   - Save to `/config/.env` as `NOTION_API_TOKEN`

### Share Databases with Integration

**For each database (repeat 3 times):**

1. Open database as full page
2. Click **••• (More)** in top right
3. Click **"Add connections"**
4. Find and select **"Fathom Automation"** integration
5. Click **"Confirm"**

**Verify access:**
- You should see a small badge on database showing integration is connected

### Get Database IDs

**For each database (repeat 3 times):**

1. Open database as full page (not inline)
2. Copy URL from browser:
   ```
   https://www.notion.so/workspace/DATABASE_ID?v=VIEW_ID
   ```
3. Extract `DATABASE_ID` (32 characters, letters/numbers)
4. Save to `/config/.env`:
   ```
   NOTION_DB_CALL_INTELLIGENCE_HUB=DATABASE_ID_HERE
   NOTION_DB_CONTENT_IDEAS_BANK=DATABASE_ID_HERE
   NOTION_DB_CLIENT_INTELLIGENCE=DATABASE_ID_HERE
   ```

---

## Database Templates (Optional)

### Call Intelligence Hub Template

Create a button to manually add calls:

1. Add button in database view
2. Configure:
   - **Label:** "➕ Add Call Manually"
   - **Action:** Add new page
   - **Template:**
     - Processing Status: 🟡 Queued
     - [Pre-fill other fields as needed]

### Content Ideas Bank Template

Create templates for each content type:

**Template: LinkedIn Post**
- Content Type: 📝 LinkedIn Post
- Platform Fit: LinkedIn
- Status: 📝 Drafted

**Template: Newsletter**
- Content Type: 📧 Newsletter
- Platform Fit: Email
- Status: 📝 Drafted

---

## Database Views (Preview)

Basic views to create (detailed in `05-dashboard-views.md`):

### Call Intelligence Hub Views
- **All Calls** (Table): Default view, sorted by Call Date descending
- **Recent Calls** (Gallery): Last 30 days, card preview
- **Failed Processing** (Table): Filter by Status = Failed

### Content Ideas Bank Views
- **Content Pipeline** (Board): Group by Status
- **By Client** (Table): Group by Client Name
- **High Priority** (Table): Filter Priority Score >= 8

### Client Intelligence Views
- **Client Health** (Gallery): Group by Current Sentiment
- **Needs Attention** (Table): Filter by Alert formula
- **All Clients** (Table): Sorted by Last Call Date

---

## Maintenance

### Regular Cleanup

**Monthly:**
- Archive calls older than 90 days (move to separate database)
- Remove unused Pain Points options (multi-select)
- Update Platform Fit options if using new channels

**Quarterly:**
- Review and consolidate duplicate client records
- Update formulas if business logic changes
- Audit relation integrity (broken links)

---

## Backup Strategy

**Notion's native backup:**
- Settings & Members → Settings → Export all workspace content
- Run: Monthly
- Format: Markdown & CSV (preserves structure)

**Database-specific exports:**
1. Open database
2. Click ••• → Export
3. Format: CSV (for data portability)
4. Save to secure location

---

## Advanced: API Access Testing

Test your integration token:

```bash
curl -X POST https://api.notion.com/v1/databases/DATABASE_ID/query \
  -H "Authorization: Bearer YOUR_INTEGRATION_TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json"
```

**Expected response:** JSON with database items

**If error:**
- 401: Invalid token
- 404: Database not found or integration not connected
- 400: Malformed request

---

## Property Naming Best Practices

**Do:**
- Use clear, descriptive names
- Be consistent across databases
- Use Title Case for visual scanning

**Don't:**
- Use special characters (emojis in options only)
- Make names too long (>30 characters)
- Use abbreviations unless universally understood

**Example:**
- ✅ "Processing Status"
- ✅ "Content Ideas Generated"
- ❌ "Proc_Status"
- ❌ "ContentIdeasThatHaveBeenGeneratedFromThisCall"

---

## Common Issues

**Problem: Integration can't access database**
- Solution: Re-share database with integration (it may have been unshared)

**Problem: Rollup shows "0" when it should have data**
- Solution: Check relation is set up correctly and has linked items

**Problem: Multi-select options not populating from Make.com**
- Solution: Ensure exact spelling, or create option in Notion first

**Problem: Formula returns error**
- Solution: Check property names referenced in formula match exactly

---

## Next Steps

Once databases are configured:

1. ✅ Verify all properties created
2. ✅ Verify relations and rollups working
3. ✅ Share databases with integration
4. ✅ Save database IDs to `.env`
5. → Move to: `docs/02-makecom-workflow-guide.md`
