# OpenRouter AI Prompts

**Version-controlled prompts for all AI processing modules**

This file contains all AI prompts used in the Make.com workflow. Update here, then copy to Make.com modules.

---

## Prompt Version History

- **v1.0** (2024-01-11): Initial prompts
- **v1.1** (TBD): Refinements based on testing

---

## Content Ideas Generator Prompt

**Module:** [5A] HTTP Request - Generate Content Ideas
**Model:** `openai/gpt-4o-mini`
**Temperature:** 0.7
**Cost per call:** ~$0.10-0.15

### System Prompt

```
You are a content strategist helping coaches create viral LinkedIn content and newsletter material from their client calls. Your output must be valid JSON only.
```

### User Prompt

```
Analyze this coaching call transcript and extract 5-10 high-value content ideas.

Call Title: {{1.title}}

Transcript:
{{2.transcript}}

For each content idea, provide:
1. title: Attention-grabbing title (5-8 words)
2. contentType: LinkedIn Post, Newsletter, Carousel, Thread, or Video Script
3. hook: Opening line that stops the scroll (10-15 words)
4. coreConcept: The main insight or teaching (2-3 sentences)
5. keyPoints: 3-5 bullet points to cover (as array)
6. whyItResonates: Why this connects with the coach's audience (1-2 sentences)
7. platformFit: Best platforms for this content (array of strings)
8. priorityScore: 1-10 based on virality potential and relevance

Rules:
- Focus on client pain points, transformations, and aha moments from the call
- Make hooks specific and curiosity-driven
- Prioritize tactical, actionable content over theory
- Score based on: clarity of value (40%), audience fit (30%), viral potential (30%)

Return ONLY a valid JSON array with no markdown code blocks:
[{"title": "...", "contentType": "...", "hook": "...", "coreConcept": "...", "keyPoints": [...], "whyItResonates": "...", "platformFit": [...], "priorityScore": 8}]
```

### Expected Output Format

```json
[
  {
    "title": "Why Most Coaches Struggle with Time Management",
    "contentType": "LinkedIn Post",
    "hook": "I asked 47 coaches about their biggest challenge. 82% said the same thing.",
    "coreConcept": "Coaches often fail at time management not because they lack skills, but because they don't protect their energy boundaries with clients.",
    "keyPoints": [
      "The myth of 24/7 availability",
      "Setting clear office hours saves relationships",
      "How to communicate boundaries without guilt"
    ],
    "whyItResonates": "Every coach struggles with being 'too available' early in their career. This provides a tactical framework to fix it.",
    "platformFit": ["LinkedIn", "Newsletter"],
    "priorityScore": 9
  }
]
```

### Tuning Notes

**If ideas are too generic:**
- Add to rules: "Extract specific examples, numbers, and quotes from the transcript"
- Increase specificity requirement in coreConcept

**If priority scores are all high:**
- Adjust scoring criteria: "Most ideas should score 4-7, only truly exceptional insights score 8-10"

**If getting markdown instead of JSON:**
- Add to prompt: "Do not use markdown code blocks. Do not include ```json. Output raw JSON only."

---

## Content Drafting Prompt

**Module:** [10A] HTTP Request - Draft Content
**Model:** `openai/gpt-4o-mini`
**Temperature:** 0.7
**Cost per draft:** ~$0.15-0.20

### System Prompt

```
You are a professional ghostwriter creating ready-to-post content for coaches.
```

### User Prompt

```
Write a complete {{7A.contentType}} based on this content idea.

Idea Title: {{7A.title}}
Hook: {{7A.hook}}
Core Concept: {{7A.coreConcept}}
Key Points:
{{join(7A.keyPoints; "\n• ")}}

Style Guidelines:
- Conversational, direct tone
- Short paragraphs (2-3 sentences max)
- Lead with the hook to grab attention
- Deliver value fast - no fluff
- End with engagement question or CTA

Length Requirements:
- LinkedIn Post: 150-200 words
- Newsletter: 300-400 words
- Carousel: 8 slides with title + 3-4 bullets each
- Thread: 8-10 tweets
- Video Script: 60-90 second script with timestamps

Output the complete, ready-to-edit draft with no meta-commentary.
```

### Expected Output Examples

**LinkedIn Post:**
```
I asked 47 coaches about their biggest challenge. 82% said the same thing.

Time management.

But here's what I learned: They don't have a time problem. They have a boundary problem.

Most coaches I work with are available 24/7. Slack messages at 9 PM. "Quick calls" on weekends. Emergency sessions on vacation.

They think availability = caring.

It doesn't. It = burnout.

Here's what changed for my clients:

→ Set specific office hours (9-5, no exceptions)
→ Turn off notifications after hours
→ Create a 48-hour response policy for non-emergencies

The surprising result?

Client satisfaction went UP.

Why? Because when you're present, you're FULLY present. Not exhausted. Not resentful. Not checking Slack during dinner.

Boundaries don't push clients away. They protect the relationship.

What's one boundary you need to set this week?
```

**Carousel (example of slide 1-2):**
```
Slide 1:
---
The Boundary Myth
That's Burning You Out
---

Slide 2:
❌ The Myth:
Being available 24/7 shows clients you care

✅ The Truth:
Availability without boundaries leads to:
• Resentment
• Exhaustion
• Lower quality work
• Damaged relationships
```

### Tuning Notes

**If content is too formal:**
- Add to style: "Write like you're talking to a friend at coffee"
- Examples: "Use contractions, sentence fragments, single-sentence paragraphs"

**If content lacks punch:**
- Add: "Lead with surprising stat, bold claim, or question"
- Add: "Use specific numbers and examples, not generalizations"

**If CTAs are weak:**
- Add: "End with thought-provoking question or clear next step"

---

## Client Intelligence Analyzer Prompt

**Module:** [5B] HTTP Request - Analyze Intelligence
**Model:** `anthropic/claude-3.5-sonnet`
**Temperature:** 0.3
**Cost per analysis:** ~$0.25-0.35

### User Prompt

```
Analyze this client call for strategic intelligence and relationship health indicators.

Call Title: {{1.title}}

Transcript:
{{2.transcript}}

Fathom Summary:
{{3.summary}}

Provide a comprehensive analysis with:

1. sentiment: Overall client sentiment classification (choose one: "Improving", "Stable", "Watch", "At Risk")
2. sentimentConfidence: 0-100 (your confidence in the sentiment assessment)
3. sentimentEvidence: Array of 2-3 specific quotes from transcript supporting your assessment
4. painPoints: Array of current challenges, frustrations, or obstacles mentioned (3-5 items)
5. wins: Array of recent successes, progress, or victories celebrated (2-3 items)
6. upsellOpportunities: Array of new needs, expanded scope hints, wish-list items, or "if only we had..." statements (2-3 items)
7. redFlags: Array of warning signs like dissatisfaction, competitor mentions, budget concerns, scope reduction, delayed responses (0-3 items, empty array if none)
8. nextActions: Array of what must happen next to maintain or improve relationship (2-3 items)
9. clientName: Extract the client's name from the call title or transcript

Analysis Guidelines:
- Sentiment "Improving": Positive momentum, expressing gratitude, expanding scope
- Sentiment "Stable": Neutral tone, steady progress, no major concerns
- Sentiment "Watch": Minor concerns, slowing momentum, needs attention
- Sentiment "At Risk": Expressing dissatisfaction, missing meetings, budget concerns
- Look for subtle signals, not just explicit statements
- Upsell opportunities can be implicit ("I wish we could...")
- Red flags include scope creep pushback, payment delays, reduced engagement

Return ONLY valid JSON with no markdown:
{"sentiment": "Stable", "sentimentConfidence": 85, "sentimentEvidence": [...], "painPoints": [...], "wins": [...], "upsellOpportunities": [...], "redFlags": [], "nextActions": [...], "clientName": "..."}
```

### Expected Output Format

```json
{
  "sentiment": "Improving",
  "sentimentConfidence": 92,
  "sentimentEvidence": [
    "The client said: 'This automation saved us 15 hours last week alone'",
    "Proactively asked about expanding to their sales team",
    "Mentioned they've been recommending us to other coaches"
  ],
  "painPoints": [
    "Still struggling with onboarding new team members to the system",
    "Notion database feels overwhelming to navigate",
    "Not sure how to track ROI for their clients"
  ],
  "wins": [
    "Successfully automated their entire client intake process",
    "Closed 3 new clients using content generated from our system",
    "Team adoption went from 30% to 85% in 2 weeks"
  ],
  "upsellOpportunities": [
    "Asked if we could build similar automation for their podcast workflow",
    "Mentioned they wish they had a dashboard to show clients",
    "Expressed interest in white-labeling for their own clients"
  ],
  "redFlags": [],
  "nextActions": [
    "Create simplified onboarding checklist for their team",
    "Schedule walkthrough of dashboard navigation",
    "Send ROI tracking template by end of week"
  ],
  "clientName": "Sarah Johnson"
}
```

### Tuning Notes

**If sentiment is always "Stable":**
- Add: "Be bold in your assessment. Most calls should NOT be exactly neutral."
- Adjust: "Look for momentum direction, not just current state"

**If missing subtle signals:**
- Add: "Pay attention to energy level, question types, and what's NOT being discussed"
- Add: "Lack of enthusiasm about results is a red flag even if not explicitly negative"

**If clientName extraction fails:**
- Add fallback: "If client name is unclear, use 'Client from [call title]'"

---

## Executive Summary Generator Prompt

**Module:** [5C] HTTP Request - Generate Summary
**Model:** `openai/gpt-4o-mini`
**Temperature:** 0.5
**Cost per summary:** ~$0.10-0.15

### System Prompt

```
You are a professional executive assistant creating polished call summaries.
```

### User Prompt

```
Create a professional executive summary of this call in clean Markdown format.

Call Title: {{1.title}}
Date: {{formatDate(1.createdAt; "MMMM DD, YYYY")}}
Duration: {{floor(1.duration / 60)}} minutes

Transcript:
{{2.transcript}}

Fathom Summary:
{{3.summary}}

Format exactly like this:

## 📊 Call Overview
- **Date:** [date]
- **Duration:** [X] minutes
- **Type:** [Client Check-in / Strategy Session / Implementation Review / Discovery Call]

## 🎯 Key Outcomes
[3-5 bullet points of most important decisions, commitments, or insights delivered]

## 💡 Top Insights
[3-4 strategic learnings or client revelations that matter]

## ✅ Actions Captured
- Content Ideas Generated: [count from your analysis]
- Key Follow-up Items: [if applicable]
- Decisions Made: [if applicable]

## 🚀 Next Steps
[3-5 immediate actions with suggested owners and timing]

## ⚠️ Watch Items
[Only include this section if there are concerns, blockers, or risks. Otherwise omit.]

Guidelines:
- Be concise and scannable
- Focus on decisions and actionable items
- Use specific numbers and commitments
- Highlight value delivered
- Keep professional but conversational tone
```

### Expected Output Example

```markdown
## 📊 Call Overview
- **Date:** January 11, 2024
- **Duration:** 42 minutes
- **Type:** Strategy Session

## 🎯 Key Outcomes
- Finalized content calendar structure for Q1 2024
- Decided to focus on LinkedIn + Newsletter (pause Instagram for now)
- Committed to publishing 3x/week: Monday motivation, Wednesday tactical, Friday case study
- Approved new automation workflow for content distribution
- Set KPI: 500 newsletter subscribers by end of Q1

## 💡 Top Insights
- Client's audience engages 3x more with tactical "how-to" content than motivational posts
- Their best-performing content features client transformation stories with specific metrics
- They're sitting on 15+ hours of recorded coaching calls that could become content goldmine
- Main content blocker isn't ideas - it's decision fatigue on what to publish

## ✅ Actions Captured
- Content Ideas Generated: 8 ideas ready to draft
- Implementation Items: 3 workflow automations approved
- Follow-up Tasks: 4 action items assigned

## 🚀 Next Steps
- **By Friday:** Draft 3 content pieces from today's ideas (Owner: [Your Team])
- **By Monday:** Set up content distribution automation (Owner: [Your Team])
- **By Next Call:** Client to send links to 5 recorded coaching calls for content mining
- **Ongoing:** Weekly content review meeting moving to Tuesdays at 10 AM

## ⚠️ Watch Items
- Client mentioned budget review happening next month - may impact Q2 scope
- Team member turnover could slow content approval process
```

### Tuning Notes

**If summaries are too long:**
- Add: "Keep each bullet to 1-2 lines max. Be ruthlessly concise."
- Limit: "Maximum 5 bullets per section"

**If missing key decisions:**
- Add: "Prioritize commitments, decisions, and specific numbers over general discussion"

**If tone is too formal:**
- Add: "Write for busy executives who scan. Use active voice and specific language."

---

## Cost Optimization Tips

**Reduce API costs without sacrificing quality:**

1. **Use cheaper models for simple tasks:**
   - Executive Summary: `openai/gpt-3.5-turbo` ($0.05/call vs $0.15)
   - Only use GPT-4 where reasoning depth matters

2. **Reduce token usage:**
   - If transcripts are >10k words, summarize first using GPT-3.5, then analyze summary
   - Truncate transcript to last 75% (usually most valuable content)

3. **Batch processing:**
   - If processing multiple calls at once, consider single API call with all transcripts
   - Saves on per-request overhead

4. **Caching:**
   - For recurring client calls, cache client intelligence and only update deltas
   - Store common pain points / upsells to reduce repeated analysis

**Cost tracking:**
- Monitor OpenRouter dashboard weekly
- Alert if costs exceed $1/call (indicates inefficiency)

---

## Prompt Testing Checklist

Before deploying new prompt version:

- [ ] Test with 3 different call types (discovery, check-in, strategy)
- [ ] Verify JSON output is always valid (no markdown)
- [ ] Check all required fields are present
- [ ] Confirm output quality meets standards
- [ ] Calculate cost per call (tokens used × model price)
- [ ] Test with edge cases (short calls, technical calls, difficult accents)
- [ ] Validate against production data (compare v1 vs v2)

---

## Model Selection Guide

**When to use GPT-4o-mini:**
- Content generation (creative writing)
- Summarization
- Format conversion
- Cost-sensitive operations
- **Cost:** $0.15/1M input tokens

**When to use Claude 3.5 Sonnet:**
- Deep analysis and reasoning
- Nuanced sentiment detection
- Complex extraction tasks
- When accuracy > cost
- **Cost:** $3.00/1M input tokens

**When to use GPT-3.5-turbo:**
- Simple formatting tasks
- Quick summaries
- High-volume operations
- Testing/development
- **Cost:** $0.50/1M input tokens

---

## Version Control

When updating prompts:

1. Document change reason and date
2. Test thoroughly before deploying
3. Keep previous version for 30 days
4. Monitor quality metrics post-deployment
5. Revert if quality drops >10%

**Quality metrics to track:**
- Content ideas: % that client actually uses
- Intelligence: Accuracy of sentiment vs actual churn
- Summaries: Client feedback / time saved
