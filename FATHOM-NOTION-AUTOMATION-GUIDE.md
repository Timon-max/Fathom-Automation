# 🎯 Fathom to Notion Automation - Complete Guide

---

## 📋 Overview

### What Does This Automation Do?

This automation is your personal AI-powered meeting assistant that works 24/7 in the background. Every time you finish a call using Fathom (a meeting recording tool), this automation springs into action automatically.

Here's the magic that happens:

**🎬 Step 1:** Fathom finishes recording your client call, sales meeting, or team discussion.

**⚡ Step 2:** The moment the recording is ready, Fathom sends all the meeting details to this automation.

**🤖 Step 3:** The automation uses AI (powered by OpenAI) to analyze your meeting transcript and extract three types of valuable insights:
- 💡 Content ideas for your social media
- 📊 Client intelligence and business insights
- ✅ Action items and tasks

**📝 Step 4:** All this information is automatically organized and saved into your Notion workspace in beautifully structured databases.


### Why Is This Valuable?

Instead of manually reviewing hours of call recordings, this automation does the heavy lifting for you. It transforms raw conversations into actionable insights, content opportunities, and clear next steps - all while you focus on what matters most: serving your clients and growing your business.


---

## 🏗️ How The Automation Works

This automation is built using Make.com (an automation platform) and consists of several connected modules that work together like a well-oiled machine.


### The Complete Flow

```
Fathom Call Recording
    ↓
Webhook Receives Data
    ↓
Create Initial Notion Entry
    ↓
Router (Splits into 3 Parallel Paths)
    ↓
┌───────────────┬──────────────────────┬───────────────┐
│   Path 1      │      Path 2          │    Path 3     │
│ Content Ideas │ Client Intelligence  │  Task Tracker │
└───────────────┴──────────────────────┴───────────────┘
    ↓                    ↓                      ↓
AI Processing        AI Processing          AI Processing
    ↓                    ↓                      ↓
Save to Notion       Save to Notion         Save to Notion
```


---

## 🔧 Module Breakdown

Let's break down each module so you understand exactly what's happening at each step.


---

### 📥 Module 1: Fathom Webhook (The Trigger)

**What It Does:**

This is the starting point of your automation. Think of it as a doorbell that rings every time Fathom finishes recording a call.

When you complete a call in Fathom, the app sends a notification (called a webhook) to Make.com with all the meeting details.


**Data Received:**

The webhook receives the following information from Fathom:
- Meeting title
- Recording start time
- Recording end time
- Full transcript of the conversation
- AI-generated summary (created by Fathom)
- URL link to the recording
- Participant information


**Field Mappings:**

| Fathom Field | What It Contains | Example |
|---|---|---|
| `meeting_title` | The name of your meeting | "Client Onboarding - John Smith" |
| `recording_start_time` | When the recording began | "2025-01-15T14:00:00Z" |
| `recording_end_time` | When the recording ended | "2025-01-15T15:00:00Z" |
| `transcript[].text` | Complete conversation text | Full word-for-word transcript |
| `default_summary` | Fathom's AI summary | Key points from the call |
| `url` | Link to the recording | "https://app.fathom.video/..." |


**Common Errors & Solutions:**

❌ **Error: "Webhook data not received"**
- **Why it happens:** Fathom didn't send the notification, or the webhook connection is broken
- **How to fix:** Go to Fathom settings and verify the webhook URL is correctly configured. Test by triggering a sample webhook.

❌ **Error: "Missing transcript data"**
- **Why it happens:** The call recording hasn't finished processing in Fathom yet
- **How to fix:** Fathom needs a few minutes to process recordings. The webhook fires when ready, so this usually resolves on its own.

❌ **Error: "Webhook validation failed"**
- **Why it happens:** The connection between Fathom and Make.com is not authenticated
- **How to fix:** Reconnect the webhook in Make.com. Delete the old webhook and create a new one.


---

### 📄 Module 2: Create Notion Page (Initial Entry)

**What It Does:**

This module creates the first entry in your Notion database immediately after receiving the webhook. It's like creating a new file folder that will hold all your meeting information.

This initial page captures the basic meeting details before the AI processing happens.


**Field Mappings:**

The automation maps Fathom data to your Notion database fields:

| Notion Field | Fathom Source | Mapping Reference | Example |
|---|---|---|---|
| **Call Title** | `meeting_title` | `{{1.meeting_title}}` | "Discovery Call - Sarah Johnson" |
| **Call Date (Start)** | `recording_start_time` | `{{1.recording_start_time}}` | January 15, 2025 |
| **Call Date (End)** | `recording_end_time` | `{{1.recording_end_time}}` | January 15, 2025 |
| **AI Summary** | `default_summary` (truncated) | `{{substring(1.default_summary; 0; 2000)}}` | First 2000 characters of summary |
| **Recording URL** | `url` | `{{1.url}}` | https://app.fathom.video/... |


**Important Notes:**

⚠️ **Character Limit on AI Summary:** The summary field is limited to 2000 characters. The automation uses a `substring` function to trim longer summaries.

⚠️ **Include Time Setting:** The date field is set to NOT include time (shows only the date, not the hour/minute).

⚠️ **Database ID:** This connects to your specific Notion database using the ID `6a1467612d6d4afe82370bf0e4fa1109`


**Common Errors & Solutions:**

❌ **Error: "Database not found"**
- **Why it happens:** The Notion database ID is incorrect or the database was deleted
- **How to fix:** Open your Notion database, copy the database ID from the URL, and update it in the module settings

❌ **Error: "Missing required property: title"**
- **Why it happens:** The Call Title field didn't receive data from Fathom
- **How to fix:** Check that `meeting_title` exists in the webhook data. You may need to add a fallback value like "Untitled Call"

❌ **Error: "Integration lacks permission"**
- **Why it happens:** The Notion connection doesn't have access to the target database
- **How to fix:** In Notion, share the database with your Make.com integration. Click "Share" → Add your integration

❌ **Error: "Invalid date format"**
- **Why it happens:** Fathom sent dates in an unexpected format
- **How to fix:** Add a date formatting function: `{{formatDate(1.recording_start_time; "YYYY-MM-DD")}}`


---

### 🔀 Module 3: Basic Router (The Traffic Controller)

**What It Does:**

The router is like a traffic controller at an intersection. It takes the meeting data and sends it down THREE different paths simultaneously.

Each path handles a different type of AI analysis:
- **Route 1:** Content Ideas Extraction
- **Route 2:** Client Intelligence Tracker
- **Route 3:** Task Extraction

All three routes run at the same time (in parallel), making the automation super fast and efficient.


**How It Works:**

Think of it like making photocopies. The router takes your original meeting data and makes three copies, sending each copy to a different AI processing station.


**Technical Details:**

- **Module Type:** Built-in Basic Router
- **Number of Routes:** 3
- **Execution Mode:** Parallel (all routes run simultaneously)
- **No Conditions:** All routes always run (no filtering)


**Common Errors & Solutions:**

❌ **Error: "Route execution failed"**
- **Why it happens:** One of the three routes encountered an error
- **How to fix:** Check each route individually to see which one failed. The router will show which path had the issue.

❌ **Error: "Insufficient operations"**
- **Why it happens:** Your Make.com plan ran out of operations
- **How to fix:** Upgrade your Make.com plan or wait until your operations reset next month

💡 **Pro Tip:** Even if one route fails, the other two will still complete successfully!


---

### 🎨 Route 1: Content Ideas Extraction

This route transforms your client calls into viral content opportunities.


---

#### 🤖 Step 1: OpenAI GPT-4 Mini (Content Ideas)

**What It Does:**

This AI module analyzes your entire call transcript and identifies THREE high-value content ideas that you can turn into social media posts, newsletters, carousels, threads, or video scripts.

The AI acts as your personal content strategist, looking for:
- Client pain points discussed in the call
- Aha moments and transformations
- Tactical insights your audience would love
- Stories and examples that resonate


**AI Prompt Breakdown:**

The AI receives these instructions:

**System Role:** "You are a content strategist helping agencies, coaches, consultants, solopreneurs and founders create viral Social Media Content from their client calls."

**User Prompt:** The AI analyzes:
- Call title
- Full transcript
- And extracts for each idea:
  - **Title:** Attention-grabbing (5-8 words)
  - **Content Type:** Social Media Post, Newsletter, Carousel, Thread, or Video Script
  - **Hook:** Opening line that stops the scroll (10-15 words)
  - **Core Concept:** Main insight or teaching (2-3 sentences)
  - **Key Points:** 3-5 bullet points to cover
  - **Why It Resonates:** Why it connects with your audience
  - **Platform Fit:** Best platforms (Instagram, LinkedIn, Twitter, etc.)
  - **Priority Score:** 1-10 based on viral potential


**Field Mappings:**

| Input Field | Source | Mapping |
|---|---|---|
| Call Title | Fathom webhook | `{{1.meeting_title}}` |
| Transcript | Fathom webhook | `{{1.transcript[].text}}` |


**Model Settings:**

- **AI Model:** GPT-4o-mini (fast and cost-effective)
- **Max Tokens:** 8,192 (allows for detailed responses)
- **Temperature:** 1 (creative responses)
- **Response Format:** JSON array


**Common Errors & Solutions:**

❌ **Error: "Invalid API key"**
- **Why it happens:** OpenAI connection is broken or API key expired
- **How to fix:** Reconnect your OpenAI account in Make.com. Generate a new API key if needed.

❌ **Error: "Token limit exceeded"**
- **Why it happens:** Your transcript is too long for the AI to process
- **How to fix:** Reduce the max_tokens setting or summarize the transcript before sending to AI

❌ **Error: "Invalid JSON response"**
- **Why it happens:** The AI didn't return properly formatted JSON
- **How to fix:** This is rare but can happen. Add error handling to retry or alert you

❌ **Error: "Rate limit reached"**
- **Why it happens:** Too many API calls to OpenAI in a short time
- **How to fix:** Add a 2-3 second delay before this module, or upgrade your OpenAI plan


---

#### 📊 Step 2: Parse JSON (Content Ideas)

**What It Does:**

The AI responds with text that looks like JSON (structured data), but Make.com can't use it directly. This module converts that text into usable data fields.

Think of it like translating a foreign language into English so everyone can understand it.


**Field Mappings:**

| Output Field | Description |
|---|---|
| `title` | The content idea title |
| `contentType` | Type of content (Post, Newsletter, etc.) |
| `hook` | The attention-grabbing opening line |
| `coreConcept` | The main teaching point |
| `keyPoints[]` | Array of bullet points |
| `whyItResonates` | Why audience will love it |
| `platformFit[]` | Best platforms for this content |
| `priorityScore` | Virality score 1-10 |


**Common Errors & Solutions:**

❌ **Error: "Invalid JSON string"**
- **Why it happens:** The AI response wasn't valid JSON format
- **How to fix:** Check the AI's response. You may need to improve the AI prompt to ensure JSON output

❌ **Error: "Unexpected token"**
- **Why it happens:** There's a syntax error in the JSON (like a missing comma or bracket)
- **How to fix:** Add validation in the previous step or manually fix the JSON structure


---

#### ♻️ Step 3: Basic Feeder (Content Ideas)

**What It Does:**

The AI gives us THREE content ideas in one response. But we want to create THREE separate entries in Notion (one for each idea).

The Feeder module splits that single bundle of three ideas into three individual bundles - one per content idea.

**Visual Example:**

```
BEFORE Feeder:
[Idea 1, Idea 2, Idea 3] → Single bundle

AFTER Feeder:
Idea 1 → Bundle 1
Idea 2 → Bundle 2
Idea 3 → Bundle 3
```


**Common Errors & Solutions:**

❌ **Error: "No bundles to iterate"**
- **Why it happens:** The JSON Parse step didn't output an array
- **How to fix:** Ensure the AI returns a JSON array `[{...}, {...}, {...}]` not a single object


---

#### 📝 Step 4: Create Notion Page (Content Ideas Database)

**What It Does:**

This module creates a NEW page in your Notion "Content Ideas" database for each of the three content ideas.

Each page includes all the details extracted by AI, beautifully organized and ready for you to turn into actual content.


**Field Mappings:**

This assumes you have a Notion database with these fields:

| Notion Field | Mapped Data | Source |
|---|---|---|
| **Title** | Content idea title | `{{8.title}}` |
| **Content Type** | Type (Post, Newsletter, etc.) | `{{8.contentType}}` |
| **Hook** | Opening line | `{{8.hook}}` |
| **Core Concept** | Main teaching | `{{8.coreConcept}}` |
| **Key Points** | Bullet points array | `{{8.keyPoints}}` |
| **Why It Resonates** | Audience connection | `{{8.whyItResonates}}` |
| **Platform Fit** | Best platforms | `{{8.platformFit}}` |
| **Priority Score** | Virality score | `{{8.priorityScore}}` |
| **Source Call** | Link to original call | Reference to Module 2 |


**Common Errors & Solutions:**

❌ **Error: "Property does not exist"**
- **Why it happens:** Your Notion database is missing one of these fields
- **How to fix:** Add the missing property to your Notion database or remove it from the mapping

❌ **Error: "Invalid select value"**
- **Why it happens:** The AI returned a content type not in your Notion dropdown options
- **How to fix:** Update your Notion select field to include all possible values, or add validation


---

### 📊 Route 2: Client Intelligence Tracker

This route extracts strategic business insights from every client conversation.


---

#### 🤖 Step 1: OpenAI GPT-4 Mini (Client Intelligence)

**What It Does:**

This AI module acts as your business intelligence analyst. It reviews the call transcript and extracts strategic information about:
- Who the client is
- What problems they're facing
- What they want to achieve
- Any concerns or objections
- How engaged they are
- What you should do next


**AI Prompt Breakdown:**

**System Role:** "You are a business intelligence analyst helping coaches, consultants, founders and agencies extract strategic insights from client calls."

**Extraction Points:**

The AI identifies and structures:

1. **Client Name:** Who is the client? (if mentioned in the call)

2. **Primary Pain Points:** What problems is the client facing? (2-3 bullet points)
   - Example: "Struggling to generate consistent leads"

3. **Stated Goals:** What outcomes does the client want? (2-3 bullet points)
   - Example: "Double revenue in the next 6 months"

4. **Objections or Concerns:** Any hesitations mentioned (array of strings)
   - Example: "Budget constraints", "Time commitment"

5. **Transformation Opportunities:** Where you can help (2-3 sentences)
   - Example: "Client needs help with sales process automation and lead nurturing"

6. **Recommended Next Steps:** Specific actions to take (3-5 action items)
   - Example: "Send proposal by Friday", "Schedule follow-up call"

7. **Client Engagement Level:** Rate 1-10 how interested they are
   - Based on enthusiasm, questions asked, commitment signals

8. **Key Quotes:** 2-3 memorable quotes that reveal client mindset
   - Example: "I'm tired of leaving money on the table"

9. **Decision Timeline:** When do they need results or want to decide?
   - Example: "Needs to make decision by end of month"


**Field Mappings:**

| Input Field | Source | Mapping |
|---|---|---|
| Call Title | Fathom webhook | `{{1.meeting_title}}` |
| Transcript | Fathom webhook | `{{1.transcript[].text}}` |


**Model Settings:**

- **AI Model:** GPT-4o-mini
- **Max Tokens:** 4,096
- **Temperature:** 1
- **Response Format:** JSON object


**Common Errors & Solutions:**

❌ **Error: "Engagement level not a number"**
- **Why it happens:** AI returned text instead of a number for engagement level
- **How to fix:** Add validation: `{{parseNumber(10.engagementLevel)}}` to force number format

❌ **Error: "Missing client name"**
- **Why it happens:** Client name wasn't mentioned in the call
- **How to fix:** This is normal. The field will show "Not mentioned" or empty. Add a fallback value.


---

#### 📊 Step 2: Parse JSON (Client Intelligence)

**What It Does:**

Converts the AI's JSON response into structured data fields that Make.com can work with.


**Output Fields:**

| Field | Data Type | Description |
|---|---|---|
| `clientName` | Text | Client's name or company |
| `painPoints[]` | Array | List of problems/challenges |
| `statedGoals[]` | Array | List of desired outcomes |
| `objections[]` | Array | Concerns or hesitations |
| `transformationOpportunities` | Text | How you can help |
| `nextSteps[]` | Array | Action items to take |
| `engagementLevel` | Number | 1-10 rating |
| `keyQuotes[]` | Array | Memorable client quotes |
| `decisionTimeline` | Text | When they need to decide |


**Common Errors & Solutions:**

❌ **Error: "Cannot parse empty response"**
- **Why it happens:** OpenAI didn't return any data
- **How to fix:** Add error handling to check if response exists before parsing


---

#### ♻️ Step 3: Basic Feeder (Client Intelligence)

**What It Does:**

Unlike the content ideas route (which creates multiple entries), this route creates ONE comprehensive client intelligence entry per call.

However, the Feeder is still needed to properly format arrays (like pain points, goals, next steps) for Notion.


---

#### 📝 Step 4: Create Notion Page (Client Intelligence Database)

**What It Does:**

Creates a detailed client intelligence page in Notion with all the strategic insights extracted from the call.

This becomes your CRM-style record for tracking client relationships and follow-ups.


**Field Mappings:**

Suggested Notion database structure:

| Notion Field | Field Type | Mapped Data | Source |
|---|---|---|---|
| **Client Name** | Title | Client name | `{{12.clientName}}` |
| **Call Reference** | Relation | Link to original call page | Reference to Module 2 |
| **Pain Points** | Multi-line text | Bullet list of problems | `{{12.painPoints}}` |
| **Stated Goals** | Multi-line text | Bullet list of goals | `{{12.statedGoals}}` |
| **Objections** | Multi-line text | Concerns raised | `{{12.objections}}` |
| **Transformation Opportunities** | Text | How you can help | `{{12.transformationOpportunities}}` |
| **Next Steps** | Multi-line text | Action items | `{{12.nextSteps}}` |
| **Engagement Level** | Number | 1-10 rating | `{{12.engagementLevel}}` |
| **Key Quotes** | Multi-line text | Memorable quotes | `{{12.keyQuotes}}` |
| **Decision Timeline** | Date or Text | When they'll decide | `{{12.decisionTimeline}}` |
| **Call Date** | Date | When call happened | `{{1.recording_start_time}}` |


**Common Errors & Solutions:**

❌ **Error: "Cannot create relation"**
- **Why it happens:** The relation field isn't properly connected to the source database
- **How to fix:** In Notion, verify the relation is set up between Client Intelligence DB and Calls DB

❌ **Error: "Multi-line text too long"**
- **Why it happens:** Notion has character limits for certain field types
- **How to fix:** Use the `substring()` function to trim text: `{{substring(12.painPoints; 0; 2000)}}`


---

### ✅ Route 3: Task Extraction

This route automatically creates actionable to-do items from your calls.


---

#### 🤖 Step 1: OpenAI GPT-4 Mini (Task Extraction)

**What It Does:**

This AI module acts as your personal task manager. It reads through the call transcript and identifies explicit action items that were mentioned or agreed upon.

Think of it as someone listening to your meeting and writing down every "we should..." or "I'll send you..." statement.


**AI Prompt Breakdown:**

**System Role:** "You are a task extraction assistant helping teams convert call discussions into actionable tasks."

**Extraction Rules:**

The AI follows strict rules:
- Only extract EXPLICIT action items mentioned in the call
- Don't infer or assume tasks that weren't discussed
- Set realistic due dates based on urgency
- Assign priority based on importance discussed
- Provide context so anyone can understand the task


**Task Fields Extracted:**

For each action item, the AI extracts:

1. **Task Name:** Clear, actionable description (5-10 words)
   - Example: "Send pricing proposal to Sarah"

2. **Status:** Always set to "To Do" (you'll update this later)

3. **Priority:** Urgent, High, Medium, or Low
   - Based on how the task was discussed in the call

4. **Due Date:** Suggested completion date in YYYY-MM-DD format
   - Example: "2025-01-20"

5. **Task Type:** Category of task
   - Options: Follow-up, Content Creation, Implementation, Research, Client Deliverable

6. **Time Estimate:** Estimated hours to complete (number)
   - Example: 2 (means 2 hours)

7. **Context:** Why this task matters (2-3 sentences)
   - Example: "Client needs proposal before board meeting on Friday. Include custom pricing for enterprise plan."


**Field Mappings:**

| Input Field | Source | Mapping |
|---|---|---|
| Call Title | Fathom webhook | `{{1.meeting_title}}` |
| Transcript | Fathom summary (formatted) | `{{1.default_summary.markdown_formatted}}` |


**Model Settings:**

- **AI Model:** GPT-4o-mini
- **Max Tokens:** 4,096
- **Temperature:** 1
- **Response Format:** JSON array


**Common Errors & Solutions:**

❌ **Error: "Invalid date format"**
- **Why it happens:** AI returned due date in wrong format (like "next week" instead of "2025-01-20")
- **How to fix:** Improve AI prompt to enforce YYYY-MM-DD format, or add date parsing function

❌ **Error: "No tasks extracted"**
- **Why it happens:** No explicit action items were mentioned in the call
- **How to fix:** This is normal for some calls. The route will complete without creating tasks.

❌ **Error: "Priority value not recognized"**
- **Why it happens:** AI returned a priority not in your allowed values
- **How to fix:** Restrict AI to exact values: "Return Priority as exactly one of: Urgent, High, Medium, Low"


---

#### 📊 Step 2: Parse JSON (Tasks)

**What It Does:**

Converts the AI's task list from text format into structured data that Make.com can process.


**Output Fields:**

| Field | Data Type | Description |
|---|---|---|
| `Task Name` | Text | Name of the action item |
| `Status` | Text | Always "To Do" |
| `Priority` | Text | Urgent / High / Medium / Low |
| `Due Date` | Date | YYYY-MM-DD format |
| `Task Type` | Text | Category of task |
| `Time Estimate` | Number | Hours to complete |
| `Context` | Text | Why it matters |


**Common Errors & Solutions:**

❌ **Error: "Failed to parse JSON"**
- **Why it happens:** AI response wasn't valid JSON
- **How to fix:** Add error handling or improve AI instructions for JSON formatting


---

#### ♻️ Step 3: Basic Feeder (Tasks)

**What It Does:**

The AI returns multiple tasks in a single array. The Feeder splits that array so each task becomes a separate Notion entry.

**Example:**

```
AI returns: [Task 1, Task 2, Task 3, Task 4]

Feeder splits into:
→ Task 1 → Creates Notion page
→ Task 2 → Creates Notion page
→ Task 3 → Creates Notion page
→ Task 4 → Creates Notion page
```


**Common Errors & Solutions:**

❌ **Error: "Empty array to iterate"**
- **Why it happens:** No tasks were found in the call
- **How to fix:** This is normal. The route will end gracefully without errors.


---

#### 📝 Step 4: Create Notion Page (Tasks Database)

**What It Does:**

Creates individual task pages in your Notion tasks database for each action item identified by the AI.

This gives you an instant to-do list from every call, ready to track and complete.


**Field Mappings:**

Suggested Notion database structure:

| Notion Field | Field Type | Mapped Data | Source |
|---|---|---|---|
| **Task Name** | Title | Action item description | `{{16.Task Name}}` |
| **Status** | Select | Current status | `{{16.Status}}` (defaults to "To Do") |
| **Priority** | Select | Urgency level | `{{16.Priority}}` |
| **Due Date** | Date | Completion deadline | `{{16.Due Date}}` |
| **Task Type** | Select | Category | `{{16.Task Type}}` |
| **Time Estimate** | Number | Hours needed | `{{16.Time Estimate}}` |
| **Context** | Text | Why it matters | `{{16.Context}}` |
| **Source Call** | Relation | Link to original call | Reference to Module 2 |
| **Created Date** | Date | When task was created | `{{now}}` |


**Notion Select Field Values:**

Make sure your Notion database has these options:

**Status options:**
- To Do
- In Progress
- Blocked
- Complete

**Priority options:**
- Urgent
- High
- Medium
- Low

**Task Type options:**
- Follow-up
- Content Creation
- Implementation
- Research
- Client Deliverable


**Common Errors & Solutions:**

❌ **Error: "Invalid select option"**
- **Why it happens:** The AI returned a value not in your Notion select field
- **How to fix:** Add all possible values to your Notion select field, or create a mapping table

❌ **Error: "Relation not created"**
- **Why it happens:** Can't link task to source call
- **How to fix:** Verify the relation field exists and points to the correct database

❌ **Error: "Invalid date"**
- **Why it happens:** Due date format is incorrect
- **How to fix:** Add date formatting: `{{formatDate(16.Due Date; "YYYY-MM-DD")}}`


---

## 🎨 Notion Database Setup

To make this automation work perfectly, you need to set up your Notion databases with the correct structure.


### Database 1: Calls Master Database

This is your main database that stores all call recordings.

**Required Fields:**

| Field Name | Field Type | Description |
|---|---|---|
| Call Title | Title | Name of the meeting |
| Call Date | Date | When the call happened (start & end) |
| AI Summary | Text (Rich Text) | Fathom's AI summary (max 2000 chars) |
| Processing Status | Select | Queued, Processing, Complete, Failed |
| Recording URL | URL | Link to Fathom recording |
| Raw Transcript | Text (Rich Text) | Full transcript |
| Call Type | Select | Discovery Call, Strategy Session, etc. |
| Key Insights | Text (Rich Text) | Manual notes or highlights |


**Select Field Options:**

**Processing Status:**
- Queued
- Processing
- Complete
- Failed

**Call Type:**
- Client Check-in
- Strategy Session
- Discovery Call
- Implementation Review


---

### Database 2: Content Ideas Database

Stores viral content opportunities extracted from calls.

**Required Fields:**

| Field Name | Field Type | Description |
|---|---|---|
| Title | Title | Content idea title |
| Content Type | Select | Post, Newsletter, Carousel, Thread, Video |
| Hook | Text | Attention-grabbing opening line |
| Core Concept | Text | Main teaching point |
| Key Points | Text | Bullet points to cover |
| Why It Resonates | Text | Audience connection |
| Platform Fit | Multi-select | Instagram, LinkedIn, Twitter, etc. |
| Priority Score | Number | 1-10 virality score |
| Source Call | Relation | Links to Calls Database |
| Status | Select | Idea, In Progress, Published |
| Created Date | Date | Auto-populated |


**Select Field Options:**

**Content Type:**
- Social Media Post
- Newsletter
- Carousel
- Thread
- Video Script

**Platform Fit (Multi-select):**
- Instagram
- LinkedIn
- Twitter
- YouTube
- TikTok
- Facebook

**Status:**
- Idea
- In Progress
- Published
- Archived


---

### Database 3: Client Intelligence Database

Tracks strategic insights about clients and opportunities.

**Required Fields:**

| Field Name | Field Type | Description |
|---|---|---|
| Client Name | Title | Name or company |
| Call Reference | Relation | Links to Calls Database |
| Pain Points | Text | Problems client is facing |
| Stated Goals | Text | What client wants to achieve |
| Objections | Text | Concerns or hesitations |
| Transformation Opportunities | Text | How you can help |
| Next Steps | Text | Action items to take |
| Engagement Level | Number | 1-10 rating |
| Key Quotes | Text | Memorable quotes |
| Decision Timeline | Text or Date | When they'll decide |
| Call Date | Date | When call happened |
| Follow-up Status | Select | No Contact, Reached Out, Proposal Sent, etc. |


**Select Field Options:**

**Follow-up Status:**
- No Contact
- Reached Out
- Proposal Sent
- Negotiating
- Closed Won
- Closed Lost


---

### Database 4: Tasks Database

Your automated task tracker from calls.

**Required Fields:**

| Field Name | Field Type | Description |
|---|---|---|
| Task Name | Title | Action item description |
| Status | Select | To Do, In Progress, Blocked, Complete |
| Priority | Select | Urgent, High, Medium, Low |
| Due Date | Date | Completion deadline |
| Task Type | Select | Follow-up, Content, Implementation, etc. |
| Time Estimate | Number | Hours needed |
| Context | Text | Why this task matters |
| Source Call | Relation | Links to Calls Database |
| Created Date | Date | When task was created |
| Completed Date | Date | When marked complete |
| Assigned To | Person | Who's responsible |


---

## ⚙️ Setup Instructions

Follow these steps to get your automation running.


### Step 1: Import the Blueprint to Make.com

1. Log into your Make.com account

2. Click **"Scenarios"** in the left sidebar

3. Click **"Create a new scenario"**

4. Click the **three dots** (⋮) in the top right

5. Select **"Import Blueprint"**

6. Upload the file: `Integration Fathom.blueprint.json`

7. Click **"Save"**


---

### Step 2: Connect Your Accounts

The automation needs connections to three services:


#### Fathom Connection:

1. Click on the **first module** (Fathom Webhook)

2. Click **"Create a webhook"**

3. Copy the webhook URL

4. Go to **Fathom** → **Settings** → **Integrations** → **Webhooks**

5. Paste the webhook URL

6. Select trigger: **"When recording is ready"**

7. Save


#### Notion Connection:

1. Click on **any Notion module**

2. Click **"Add"** next to Connection

3. Click **"Sign in with Notion"**

4. Select your workspace

5. Grant access to the databases you created

6. Click **"Allow access"**


#### OpenAI Connection:

1. Click on **any OpenAI module**

2. Click **"Add"** next to Connection

3. Go to **OpenAI** → **API Keys** → **Create new key**

4. Copy the API key

5. Paste into Make.com

6. Click **"Save"**


---

### Step 3: Configure Database IDs

You need to tell the automation which Notion databases to use.


1. Open your **Notion Calls Database**

2. Copy the URL from your browser
   - Example: `https://notion.so/6a1467612d6d4afe82370bf0e4fa1109?v=...`

3. The database ID is the 32-character code: `6a1467612d6d4afe82370bf0e4fa1109`

4. In Make.com, find **Module 4** (Create Notion Page)

5. Click the module → Paste your database ID in the **"Database ID"** field

6. Repeat for:
   - **Content Ideas Database** (used in Route 1, Module 9)
   - **Client Intelligence Database** (used in Route 2, Module 13)
   - **Tasks Database** (used in Route 3, Module 17)


---

### Step 4: Test the Automation

1. In Make.com, click **"Run once"** at the bottom

2. The scenario will wait for a webhook trigger

3. Go to **Fathom** and record a test call (or use a past recording)

4. Trigger the webhook manually from Fathom settings

5. Watch the automation run in Make.com

6. Check your Notion databases for the new entries

7. If everything works, click **"Scheduling"** → **"On"** to activate


---

## 🚨 Common Errors & Troubleshooting


### General Errors


❌ **Error: "Scenario is not active"**
- **Solution:** Turn on scheduling in the scenario settings


❌ **Error: "Connection expired"**
- **Solution:** Reconnect the expired service (Notion, OpenAI, or Fathom)


❌ **Error: "Insufficient operations"**
- **Solution:** You've used all your Make.com operations for the month. Upgrade or wait for reset.


❌ **Error: "Module timed out"**
- **Solution:** The OpenAI API took too long. Reduce the transcript length or increase timeout settings.


---

### Fathom Webhook Errors


❌ **Error: "Webhook not receiving data"**
- **Why:** Fathom isn't sending notifications
- **Solution:** Verify webhook URL in Fathom settings. Test with a manual trigger.


❌ **Error: "Invalid webhook payload"**
- **Why:** Fathom changed their data structure
- **Solution:** Check what data Fathom is sending. Update field mappings if needed.


---

### Notion Errors


❌ **Error: "Database not found"**
- **Why:** Incorrect database ID or database was deleted
- **Solution:** Verify the database ID is correct. Re-share the database with Make.com integration.


❌ **Error: "Property does not exist"**
- **Why:** Your Notion database is missing a required field
- **Solution:** Add the missing property to your Notion database


❌ **Error: "Invalid property value"**
- **Why:** The data type doesn't match (e.g., trying to put text in a number field)
- **Solution:** Check your field mappings. Use parsing functions like `parseNumber()` or `formatDate()`


❌ **Error: "Integration lacks permission"**
- **Why:** Make.com doesn't have access to the database
- **Solution:** In Notion, click "Share" on the database → Add your Make.com integration


❌ **Error: "Select option not found"**
- **Why:** The value isn't in your Notion select field options
- **Solution:** Add the missing option to your Notion select field


---

### OpenAI Errors


❌ **Error: "Invalid API key"**
- **Why:** API key is wrong or expired
- **Solution:** Generate a new API key in OpenAI and update Make.com connection


❌ **Error: "Insufficient quota"**
- **Why:** You've used all your OpenAI credits
- **Solution:** Add credits to your OpenAI account or upgrade plan


❌ **Error: "Rate limit exceeded"**
- **Why:** Too many API calls too quickly
- **Solution:** Add a 2-3 second delay module before OpenAI calls


❌ **Error: "Context length exceeded"**
- **Why:** Your transcript is too long for the AI model
- **Solution:** Reduce max_tokens or summarize the transcript before sending


❌ **Error: "Invalid JSON in response"**
- **Why:** AI didn't format the response correctly
- **Solution:** Improve the AI prompt to be more explicit about JSON format


---

### Router Errors


❌ **Error: "All routes failed"**
- **Why:** A critical error occurred before the router
- **Solution:** Check modules before the router (Webhook and initial Notion creation)


❌ **Error: "Partial route failure"**
- **Why:** One or two routes failed, but others succeeded
- **Solution:** Identify which route failed and troubleshoot that specific path


---

## 💡 Pro Tips & Best Practices


### Optimization Tips


**🚀 Speed Up Processing:**
- Use GPT-4o-mini instead of GPT-4 (faster and cheaper)
- Reduce max_tokens if responses are too long
- Run routes in parallel (already configured)


**💰 Reduce Costs:**
- Monitor your OpenAI usage in their dashboard
- Set token limits to avoid expensive calls
- Use Fathom's summary instead of full transcript when possible


**📊 Better AI Results:**
- Review AI outputs and refine prompts over time
- Add examples in the AI prompt for consistency
- Use temperature 0.7 for more predictable outputs (vs 1.0 for creative)


**🎯 Customize for Your Business:**
- Modify the Content Type options to match your content strategy
- Adjust Task Type categories to fit your workflow
- Change Priority scoring to match your criteria


---

### Monitoring & Maintenance


**Check These Weekly:**
- ✅ Make.com scenario execution history for errors
- ✅ OpenAI usage and costs
- ✅ Notion database entries are complete and accurate
- ✅ Fathom webhook is still connected


**Monthly Review:**
- 📊 Analyze which content ideas got published
- 📊 Review client intelligence insights for trends
- 📊 Check task completion rates
- 📊 Optimize AI prompts based on output quality


---

### Scaling Up


**When You Grow:**

- **Multiple team members:** Add assignment logic to route tasks to specific people
- **More call types:** Create additional router paths for different call categories
- **International clients:** Add timezone conversion for call dates
- **Multiple workspaces:** Duplicate the scenario for each Notion workspace


---

## 📚 Understanding the Data Flow


### Complete Journey of Your Data


**🎙️ The Call Happens**
- You have a client call using Fathom
- Fathom records and transcribes everything
- AI summary is generated by Fathom


**📡 Webhook Trigger**
- Fathom sends notification to Make.com
- All call data is transmitted instantly
- Automation starts automatically


**📝 Initial Notion Entry**
- Basic call information is saved
- Creates a "master record" for this call
- This entry will be referenced by all other databases


**🔀 Router Splits the Data**
- Three parallel processes begin
- Each path runs independently
- All paths reference the same source call


**🤖 AI Processing (3 Paths)**

**Path 1 - Content:**
- AI reads transcript
- Identifies 3 content opportunities
- Structures as JSON
- Each idea becomes separate Notion page

**Path 2 - Intelligence:**
- AI analyzes conversation
- Extracts strategic insights
- Single comprehensive page created
- Links back to source call

**Path 3 - Tasks:**
- AI finds action items
- Assigns priority and due dates
- Each task becomes separate page
- All tasks link to source call


**✅ Final Result**
- 1 call master entry
- 3 content idea pages
- 1 client intelligence page
- Multiple task pages (varies per call)
- All interconnected via relations


---

## 🎓 Frequently Asked Questions


### General Questions


**Q: How much does this cost to run?**
A: Costs include:
- Make.com: Free for 1,000 operations/month, then $9-29/month
- OpenAI: ~$0.50-2.00 per call (depending on length)
- Notion: Free for personal use
- Fathom: Varies by plan

Total monthly cost for 20 calls: ~$20-50


**Q: How long does it take to process a call?**
A: Typically 1-2 minutes from when Fathom finishes recording. The three AI routes run in parallel, so it's very fast.


**Q: Can I customize the AI prompts?**
A: Absolutely! Click on any OpenAI module and edit the "messages" field. Tailor the prompts to your industry, tone, and needs.


**Q: What if I don't want all three routes?**
A: You can disable routes you don't need. Click the route in the Router module and delete it.


**Q: Can I add more routes?**
A: Yes! Click the Router module → Add another route → Build your custom AI analysis path.


---

### Technical Questions


**Q: Why does the AI sometimes return invalid JSON?**
A: AI models occasionally make formatting mistakes. Add error handling or use OpenAI's structured output mode (if available).


**Q: How do I handle really long transcripts?**
A: Use Fathom's summary instead of full transcript, or add a text truncation function: `{{substring(1.transcript; 0; 15000)}}`


**Q: Can this work with other meeting tools besides Fathom?**
A: Yes! Replace the Fathom webhook with webhooks from Zoom, Google Meet (via integrations), or upload transcripts manually.


**Q: How do I back up my automation?**
A: In Make.com, click the three dots → Export Blueprint. Save the JSON file safely.


**Q: Can I use this for languages other than English?**
A: Yes! OpenAI supports multiple languages. Just ensure your Fathom transcripts are in the target language.


---

### Notion Questions


**Q: Do I need Notion Plus or can I use the free plan?**
A: The free Notion plan works perfectly for this automation.


**Q: Can I use different database structures?**
A: Yes, but you'll need to update the field mappings in Make.com to match your custom fields.


**Q: How do I link tasks back to the original call?**
A: Use a Relation field in your Tasks database that points to the Calls database. Map it to the page ID from Module 2.


**Q: Can I use this across multiple Notion workspaces?**
A: You'll need to duplicate the scenario in Make.com for each workspace and update the connections.


---

## 🎉 Conclusion

Congratulations! You now have a fully automated system that transforms every client call into:

✅ Viral content ideas ready to publish

✅ Strategic client insights for better relationships

✅ Actionable tasks so nothing falls through the cracks


This automation runs 24/7 in the background, giving you back hours of time every week while ensuring you never miss an opportunity hidden in your conversations.


### What to Do Next:

1. **Test thoroughly:** Run a few test calls to ensure everything works

2. **Customize the prompts:** Tailor the AI instructions to your specific business

3. **Build your Notion workflow:** Create views, filters, and dashboards to maximize the value

4. **Monitor and optimize:** Review outputs weekly and refine the automation

5. **Share with your team:** If you have team members, train them on the system


### Need Help?

If you run into issues:
- Check the troubleshooting section above
- Review Make.com execution history for error details
- Verify all connections are active
- Test each module individually to isolate issues


---

**Remember:** This automation is a powerful tool, but the real magic happens when you actually USE the insights it generates. Schedule time weekly to:
- Review content ideas and create posts
- Follow up on client intelligence insights
- Complete the tasks it identifies


Happy automating! 🚀
