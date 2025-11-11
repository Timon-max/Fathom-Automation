# Fathom Automation - Quick Start Guide

**Get up and running in 4-5 hours**

---

## Prerequisites (10 minutes)

You need accounts for:

1. **Fathom** - Get API key from Settings → Integrations → API
2. **Make.com** - Sign up at make.com (free tier works)
3. **OpenRouter** - Sign up at openrouter.ai, add $10 credit
4. **Notion** - Create integration at notion.so/my-integrations

---

## Setup Steps

### 1. Copy Environment Variables (2 minutes)

```bash
cp config/.env.example config/.env
```

Edit `config/.env` and fill in:
- `FATHOM_API_KEY` - From Fathom settings
- `OPENROUTER_API_KEY` - From openrouter.ai/keys
- `NOTION_API_TOKEN` - From notion.so/my-integrations
- `NOTION_DB_*` - Your 3 database IDs (32-char strings from URLs)

---

### 2. Share Notion Databases (3 minutes)

For each database:
1. Open database
2. Click ••• → Add connections
3. Select your integration

Databases needed:
- Call Intelligence Hub
- Content Ideas Bank
- Client Intelligence

---

### 3. Build Make.com Scenario (3-4 hours)

**Open:** `docs/02-makecom-workflow-guide.md`

**Follow step-by-step:**
- Part 1: Core Workflow (Modules 1-5)
- Part 2: Path A - Content Ideas Generator
- Part 3: Path B - Client Intelligence Analyzer
- Part 4: Path C - Executive Summary Generator
- Part 5: Testing

**Tips:**
- Build one module at a time
- Test after each major section
- Reference `config/openrouter-prompts.md` for AI prompts
- Use troubleshooting section if stuck

---

### 4. Test with Recording (15 minutes)

1. Run scenario once (click "Run once")
2. Check Make.com execution history
3. Verify in Notion:
   - Call record created in Call Intelligence Hub
   - 5-10 content ideas in Content Ideas Bank
   - Client record in Client Intelligence
   - Executive Summary populated

**If errors:** See `docs/04-testing-protocol.md`

---

### 5. Configure Dashboard (30 minutes)

**Open:** `docs/05-dashboard-views.md`

Create these views:
1. Recent Calls (gallery)
2. Content Pipeline (board - grouped by Status)
3. Client Health Monitor (gallery - grouped by Sentiment)
4. High-Priority Content (table - filter Priority >= 8)

---

### 6. Go Live (5 minutes)

1. Toggle scenario ON in Make.com
2. Set schedule: Every 15 minutes
3. Enable error notifications
4. Export scenario as backup

---

## What Happens Next?

**Every 15 minutes:**
- Make.com checks for new Fathom recordings
- New calls are processed automatically
- Results appear in your Notion databases

**You should:**
- Check dashboard weekly (5 min)
- Review content ideas and publish best ones
- Monitor client health alerts
- Track costs in OpenRouter dashboard

---

## Cost Estimate

**Per call:** ~$0.50
**Per month (20 calls):** $10-15

---

## Need Help?

**See full documentation:**
- `HANDOFF.md` - Complete status and next steps
- `docs/02-makecom-workflow-guide.md` - Main implementation guide
- `docs/04-testing-protocol.md` - Troubleshooting

**Common issues:**
- Property not found → Check exact property names in Notion
- 401 error → Verify API key in headers
- JSON parse error → Update prompt (see troubleshooting guide)

---

## Success Checklist

- [ ] All API keys configured
- [ ] Notion databases shared with integration
- [ ] Make.com scenario built and tested
- [ ] First recording processed successfully
- [ ] Dashboard views created
- [ ] Scenario enabled and running

**You're done!** 🎉

Check your Notion dashboard in 15 minutes to see your first automated call processing.
