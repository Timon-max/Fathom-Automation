# Notion Dashboard Views Configuration

**Create a powerful command center for call intelligence and content management**

---

## Overview

Transform your three databases into an actionable dashboard with strategic views for:
- Monitoring call processing and content pipeline
- Tracking client health and relationship signals
- Finding high-value content ready to publish
- Identifying clients who need attention

---

## Dashboard Page Structure

### Create Central Command Center Page

1. Create new page: **"Call Intelligence Command Center"**
2. Add icon: 🎯
3. Add cover image (optional)
4. Structure with sections:

```
🎯 Call Intelligence Command Center

[Quick Stats Section]
[Recent Calls Section]
[Content Pipeline Section]
[Client Health Monitor Section]
[High-Priority Content Section]
[Clients Needing Attention Section]
```

---

## Section 1: Quick Stats

**Purpose:** At-a-glance metrics and KPIs

### Setup

1. Add **Heading 1:** "📊 Quick Stats"
2. Add **3-column layout** (use /columns command)
3. In each column, add a **database view**:

**Column 1: Total Calls This Month**
- Database: Call Intelligence Hub
- View type: Table (minimal)
- Filter: `Call Date` → `is within` → `This month`
- Properties shown: None (just count)
- Display: Show inline
- At bottom: Will show "X items"

**Column 2: Content Ideas Generated**
- Database: Content Ideas Bank
- View type: Table (minimal)
- Filter: `Created` → `is within` → `This month`
- Properties shown: None
- Display: Show inline

**Column 3: Clients at Risk**
- Database: Client Intelligence
- View type: Table (minimal)
- Filter: `Current Sentiment` → `is` → `🔴 At Risk`
- Properties shown: None
- Display: Show inline

### Alternative: Use Text Blocks

If you want custom calculations:

```
📊 This Month
• Calls Processed: [manually update weekly]
• Content Ideas: [manually update weekly]
• Avg Ideas/Call: [calculate]
• Clients at Risk: [from database]
```

---

## Section 2: Recent Calls

**Purpose:** Monitor call processing and access recent recordings

### View Configuration

1. Add **Heading 2:** "🎙️ Recent Calls"
2. Add **Linked database view:**
   - Select: Call Intelligence Hub
   - Click: "Create linked database"
   - View type: Gallery

**View Settings:**

**Name:** Recent Calls (Last 30 Days)

**Filter:**
- `Call Date` → `is within` → `the past 30 days`

**Sort:**
- `Call Date` → Descending (newest first)

**Layout:**
- Size: Medium
- Fit image: Cover

**Card Preview:**
- Title: Call Title
- Properties visible on card:
  - Call Date (show as: Friendly)
  - Processing Status (icon visible)
  - Sentiment Score (icon visible)
  - Content Ideas Generated (number)
  - Duration (number)

**Card Cover:** None (or use custom icons based on status)

**Properties shown in page:** All (when you open the call)

---

## Section 3: Content Pipeline

**Purpose:** Kanban board to manage content from idea → published

### View Configuration

1. Add **Heading 2:** "📝 Content Pipeline"
2. Add **Linked database view:**
   - Select: Content Ideas Bank
   - View type: Board

**View Settings:**

**Name:** Content Pipeline

**Group by:** Status

**Sub-group:** None

**Filter:** None (show all)

**Sort within groups:**
- `Priority Score` → Descending
- `Created` → Descending

**Board Properties:**

**Hidden groups:** None (show all statuses)

**Card Properties visible:**
- Content Type (icon)
- Priority Score (number with color coding)
- Platform Fit (pills)
- Client Name (text)
- Created (relative time)

**Card preview size:** Small

### Board Columns (Left to Right)

```
[🆕 New Idea] → [✍️ Ready to Draft] → [📝 Drafted] → [📅 Scheduled] → [✅ Published]
```

**Workflow:**
1. AI populates "New Idea" or "Ready to Draft"
2. Manual: Review and move to "Drafted"
3. Manual: Schedule and move to "Scheduled"
4. Manual: Publish and move to "Published" (add URL)

---

## Section 4: Client Health Monitor

**Purpose:** Visual client health dashboard with sentiment grouping

### View Configuration

1. Add **Heading 2:** "💚 Client Health Monitor"
2. Add **Linked database view:**
   - Select: Client Intelligence
   - View type: Gallery

**View Settings:**

**Name:** Client Health Dashboard

**Group by:** Current Sentiment

**Sort within groups:**
- `Last Call Date` → Descending

**Filter:** None

**Layout:**
- Size: Small
- Fit image: None

**Card Properties visible:**
- Last Call Date (relative)
- Days Since Contact (formula)
- Total Calls (rollup, if configured)
- Account Status (icon)

**Groups (collapsible):**
```
🟢 Improving [X clients]
🔵 Stable [X clients]
🟡 Watch [X clients]
🔴 At Risk [X clients]
```

**Group Colors:**
- Improving: Green background
- Stable: Blue background
- Watch: Yellow background
- At Risk: Red background

---

## Section 5: High-Priority Content Ready

**Purpose:** Quick access to AI-drafted content ready for review

### View Configuration

1. Add **Heading 2:** "⭐ High-Priority Content (Ready to Post)"
2. Add **Linked database view:**
   - Select: Content Ideas Bank
   - View type: Table

**View Settings:**

**Name:** High Priority - Ready to Review

**Filter (combine with AND):**
- `Priority Score` → `Greater than or equal` → `8`
- `Status` → `is any of` → `Ready to Draft, Drafted`
- `Polished Draft` → `is not empty`

**Sort:**
- `Priority Score` → Descending
- `Created` → Descending

**Properties shown (in order):**
1. Idea Title (Title)
2. Content Type (Select with icon)
3. Priority Score (Number)
4. Platform Fit (Multi-select)
5. Hook/Opening (Text - preview)
6. Status (Select)
7. Created (Date - relative)

**Row height:** Default

**Wrap cells:** No

**This view should be small (10-15 items max)** - these are your quickest wins

---

## Section 6: Clients Needing Attention

**Purpose:** Alert for clients requiring immediate action

### View Configuration

1. Add **Heading 2:** "⚠️ Clients Needing Attention"
2. Add **Linked database view:**
   - Select: Client Intelligence
   - View type: Table

**View Settings:**

**Name:** Action Required

**Filter (combine with OR):**
- `Current Sentiment` → `is any of` → `Watch, At Risk`
- `Days Since Contact` → `Greater than` → `30`
- `Red Flags` → `is not empty`

**Sort:**
- `Current Sentiment` → Custom (At Risk first, then Watch, then Stable)
- `Last Call Date` → Ascending (oldest first = most urgent)

**Properties shown:**
1. Client Name (Title)
2. Current Sentiment (Select with icon)
3. Last Call Date (Date)
4. Days Since Contact (Formula - red if >30)
5. Red Flags (Text - preview)
6. Next Call Date (Date)
7. Account Status (Select)

**Conditional formatting (if available):**
- Days Since Contact > 30: Red text
- Current Sentiment = At Risk: Red row background

---

## Additional Useful Views

### View: All Content by Client

**Database:** Content Ideas Bank
**View type:** Table
**Group by:** Client Name
**Sort:** Created → Descending

**Use case:** Review all content generated for specific client

---

### View: Content by Type

**Database:** Content Ideas Bank
**View type:** Board
**Group by:** Content Type
**Filter:** Status → Not Published

**Use case:** See what formats you have available (LinkedIn posts, newsletters, etc.)

---

### View: Failed Processing

**Database:** Call Intelligence Hub
**View type:** Table
**Filter:** Processing Status → is → 🔴 Failed
**Sort:** Call Date → Descending

**Use case:** Troubleshoot automation errors

---

### View: Content Performance Tracking

**Database:** Content Ideas Bank
**View type:** Table
**Filter:** Status → is → ✅ Published
**Properties:**
- Published URL
- Content Type
- Platform Fit
- Priority Score
- Created (as "Publish Date")

**Manual enhancement:**
- Add property: "Engagement Score" (number, manual entry)
- Add property: "Views/Impressions" (number, manual entry)

**Use case:** Track what content performs best to tune AI prioritization

---

## Dashboard Customization Tips

### Visual Hierarchy

**Use heading sizes strategically:**
- Heading 1: Main dashboard title
- Heading 2: Major sections
- Heading 3: Subsections within major sections

**Color coding:**
- Use callout blocks for important alerts
- Color-code status options in databases
- Use column backgrounds to separate sections

---

### Interactive Elements

**Add buttons for common actions:**

**Button: "📞 Log Call Manually"**
- Action: Add page to Call Intelligence Hub
- Template: Pre-fill Processing Status as "Complete" (manual entry)

**Button: "💡 Add Content Idea"**
- Action: Add page to Content Ideas Bank
- Template: Pre-fill Status as "New Idea"

**Button: "➕ Add New Client"**
- Action: Add page to Client Intelligence
- Template: Pre-fill Account Status as "Onboarding"

---

### Quick Links Section

Add at top of dashboard:

```
Quick Actions:
• [Fathom Recordings](https://app.fathom.video/) - View all recordings
• [Make.com Scenario](https://make.com/scenarios/...) - Monitor automation
• [OpenRouter Dashboard](https://openrouter.ai/activity) - Check API usage
• [Prompts Config](notion link to prompts) - AI prompt versions
```

---

## Mobile Optimization

**For mobile access:**

1. **Simplify views:**
   - Use list view instead of gallery on mobile
   - Show fewer properties (3-4 max)
   - Use icons instead of text where possible

2. **Priority order:**
   - Put most important views at top
   - Collapse less-used sections by default

3. **Use Notion mobile app:**
   - Create shortcut to Command Center page
   - Enable offline access for key databases

---

## Dashboard Maintenance

### Weekly Review

**Every Monday morning (5 minutes):**

1. Check **Recent Calls** - any failed processing?
2. Review **Clients Needing Attention** - schedule calls
3. Scan **High-Priority Content** - move 2-3 to drafted
4. Update **Quick Stats** if not automatic

### Monthly Audit

**First Monday of month (15 minutes):**

1. Archive old content (published >60 days ago)
2. Review client sentiment trends (improving vs declining)
3. Check content performance (if tracking manually)
4. Update any static metrics or goals

---

## Advanced: Dashboard Metrics

### Create Calculated Metrics

**Average Content Ideas per Call:**

1. Create new database view: Calls This Month (table)
2. Properties: Call Title, Content Ideas Generated
3. Manually calculate average
4. Display in callout block

**Sentiment Distribution:**

1. Create view: Clients by Sentiment (gallery)
2. Group by Current Sentiment
3. Each group shows count
4. Monitor month-over-month changes

**Content Velocity:**

1. Track: Ideas generated → Drafted → Published
2. Calculate: Days from idea to published
3. Goal: <14 days for high-priority content

---

## Dashboard Templates

### Template 1: Executive Dashboard (Simple)

```
🎯 Call Intelligence Dashboard

📊 This Week
• 5 calls processed
• 37 content ideas generated
• 2 clients need follow-up

🎙️ Recent Calls
[Gallery view - last 7 days]

📝 Ready to Post
[Table view - Priority 8+, has draft]

⚠️ Action Required
[Table view - Clients at risk or >30 days]
```

---

### Template 2: Content Operations Dashboard

```
📝 Content Production Hub

Pipeline
[Board view - Status groups]

By Client
[Table view - Grouped by client]

Performance
[Table view - Published content with metrics]

Ideas Backlog
[Table view - Priority 6-7, not yet drafted]
```

---

### Template 3: Client Success Dashboard

```
💚 Client Health Command Center

Risk Alert
[Gallery view - Watch + At Risk clients]

All Clients
[Table view - All clients with key metrics]

Upsell Pipeline
[Custom view showing clients with opportunities]

Win Tracking
[View showing recent wins by client]
```

---

## Integration with Make.com

**Link dashboard to automation:**

1. Add Make.com scenario URL to dashboard
2. Embed scenario execution stats (if available)
3. Link to Make.com execution history

**Monitor operations:**
- Track operations used per week
- Alert when approaching limit (900/1000)
- Plan upgrade timing

---

## Sharing & Permissions

**Team access:**

**Role: Admin**
- Full access to all databases
- Can edit views and properties
- Can modify automation settings

**Role: Content Manager**
- Read/write: Content Ideas Bank
- Read only: Call Intelligence Hub, Client Intelligence
- Can move ideas through pipeline

**Role: Executive**
- Read only: All databases
- Focus: Dashboard views only
- Mobile app access for on-the-go reviews

---

## Common View Issues

**Problem: View shows no data**
- Check filters (too restrictive?)
- Verify data exists in source database
- Confirm date ranges are correct

**Problem: Grouped view shows wrong counts**
- Re-sync database (reload page)
- Check grouping property has values
- Verify no empty/null values causing miscounts

**Problem: Slow loading times**
- Reduce number of visible properties
- Use filters to limit items shown
- Consider splitting into multiple smaller views

---

## Next Steps

After setting up dashboard:

1. ✅ Test each view with sample data
2. ✅ Customize for your workflow
3. ✅ Set up mobile access
4. ✅ Schedule weekly dashboard review time
5. → Start processing calls and building content library!

---

## Dashboard Evolution

**Phase 1 (MVP):**
- Basic views (recent calls, content pipeline, client health)
- Manual metrics tracking
- Weekly reviews

**Phase 2 (Month 2):**
- Add performance tracking
- Integrate task management
- Automated reports (via Make.com)

**Phase 3 (Month 3+):**
- Custom charts (using Notion integrations)
- Predictive client health scoring
- Content calendar integration
- ROI tracking dashboard
