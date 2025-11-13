# Fathom Automation System

**Transforms every client call into actionable content, intelligence insights, and professional summaries.**

## Overview

This automation system connects Fathom (call recording) → Make.com (workflow automation) → Notion (content & intelligence hub) to:

- Generate 5-10 content ideas per call with AI-drafted versions for high-priority items
- Track client health intelligence (sentiment, pain points, upsells, red flags)
- Create executive call summaries automatically
- Build a searchable content pipeline for ghostwriting services

## System Architecture

```
Fathom Recording
    ↓
Make.com Workflow
    ├── Fathom: Watch New Recordings (trigger)
    ├── Fathom: Get Transcript
    ├── Fathom: Get Summary
    ├── Notion: Create Call Record
    └── Router (3 parallel paths)
        ├── Path A: Content Ideas Generator (OpenRouter/GPT-4)
        ├── Path B: Client Intelligence Analyzer (OpenRouter/Claude)
        └── Path C: Executive Summary Generator (OpenRouter/GPT-4)
    ↓
Notion Databases
    ├── Call Intelligence Hub
    ├── Content Ideas Bank
    └── Client Intelligence
```

## Tech Stack

- **Fathom**: Call recording & transcription
- **Make.com**: Workflow automation (processes recordings every 15 min)
- **OpenRouter**: AI API gateway (GPT-4 + Claude access)
- **Notion**: Data storage, dashboard, content pipeline

## Repository Structure

```
/
├── README.md                          # This file
├── docs/
│   ├── 01-notion-database-setup.md   # Database property configurations
│   ├── 02-makecom-workflow-guide.md  # Complete Make.com scenario setup
│   ├── 03-openrouter-setup.md        # API key setup & model selection
│   ├── 04-testing-protocol.md        # Testing & troubleshooting
│   ├── 05-dashboard-views.md         # Notion dashboard configuration
│   └── 06-troubleshooting-common-errors.md  # Error resolution guide
├── config/
│   ├── .env.example                  # Environment variables template
│   └── openrouter-prompts.md         # All AI prompts with versioning
└── make-scenarios/
    └── README.md                      # JSON scenario import instructions
```

## Quick Start

### Prerequisites

1. **Fathom Account** with API access
   - Get API credentials from Fathom settings
   - Note your workspace ID

2. **Make.com Account** (Free tier works for MVP)
   - Sign up at make.com
   - Minimum 1,000 operations/month needed (Free: 1,000, Core: 10,000)

3. **OpenRouter Account**
   - Sign up at openrouter.ai
   - Get API key (free tier available)
   - Add $5-10 credit for testing

4. **Notion Workspace**
   - Create integration at notion.so/my-integrations
   - Get integration token
   - Share databases with integration

### Setup Steps

1. **Set up Notion databases** → See `docs/01-notion-database-setup.md`
2. **Configure OpenRouter** → See `docs/03-openrouter-setup.md`
3. **Build Make.com workflow** → See `docs/02-makecom-workflow-guide.md`
4. **Test with sample recording** → See `docs/04-testing-protocol.md`
5. **Configure dashboard views** → See `docs/05-dashboard-views.md`

## Cost Breakdown

| Service | Tier | Monthly Cost | Notes |
|---------|------|--------------|-------|
| Fathom | Pro | $0-19 | Free tier available, Pro has API access |
| Make.com | Free/Core | $0-9 | Free: 1K ops, Core: 10K ops |
| OpenRouter | Pay-as-go | $10-15 | GPT-4o-mini: ~$0.15/call, Claude: ~$0.30/call |
| Notion | Plus | $8-10 | Required for API access |
| **Total** | | **$18-53** | Scales with call volume |

**Estimated Operations per Call:**
- Fathom trigger: 1 op
- Get transcript: 1 op
- Get summary: 1 op
- Create call record: 1 op
- Content ideas: 10-15 ops (API call + iterator + Notion creates)
- Intelligence analyzer: 3 ops
- Executive summary: 2 ops
- **Total: ~20-25 ops per call**

**Monthly at 20 calls:** ~400-500 Make.com operations (well within free tier)

## What Gets Automated

### Per Call Processing

**Content Ideas (Path A):**
- 5-10 content ideas extracted from transcript
- Each idea includes: title, hook, core concept, platform fit, priority score
- High-priority ideas (8+) get full AI-drafted content
- Auto-categorized by type: LinkedIn Post, Newsletter, Carousel, Thread, Video Script

**Client Intelligence (Path B):**
- Sentiment analysis (Improving/Stable/Watch/At Risk)
- Pain points extraction
- Wins & celebrations captured
- Upsell opportunities identified
- Red flags detection
- Recommended next actions

**Executive Summary (Path C):**
- Professional call overview
- Key outcomes & decisions
- Top insights
- Action items count
- Next steps with timing
- Watch items (if applicable)

## Success Metrics

**Efficiency Gains:**
- Manual note-taking: 30-45 min → 0 min
- Content ideation: 60-90 min → automated
- Client follow-up doc: 20-30 min → automated
- **Total time saved: 2-3 hours per call**

**Revenue Impact:**
- Content ideas feed ghostwriting service
- Client intelligence prevents churn
- Professional summaries increase perceived value
- Clear ROI for client retainers

## Support & Troubleshooting

### Common Issues

1. **Notion date range errors**
   - Error: "[400] invalid date range, start date must be before end date"
   - Fix: Change date properties from date range to single date in Notion
   - See: `docs/06-troubleshooting-common-errors.md` for detailed solution

2. **Make.com scenario not triggering**
   - Check Fathom API connection
   - Verify webhook is active
   - Check operation limit

3. **OpenRouter API errors**
   - Verify API key in headers
   - Check credit balance
   - Confirm model availability

4. **Notion creation failures**
   - Verify integration has access to databases
   - Check property name matches exactly
   - Confirm relation fields use database IDs

**Full error reference:** See `docs/06-troubleshooting-common-errors.md` for comprehensive solutions to all common errors.

## Roadmap

### MVP (Current)
- ✅ Notion database structure
- 🔄 Make.com core workflow
- ⏳ Content Ideas Generator
- ⏳ Client Intelligence Analyzer
- ⏳ Executive Summary Generator

### Phase 2 (Weeks 3-4)
- Smart Task Generator with auto-assignment
- Due date calculations
- Task dependency tracking

### Phase 3 (Month 2+)
- Program Implementation Tracker
- Automation opportunity detection
- Implementation backlog builder
- Complexity & hour estimation

### Phase 4 (Month 3+)
- Automated client email follow-ups
- Weekly intelligence digest
- Content performance tracking
- Client sentiment trending

## License

Private - Internal use only

## Questions or Issues?

Refer to documentation in `/docs` or check Make.com execution history for debugging.
