# Make.com Workflow Guide

**Complete step-by-step guide to building the Fathom → Notion automation workflow**

## Overview

This guide walks you through building a Make.com scenario that:
1. Watches for new Fathom recordings (every 15 minutes)
2. Retrieves transcript and summary
3. Creates a record in Notion Call Intelligence Hub
4. Splits into 3 parallel AI processing paths
5. Populates Content Ideas Bank and Client Intelligence databases

**Estimated build time:** 3-4 hours
**Complexity level:** Intermediate

---

## Prerequisites Checklist

Before starting, ensure you have:

- [ ] Make.com account (free tier works)
- [ ] Fathom API credentials
  - [ ] API Key
  - [ ] Workspace ID
- [ ] OpenRouter API key with $5+ credit
- [ ] Notion Integration token
- [ ] All three Notion database IDs:
  - [ ] Call Intelligence Hub database ID
  - [ ] Content Ideas Bank database ID
  - [ ] Client Intelligence database ID

### Getting Your Credentials

**Fathom API:**
1. Go to Fathom Settings → Integrations → API
2. Generate new API key
3. Copy workspace ID from URL: `app.fathom.video/workspace/[WORKSPACE_ID]`

**OpenRouter:**
1. Sign up at openrouter.ai
2. Go to Keys → Create new key
3. Add $5-10 credit in Billing

**Notion:**
1. Go to notion.so/my-integrations
2. Create new integration
3. Copy Internal Integration Token
4. Share each database with your integration:
   - Open database → Click ••• → Add connections → Select your integration

**Notion Database IDs:**
1. Open database as full page
2. Copy URL: `notion.so/[workspace]/[DATABASE_ID]?v=...`
3. The DATABASE_ID is the 32-character string before `?v=`

---

## Scenario Architecture

```
Module Flow:

[1] Fathom: Watch New Recordings (Trigger)
    │
    ├─→ [2] Fathom: Get a Transcript
    │
    └─→ [3] Fathom: Get a Summary
         │
         └─→ [4] Notion: Create Database Item (Call Intelligence Hub)
              │
              └─→ [5] Router (3 paths)
                   │
                   ├─→ PATH A: Content Ideas Generator
                   │    ├─→ [5A] HTTP: OpenRouter API (Generate Ideas)
                   │    ├─→ [6A] JSON: Parse Response
                   │    ├─→ [7A] Iterator: Loop Ideas
                   │    ├─→ [8A] Notion: Create Content Idea
                   │    ├─→ [9A] Filter: Priority >= 8
                   │    ├─→ [10A] HTTP: OpenRouter API (Draft Content)
                   │    └─→ [11A] Notion: Update with Draft
                   │
                   ├─→ PATH B: Client Intelligence Analyzer
                   │    ├─→ [5B] HTTP: OpenRouter API (Analyze Intelligence)
                   │    ├─→ [6B] JSON: Parse Response
                   │    ├─→ [7B] Notion: Search Client Record
                   │    └─→ [8B] Router: Create or Update Client
                   │
                   └─→ PATH C: Executive Summary Generator
                        ├─→ [5C] HTTP: OpenRouter API (Generate Summary)
                        └─→ [6C] Notion: Update Call Record
```

---

## Part 1: Core Workflow Setup

### Module [1]: Fathom Watch New Recordings (Trigger)

**Purpose:** Automatically detects new call recordings

**Setup:**

1. In Make.com, create new scenario
2. Click the + button → Search "Fathom"
3. Select **"Watch New Recordings"**
4. Click **"Create a connection"**
   - **Connection name:** `Fathom Production`
   - **API Key:** Paste your Fathom API key
   - Click Save
5. Configure trigger:
   - **Workspace ID:** Your workspace ID
   - **Limit:** 10 (processes up to 10 new recordings per run)
6. Click **"Schedule settings"** (clock icon)
   - **Interval:** 15 minutes
   - This checks for new recordings every 15 min

**Output variables you'll use later:**
- `{{1.id}}` - Recording ID (needed for transcript/summary)
- `{{1.title}}` - Call title
- `{{1.createdAt}}` - Recording date
- `{{1.duration}}` - Call duration in seconds
- `{{1.shareUrl}}` - Link to Fathom recording

**Test it:** Click "Run once" - it should show any recent recordings

---

### Module [2]: Fathom Get a Transcript

**Purpose:** Retrieves full transcript text

**Setup:**

1. Click + after module 1 → Search "Fathom"
2. Select **"Get a Transcript"**
3. Connection: Use existing `Fathom Production`
4. Configure:
   - **Recording ID:** Click field → Select `{{1.id}}` from Module 1

**Output:**
- `{{2.transcript}}` - Full transcript text

**Note:** If no transcript exists yet (Fathom still processing), this module will return empty. Add a 2-minute delay if processing very recent calls.

---

### Module [3]: Fathom Get a Summary

**Purpose:** Retrieves Fathom's AI-generated summary

**Setup:**

1. Click + → Search "Fathom"
2. Select **"Get a Summary"**
3. Connection: Use existing `Fathom Production`
4. Configure:
   - **Recording ID:** `{{1.id}}`

**Output:**
- `{{3.summary}}` - Fathom's summary text

---

### Module [4]: Notion Create Database Item

**Purpose:** Creates call record in Call Intelligence Hub

**Setup:**

1. Click + → Search "Notion"
2. Select **"Create a Database Item"**
3. Click **"Create a connection"**
   - **Connection name:** `Notion Production`
   - **Token:** Paste your Notion Internal Integration Token
   - Click Save
4. Configure:
   - **Database ID:** Paste Call Intelligence Hub database ID

5. **Map properties** (match these to your Notion database property names EXACTLY):

```
Property Mapping:

Call Title (Title property):
  → {{1.title}}

Call Date (Date property):
  → {{1.createdAt}}

Recording URL (URL property):
  → {{1.shareUrl}}

Duration (Number property):
  → {{1.duration}}
  Note: This is in seconds. To convert to minutes: {{1.duration / 60}}

Raw Transcript (Long Text property):
  → {{2.transcript}}

Fathom Summary (Long Text property):
  → {{3.summary}}

Processing Status (Select property):
  → Processing
  (Manually type this - it's a fixed value)

Sentiment Score (Select property):
  → Leave empty for now (Path B will populate this)
```

**Critical:** Save the output variable name for later use:
- `{{4.id}}` - This is the Notion page ID you'll need for relations

**Troubleshooting:**
- If property names don't appear, verify your integration has access to the database
- Property names are case-sensitive
- Select/Multi-select properties need exact option names

---

### Module [5]: Router

**Purpose:** Splits workflow into 3 parallel processing paths

**Setup:**

1. Click + → Search "Router"
2. Select **"Router"**
3. You'll see it creates a router with 2 paths by default
4. Click **"Add route"** at bottom to add a 3rd path

**Routes:**
- **Route 1:** Content Ideas Generator
- **Route 2:** Client Intelligence Analyzer
- **Route 3:** Executive Summary Generator

**Filters:** Leave all paths without filters (they all run every time)

**What this does:** After the router, the workflow branches into 3 independent paths that run in parallel. Each path can access all previous module outputs (1-4).

---

## Part 2: Path A - Content Ideas Generator

**Goal:** Extract 5-10 content ideas and auto-draft high-priority ones

### Module [5A]: HTTP Request - Generate Content Ideas

**Purpose:** Call OpenRouter API to analyze transcript and extract content ideas

**Setup:**

1. In Route 1, click + → Search "HTTP"
2. Select **"Make a request"**
3. Configure:

**URL:**
```
https://openrouter.ai/api/v1/chat/completions
```

**Method:** POST

**Headers:**
Click "Add item" twice to add these headers:

```
Header 1:
Name: Authorization
Value: Bearer YOUR_OPENROUTER_API_KEY
(Replace with your actual key)

Header 2:
Name: Content-Type
Value: application/json
```

**Body type:** Raw

**Content type:** JSON (application/json)

**Request content:** Paste this JSON (Make.com will let you insert variables):

```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [
    {
      "role": "system",
      "content": "You are a content strategist helping coaches create viral LinkedIn content and newsletter material from their client calls. Your output must be valid JSON only."
    },
    {
      "role": "user",
      "content": "Analyze this coaching call transcript and extract 5-10 high-value content ideas.\n\nCall Title: {{1.title}}\n\nTranscript:\n{{2.transcript}}\n\nFor each content idea, provide:\n1. title: Attention-grabbing title (5-8 words)\n2. contentType: LinkedIn Post, Newsletter, Carousel, Thread, or Video Script\n3. hook: Opening line that stops the scroll (10-15 words)\n4. coreConcept: The main insight or teaching (2-3 sentences)\n5. keyPoints: 3-5 bullet points to cover (as array)\n6. whyItResonates: Why this connects with the coach's audience (1-2 sentences)\n7. platformFit: Best platforms for this content (array of strings)\n8. priorityScore: 1-10 based on virality potential and relevance\n\nRules:\n- Focus on client pain points, transformations, and aha moments from the call\n- Make hooks specific and curiosity-driven\n- Prioritize tactical, actionable content over theory\n- Score based on: clarity of value (40%), audience fit (30%), viral potential (30%)\n\nReturn ONLY a valid JSON array with no markdown code blocks:\n[{\"title\": \"...\", \"contentType\": \"...\", \"hook\": \"...\", \"coreConcept\": \"...\", \"keyPoints\": [...], \"whyItResonates\": \"...\", \"platformFit\": [...], \"priorityScore\": 8}]"
    }
  ],
  "temperature": 0.7
}
```

**Cost:** ~$0.10-0.15 per call with GPT-4o-mini

**Output:**
- `{{5A.data.choices[0].message.content}}` - JSON array of content ideas

**Troubleshooting:**
- If you get 401 error: Check API key in headers
- If you get 402 error: Add credit to OpenRouter account
- If response is empty: Check your Fathom transcript has content

---

### Module [6A]: JSON Parse Response

**Purpose:** Convert JSON string to usable array

**Setup:**

1. Click + → Search "JSON"
2. Select **"Parse JSON"**
3. Configure:
   - **JSON string:** `{{5A.data.choices[0].message.content}}`

**Output:**
- `{{6A}}` becomes an array you can iterate over
- Each item has: title, contentType, hook, coreConcept, etc.

---

### Module [7A]: Iterator

**Purpose:** Loop through each content idea

**Setup:**

1. Click + → Search "Iterator"
2. Select **"Iterator"**
3. Configure:
   - **Array:** `{{6A.array}}`

**What this does:** Everything after this module runs once per content idea

---

### Module [8A]: Notion Create Content Idea

**Purpose:** Save each content idea to Content Ideas Bank

**Setup:**

1. Click + → Search "Notion"
2. Select **"Create a Database Item"**
3. Connection: Use existing `Notion Production`
4. Configure:
   - **Database ID:** Content Ideas Bank database ID

5. **Map properties:**

```
Idea Title (Title):
  → {{7A.title}}

Content Type (Select):
  → {{7A.contentType}}

Source Call (Relation):
  → {{4.id}}
  CRITICAL: This links to the call record created in module [4]

Raw Idea (Long Text):
  → {{7A.coreConcept}}

Hook/Opening (Text):
  → {{7A.hook}}

Key Points (Long Text):
  → {{join(7A.keyPoints; "\n• ")}}
  This converts array to bullet list

Priority Score (Number):
  → {{7A.priorityScore}}

Platform Fit (Multi-select):
  → {{7A.platformFit}}
  Make.com will automatically handle the array

Status (Select):
  → New Idea
  (Fixed value)
```

**Output:**
- `{{8A.id}}` - The Notion page ID of created content idea

---

### Module [9A]: Filter High-Priority Ideas

**Purpose:** Only draft content for ideas with priority 8+

**Setup:**

1. Click + → Search "Filter"
2. Select **"Filter"**
3. Click "Set up filter"
4. Configure condition:
   - **Field:** `{{7A.priorityScore}}`
   - **Operator:** Numeric: Greater than or equal to
   - **Value:** 8

**What happens:** If priority score >= 8, continue to drafting. Otherwise, skip to next idea.

---

### Module [10A]: HTTP Request - Draft Content

**Purpose:** Generate full content draft for high-priority ideas

**Setup:**

1. Click + → Search "HTTP"
2. Select **"Make a request"**
3. Configure same as [5A]:
   - **URL:** `https://openrouter.ai/api/v1/chat/completions`
   - **Method:** POST
   - **Headers:** Same Authorization and Content-Type

**Request content:**

```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [
    {
      "role": "system",
      "content": "You are a professional ghostwriter creating ready-to-post content for coaches."
    },
    {
      "role": "user",
      "content": "Write a complete {{7A.contentType}} based on this content idea.\n\nIdea Title: {{7A.title}}\nHook: {{7A.hook}}\nCore Concept: {{7A.coreConcept}}\nKey Points:\n{{join(7A.keyPoints; \"\n• \")}}\n\nStyle Guidelines:\n- Conversational, direct tone\n- Short paragraphs (2-3 sentences max)\n- Lead with the hook to grab attention\n- Deliver value fast - no fluff\n- End with engagement question or CTA\n\nLength Requirements:\n- LinkedIn Post: 150-200 words\n- Newsletter: 300-400 words  \n- Carousel: 8 slides with title + 3-4 bullets each\n- Thread: 8-10 tweets\n- Video Script: 60-90 second script with timestamps\n\nOutput the complete, ready-to-edit draft with no meta-commentary."
    }
  ],
  "temperature": 0.7
}
```

**Output:**
- `{{10A.data.choices[0].message.content}}` - Full drafted content

**Cost:** ~$0.15-0.20 per draft

---

### Module [11A]: Notion Update Content Idea

**Purpose:** Add the drafted content to the content idea record

**Setup:**

1. Click + → Search "Notion"
2. Select **"Update a Database Item"**
3. Connection: Use existing `Notion Production`
4. Configure:
   - **Page ID:** `{{8A.id}}` (from the content idea we created)

5. **Update properties:**

```
Polished Draft (Long Text):
  → {{10A.data.choices[0].message.content}}

Status (Select):
  → Ready to Draft
```

**Path A Complete!** Now every content idea is saved, and high-priority ones have full drafts.

---

## Part 3: Path B - Client Intelligence Analyzer

**Goal:** Extract client health signals and populate Client Intelligence database

### Module [5B]: HTTP Request - Analyze Intelligence

**Purpose:** Use Claude to analyze client health and extract strategic insights

**Setup:**

1. In Route 2, click + → Search "HTTP"
2. Select **"Make a request"**
3. Configure:
   - **URL:** `https://openrouter.ai/api/v1/chat/completions`
   - **Method:** POST
   - **Headers:** Same as before (Authorization + Content-Type)

**Request content:**

```json
{
  "model": "anthropic/claude-3.5-sonnet",
  "messages": [
    {
      "role": "user",
      "content": "Analyze this client call for strategic intelligence and relationship health indicators.\n\nCall Title: {{1.title}}\n\nTranscript:\n{{2.transcript}}\n\nFathom Summary:\n{{3.summary}}\n\nProvide a comprehensive analysis with:\n\n1. sentiment: Overall client sentiment classification (choose one: \"Improving\", \"Stable\", \"Watch\", \"At Risk\")\n2. sentimentConfidence: 0-100 (your confidence in the sentiment assessment)\n3. sentimentEvidence: Array of 2-3 specific quotes from transcript supporting your assessment\n4. painPoints: Array of current challenges, frustrations, or obstacles mentioned (3-5 items)\n5. wins: Array of recent successes, progress, or victories celebrated (2-3 items)\n6. upsellOpportunities: Array of new needs, expanded scope hints, wish-list items, or \"if only we had...\" statements (2-3 items)\n7. redFlags: Array of warning signs like dissatisfaction, competitor mentions, budget concerns, scope reduction, delayed responses (0-3 items, empty array if none)\n8. nextActions: Array of what must happen next to maintain or improve relationship (2-3 items)\n9. clientName: Extract the client's name from the call title or transcript\n\nAnalysis Guidelines:\n- Sentiment \"Improving\": Positive momentum, expressing gratitude, expanding scope\n- Sentiment \"Stable\": Neutral tone, steady progress, no major concerns\n- Sentiment \"Watch\": Minor concerns, slowing momentum, needs attention\n- Sentiment \"At Risk\": Expressing dissatisfaction, missing meetings, budget concerns\n- Look for subtle signals, not just explicit statements\n- Upsell opportunities can be implicit (\"I wish we could...\")\n- Red flags include scope creep pushback, payment delays, reduced engagement\n\nReturn ONLY valid JSON with no markdown:\n{\"sentiment\": \"Stable\", \"sentimentConfidence\": 85, \"sentimentEvidence\": [...], \"painPoints\": [...], \"wins\": [...], \"upsellOpportunities\": [...], \"redFlags\": [], \"nextActions\": [...], \"clientName\": \"...\"}"
    }
  ],
  "temperature": 0.3
}
```

**Why Claude:** Better at nuanced analysis and extracting subtle signals than GPT-4

**Cost:** ~$0.25-0.35 per analysis

**Output:**
- `{{5B.data.choices[0].message.content}}` - JSON with intelligence data

---

### Module [6B]: JSON Parse Response

**Setup:**

1. Click + → Search "JSON"
2. Select **"Parse JSON"**
3. Configure:
   - **JSON string:** `{{5B.data.choices[0].message.content}}`

**Output variables:**
- `{{6B.sentiment}}`
- `{{6B.clientName}}`
- `{{6B.painPoints}}` (array)
- `{{6B.wins}}` (array)
- `{{6B.upsellOpportunities}}` (array)
- `{{6B.redFlags}}` (array)
- etc.

---

### Module [7B]: Notion Search for Existing Client

**Purpose:** Check if client already has a record in Client Intelligence database

**Setup:**

1. Click + → Search "Notion"
2. Select **"Search Objects"**
3. Connection: Use existing `Notion Production`
4. Configure:
   - **Search:** `{{6B.clientName}}`
   - **Filter by:** Database
   - **Database ID:** Client Intelligence database ID
   - **Limit:** 1

**Output:**
- If found: `{{7B.id}}` exists
- If not found: No results

**Next:** Add a router to handle both cases

---

### Module [8B]: Router - Create or Update

**Purpose:** Branch based on whether client record exists

**Setup:**

1. Click + → Search "Router"
2. Select **"Router"**

**Route 1: Update Existing Client**
- Click "Set up a filter"
- Condition: `{{7B.id}}` → Operator: "Exists"
- This runs if we found an existing client record

**Route 2: Create New Client**
- Leave without filter (fallback route)
- This runs if no existing record found

---

### Module [8B-1]: Update Existing Client

**In Route 1 of router [8B]:**

1. Click + → Search "Notion"
2. Select **"Update a Database Item"**
3. Connection: Use existing `Notion Production`
4. Configure:
   - **Page ID:** `{{7B.id}}`

5. **Update properties:**

```
Last Call Date (Date):
  → {{1.createdAt}}

Current Sentiment (Select):
  → {{6B.sentiment}}

Pain Points (Multi-select):
  → {{6B.painPoints}}
  Make.com will add new items without removing existing

Recent Wins (Long Text):
  → {{formatDate(1.createdAt; "MM/DD/YYYY")}} Call:\n{{join(6B.wins; "\n• ")}}\n\n{{7B.Recent Wins}}
  This prepends new wins to existing ones

Upsell Opportunities (Long Text):
  → {{join(6B.upsellOpportunities; "\n• ")}}

Red Flags (Long Text):
  → {{if(length(6B.redFlags) > 0; join(6B.redFlags; "\n🚩 "); "None detected")}}
```

---

### Module [8B-2]: Create New Client

**In Route 2 (fallback) of router [8B]:**

1. Click + → Search "Notion"
2. Select **"Create a Database Item"**
3. Connection: Use existing `Notion Production`
4. Configure:
   - **Database ID:** Client Intelligence database ID

5. **Map properties:**

```
Client Name (Title):
  → {{6B.clientName}}

Last Call Date (Date):
  → {{1.createdAt}}

Current Sentiment (Select):
  → {{6B.sentiment}}

Pain Points (Multi-select):
  → {{6B.painPoints}}

Recent Wins (Long Text):
  → {{join(6B.wins; "\n• ")}}

Upsell Opportunities (Long Text):
  → {{join(6B.upsellOpportunities; "\n• ")}}

Red Flags (Long Text):
  → {{if(length(6B.redFlags) > 0; join(6B.redFlags; "\n🚩 "); "None detected")}}
```

**Path B Complete!** Now client intelligence is automatically tracked and updated.

---

## Part 4: Path C - Executive Summary Generator

**Goal:** Create professional call summary in Markdown

### Module [5C]: HTTP Request - Generate Summary

**Purpose:** Create executive summary of the call

**Setup:**

1. In Route 3, click + → Search "HTTP"
2. Select **"Make a request"**
3. Configure:
   - **URL:** `https://openrouter.ai/api/v1/chat/completions`
   - **Method:** POST
   - **Headers:** Same Authorization + Content-Type

**Request content:**

```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [
    {
      "role": "system",
      "content": "You are a professional executive assistant creating polished call summaries."
    },
    {
      "role": "user",
      "content": "Create a professional executive summary of this call in clean Markdown format.\n\nCall Title: {{1.title}}\nDate: {{formatDate(1.createdAt; \"MMMM DD, YYYY\")}}\nDuration: {{floor(1.duration / 60)}} minutes\n\nTranscript:\n{{2.transcript}}\n\nFathom Summary:\n{{3.summary}}\n\nFormat exactly like this:\n\n## 📊 Call Overview\n- **Date:** [date]\n- **Duration:** [X] minutes\n- **Type:** [Client Check-in / Strategy Session / Implementation Review / Discovery Call]\n\n## 🎯 Key Outcomes\n[3-5 bullet points of most important decisions, commitments, or insights delivered]\n\n## 💡 Top Insights\n[3-4 strategic learnings or client revelations that matter]\n\n## ✅ Actions Captured\n- Content Ideas Generated: [count from your analysis]\n- Key Follow-up Items: [if applicable]\n- Decisions Made: [if applicable]\n\n## 🚀 Next Steps\n[3-5 immediate actions with suggested owners and timing]\n\n## ⚠️ Watch Items\n[Only include this section if there are concerns, blockers, or risks. Otherwise omit.]\n\nGuidelines:\n- Be concise and scannable\n- Focus on decisions and actionable items\n- Use specific numbers and commitments\n- Highlight value delivered\n- Keep professional but conversational tone"
    }
  ],
  "temperature": 0.5
}
```

**Output:**
- `{{5C.data.choices[0].message.content}}` - Markdown formatted summary

---

### Module [6C]: Update Call Record with Summary

**Purpose:** Add executive summary to the call record

**Setup:**

1. Click + → Search "Notion"
2. Select **"Update a Database Item"**
3. Connection: Use existing `Notion Production`
4. Configure:
   - **Page ID:** `{{4.id}}` (from the call record we created at the start)

5. **Update properties:**

```
Executive Summary (Long Text):
  → {{5C.data.choices[0].message.content}}

Processing Status (Select):
  → Complete
```

**Path C Complete!** Professional summary added to call record.

---

## Part 5: Testing & Validation

### Pre-flight Checklist

Before running the scenario:

- [ ] All modules are connected (no orphaned modules)
- [ ] All 3 router paths are complete
- [ ] OpenRouter API key is correct in all HTTP modules
- [ ] Notion connection has access to all 3 databases
- [ ] Database IDs are correct
- [ ] Property names match exactly (case-sensitive)

### Test Run

1. **Save the scenario** (Ctrl+S or Save button)
2. **Name it:** "Fathom Content Intelligence Pipeline"
3. Click **"Run once"** button
4. Watch the execution:
   - Blue = running
   - Green = success
   - Red = error

### Expected Results

After successful run, check Notion:

**Call Intelligence Hub:**
- [ ] New call record created
- [ ] Transcript and summary populated
- [ ] Executive Summary populated
- [ ] Processing Status = "Complete"

**Content Ideas Bank:**
- [ ] 5-10 new content idea records
- [ ] Each linked to source call (Relation working)
- [ ] Priority scores assigned
- [ ] High-priority ideas (8+) have Polished Draft field filled
- [ ] Status = "New Idea" or "Ready to Draft"

**Client Intelligence:**
- [ ] Client record created or updated
- [ ] Sentiment score assigned
- [ ] Pain points listed
- [ ] Upsell opportunities captured

### Troubleshooting Common Errors

**Error: "Property not found"**
- Solution: Check property name spelling in Notion (case-sensitive)
- Verify: Integration has access to database

**Error: "Invalid database ID"**
- Solution: Get database ID from full-page URL, not inline database
- Format: 32-character string, no dashes

**Error: 401 from OpenRouter**
- Solution: Check API key in HTTP module headers
- Verify: Key starts with "sk-or-..."

**Error: "Relation could not be created"**
- Solution: Verify `{{4.id}}` exists before trying to relate
- Check: Both databases share same integration access

**Error: JSON parse failed**
- Solution: OpenRouter returned non-JSON (error message or markdown)
- Check: Prompt explicitly says "return ONLY valid JSON"
- Try: Lower temperature (0.3 instead of 0.7)

**Error: Iterator has no items**
- Solution: JSON parsing didn't create array
- Check: `{{5A.data.choices[0].message.content}}` has valid JSON
- Try: Add JSON parsing step before iterator

---

## Part 6: Optimization & Monitoring

### Enable Scenario

1. Toggle scenario to **ON** (top right)
2. Set schedule: Every 15 minutes
3. Verify webhook is active

### Monitor Operations Usage

**Make.com free tier:** 1,000 operations/month

**Per call cost:**
- Trigger: 1
- Get transcript: 1
- Get summary: 1
- Create call: 1
- Path A: ~10-15 ops (depending on idea count)
- Path B: ~3-5 ops
- Path C: ~2 ops
- **Total: ~20-25 ops per call**

**At 40 calls/month:** ~800-1,000 ops (at free tier limit)
**At 50+ calls/month:** Upgrade to Core ($9/mo, 10,000 ops)

### Error Notifications

1. Go to scenario settings
2. Enable **"Email on error"**
3. Set **Max consecutive errors: 3**
4. This alerts you if something breaks

### Execution History

- View last 30 days of runs
- Click any execution to see step-by-step data
- Download execution logs for debugging

---

## Next Steps

Once core workflow is running:

1. **Monitor for 1 week** - Verify all paths working correctly
2. **Adjust AI prompts** - Refine based on output quality (see `config/openrouter-prompts.md`)
3. **Add error handling** - Add retries for API timeouts
4. **Build dashboard views** - See `docs/05-dashboard-views.md`
5. **Phase 2 features** - Task generator, advanced intelligence

---

## Quick Reference: Module Variables

**Module 1 (Fathom Trigger):**
- `{{1.id}}` - Recording ID
- `{{1.title}}` - Call title
- `{{1.createdAt}}` - Date
- `{{1.duration}}` - Duration (seconds)
- `{{1.shareUrl}}` - Fathom link

**Module 4 (Create Call Record):**
- `{{4.id}}` - Call record page ID (for relations)

**Module 6A (Content Ideas Array):**
- `{{6A.array}}` - All ideas

**Module 7A (Iterator):**
- `{{7A.title}}` - Idea title
- `{{7A.contentType}}` - Type
- `{{7A.priorityScore}}` - Score
- `{{7A.hook}}` - Hook line
- etc.

**Module 8A (Created Content Idea):**
- `{{8A.id}}` - Content idea page ID

**Module 6B (Intelligence Data):**
- `{{6B.sentiment}}` - Client sentiment
- `{{6B.clientName}}` - Extracted client name
- `{{6B.painPoints}}` - Array
- etc.

---

## Support

**Make.com Documentation:** make.com/en/help
**Fathom API Docs:** fathom.video/api-docs
**OpenRouter Docs:** openrouter.ai/docs
**Notion API Docs:** developers.notion.com

**Stuck?** Check execution history → Click failed module → View error details
