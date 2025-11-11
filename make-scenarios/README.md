# Make.com Scenario Files

**Instructions for exporting, importing, and version controlling Make.com scenarios**

---

## Overview

This directory will contain JSON exports of your Make.com scenarios for:
- **Backup:** Restore if scenario breaks
- **Version control:** Track changes over time
- **Portability:** Move to different Make.com account
- **Documentation:** Reference exact configuration

---

## How to Export a Scenario

### From Make.com Interface

1. Open your scenario in Make.com
2. Click the **hamburger menu** (≡) in top left
3. Select **"Download blueprint"**
4. Save as: `fathom-content-intelligence-v1.json`
5. Move file to this directory: `/make-scenarios/`

### What Gets Exported

**Included:**
- All module configurations
- Module connections and flow
- Filters and routers
- Variable mappings
- Scheduling settings

**NOT included (you'll need to reconfigure on import):**
- API keys and credentials
- Connection details (Fathom, Notion, OpenRouter)
- Database IDs
- Scenario ID (gets new ID on import)

---

## How to Import a Scenario

### Into Make.com

1. Go to: https://make.com/scenarios
2. Click **"Create a new scenario"**
3. Click **hamburger menu** (≡) → **"Import blueprint"**
4. Select JSON file from this directory
5. Click **"Import"**

### Post-Import Configuration Required

After importing, you MUST reconfigure:

**1. Recreate connections:**
- Fathom connection (API key)
- Notion connection (integration token)
- OpenRouter connection (HTTP headers with API key)

**2. Update IDs:**
- Notion database IDs (3 databases)
- Fathom workspace ID

**3. Verify mappings:**
- Check all variable references still work
- Property names match your Notion databases

**4. Test thoroughly:**
- Run once with test data
- Verify all paths execute
- Check Notion receives data correctly

---

## File Naming Convention

Use semantic versioning for scenario exports:

```
fathom-content-intelligence-v[MAJOR].[MINOR].[PATCH].json
```

**Examples:**

```
fathom-content-intelligence-v1.0.0.json
  - Initial MVP release
  - Core functionality: content ideas, intelligence, summary

fathom-content-intelligence-v1.1.0.json
  - Added: High-priority content drafting
  - Added: Client duplicate detection

fathom-content-intelligence-v1.1.1.json
  - Fixed: JSON parsing error handling
  - Updated: AI prompts for better quality

fathom-content-intelligence-v2.0.0.json
  - Breaking change: New database structure
  - Added: Task generation module
```

**When to increment:**
- **MAJOR:** Breaking changes (requires database restructure)
- **MINOR:** New features (backward compatible)
- **PATCH:** Bug fixes, prompt updates, minor tweaks

---

## Version Control Strategy

### Recommended Workflow

**Before making major changes:**

1. Export current working version
2. Name with date: `fathom-content-intelligence-v1.0.0-backup-2024-01-11.json`
3. Make changes in Make.com
4. Test thoroughly
5. If successful: Export as new version
6. If failed: Import backup and revert

**Weekly backups:**
- Every Friday, export scenario
- Keep last 4 weekly backups
- Delete older backups (keep major versions only)

---

## Scenario Templates

This directory will contain:

### 1. Core MVP Scenario

**File:** `fathom-content-intelligence-mvp.json`

**Includes:**
- Fathom trigger (watch recordings)
- Get transcript + summary
- Create call record
- Path A: Content ideas generator
- Path B: Client intelligence analyzer
- Path C: Executive summary generator

**Use case:** Initial deployment

---

### 2. Testing Scenario

**File:** `fathom-test-scenario.json`

**Includes:**
- Manual trigger (button) instead of scheduled
- Uses cheaper models (GPT-3.5)
- Single test recording ID (hardcoded)
- All AI calls disabled (dry run mode)

**Use case:** Testing changes without using API credits

---

### 3. Phase 2 Scenario

**File:** `fathom-content-intelligence-v2.json`

**Includes:**
- Everything from MVP
- Task generator module
- Automated email follow-ups
- Slack notifications

**Use case:** Future expansion (Month 2+)

---

## Configuration Files

### Connection Template

**File:** `connections-template.json`

Contains connection names and types (without credentials):

```json
{
  "connections": [
    {
      "name": "Fathom Production",
      "type": "fathom",
      "required_credentials": ["api_key", "workspace_id"],
      "notes": "Get from app.fathom.video/settings/integrations"
    },
    {
      "name": "Notion Production",
      "type": "notion",
      "required_credentials": ["integration_token"],
      "notes": "Get from notion.so/my-integrations"
    },
    {
      "name": "OpenRouter API",
      "type": "http",
      "required_credentials": ["api_key"],
      "notes": "Get from openrouter.ai/keys"
    }
  ]
}
```

**Use case:** Reference when setting up new Make.com account

---

### Database IDs Template

**File:** `database-ids-template.json`

```json
{
  "notion_databases": {
    "call_intelligence_hub": "YOUR_32_CHAR_DATABASE_ID",
    "content_ideas_bank": "YOUR_32_CHAR_DATABASE_ID",
    "client_intelligence": "YOUR_32_CHAR_DATABASE_ID"
  },
  "instructions": "Replace these IDs with your actual Notion database IDs. Get from database URL when opened as full page."
}
```

**Use case:** Quick reference for all database IDs in one place

---

## Scenario Documentation

For each exported scenario, create a companion markdown file:

### Template: Scenario Changelog

**File:** `fathom-content-intelligence-v1.0.0-changelog.md`

```markdown
# Scenario Version 1.0.0

**Release Date:** 2024-01-11
**Author:** [Your Name]
**Make.com Scenario ID:** 123456

## Changes

### Added
- Initial MVP implementation
- Content ideas generator (Path A)
- Client intelligence analyzer (Path B)
- Executive summary generator (Path C)

### Changed
- N/A (initial release)

### Fixed
- N/A (initial release)

## Configuration Notes

**API Models Used:**
- Content: openai/gpt-4o-mini
- Intelligence: anthropic/claude-3.5-sonnet
- Summary: openai/gpt-4o-mini

**Estimated Cost:**
- $0.45-0.65 per call
- ~20-25 Make.com operations per call

## Known Issues

- None

## Testing Checklist

- [x] Fathom trigger works
- [x] Transcript retrieval works
- [x] All 3 paths execute
- [x] Notion records created correctly
- [x] JSON parsing handles errors
- [x] Client duplicate detection works

## Rollback Instructions

If this version breaks:
1. Import: `fathom-content-intelligence-v0.9.0.json`
2. Reconfigure connections
3. Test with single recording

## Next Planned Changes

- Add error handling for empty transcripts
- Optimize token usage in prompts
- Add retry logic for API timeouts
```

---

## Git Best Practices

### What to Commit

**DO commit:**
- ✅ Scenario JSON exports
- ✅ Configuration templates
- ✅ Changelog files
- ✅ This README

**DON'T commit:**
- ❌ Files with actual API keys
- ❌ Filled-in database IDs (use template only)
- ❌ Credentials of any kind
- ❌ Test exports with sensitive data

---

### .gitignore Additions

Add to `.gitignore`:

```
# Exclude working drafts
make-scenarios/*-draft.json
make-scenarios/*-temp.json
make-scenarios/*-backup-*.json

# Exclude filled configs
make-scenarios/connections.json
make-scenarios/database-ids.json

# Keep only templates and versioned releases
!make-scenarios/*-template.json
!make-scenarios/fathom-content-intelligence-v*.json
```

---

## Disaster Recovery

### Full Scenario Loss

If Make.com account is deleted or scenario corrupted:

**Recovery steps:**

1. Create new Make.com account (or new scenario)
2. Import latest version: `fathom-content-intelligence-v[latest].json`
3. Reconfigure connections:
   - Use `/config/.env` for API keys
   - Use `database-ids-template.json` for Notion IDs
4. Follow post-import checklist above
5. Run test with single recording
6. Activate scenario

**Recovery time:** ~30-45 minutes

---

### Partial Corruption

If only some modules break:

1. Export broken scenario as `broken-[date].json` (for debugging)
2. Import last working version
3. Manually port changes made since last version
4. Test thoroughly
5. Export as new patch version

---

## Sharing Scenarios

### Making Scenario Portable

Before sharing with team member or client:

1. Export scenario
2. Create sanitized version:
   - Remove all database IDs → replace with `YOUR_DATABASE_ID_HERE`
   - Remove API keys → replace with `YOUR_API_KEY_HERE`
   - Add comments/notes in modules explaining configuration
3. Create setup guide specific to that scenario
4. Test import on fresh Make.com account

**Example portable file:**
`fathom-content-intelligence-v1.0.0-portable.json`

---

## Scenario Comparison

### Comparing Two Versions

**To see what changed:**

1. Export both versions
2. Use JSON diff tool:
   - Online: jsondiff.com
   - CLI: `diff -u v1.json v2.json`
   - VSCode: Install "Diff" extension

**Focus on:**
- New modules added
- Changed filters
- Updated API endpoints
- Modified variable mappings

---

## Advanced: Scenario Automation

### Automated Exports (Future Enhancement)

**Using Make.com API:**

Create a separate scenario that:
1. Triggers weekly (Sunday midnight)
2. Calls Make.com API to export scenario
3. Saves to GitHub via API
4. Tags with version number
5. Sends notification to Slack

**Benefits:**
- Never forget to backup
- Full change history
- Automatic documentation

**Setup guide:** TBD (Phase 3)

---

## Scenario Performance Tracking

### Execution Logs

**File:** `execution-logs/YYYY-MM-execution-summary.md`

Track monthly performance:

```markdown
# January 2024 Execution Summary

## Stats
- Total executions: 87
- Successful: 85 (97.7%)
- Failed: 2 (2.3%)
- Avg duration: 3m 42s
- Total operations: 1,847

## Failures
1. 2024-01-05 14:30 - JSON parse error (prompt updated)
2. 2024-01-18 09:15 - OpenRouter 502 (timeout, retried successfully)

## Optimizations Made
- Reduced token usage by 15% (truncate long transcripts)
- Added 2-second delay between API calls
- Switched summary generation to GPT-3.5-turbo

## Cost Analysis
- OpenRouter: $12.45
- Make.com: $0 (under free tier)
- Total: $12.45
- Cost per call: $0.14 (down from $0.21 last month)
```

---

## Support

**Make.com Blueprint Help:**
https://www.make.com/en/help/scenarios/blueprint

**Community Forum:**
https://community.make.com

**Issues:**
If you encounter problems with imports/exports, document in:
`make-scenarios/issues/[date]-[issue-description].md`

---

## Checklist: Before Exporting

- [ ] Scenario is working (run once test passed)
- [ ] All modules properly connected
- [ ] No broken connections
- [ ] Version number incremented
- [ ] Changelog updated
- [ ] Test recording successful
- [ ] Committed to Git with meaningful message

---

## Next Steps

1. ✅ Build your first scenario (see `docs/02-makecom-workflow-guide.md`)
2. ✅ Test thoroughly
3. ✅ Export as v1.0.0
4. ✅ Save to this directory
5. ✅ Create changelog file
6. ✅ Commit to Git
