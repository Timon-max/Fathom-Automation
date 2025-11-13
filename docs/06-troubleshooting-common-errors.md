# Troubleshooting Common Errors

**Quick reference guide for resolving common Make.com and Notion integration issues**

---

## Notion Date Range Errors

### Error: `[400] has an invalid date range, start date must be before end date`

**Cause:** A Notion date property is configured as a date range (requiring both start and end dates), but Make.com is only sending a single date value.

**Affected Properties:**
- Call Date (Call Intelligence Hub)
- Last Call Date (Client Intelligence)
- Next Call Date (Client Intelligence)

**Solution 1: Change Notion Property to Single Date (Recommended)**

1. Open the Notion database with the problematic property
2. Click on the date property name to open its settings
3. Look for "Date range" or "End date" toggle
4. **Turn OFF** the date range feature
5. Click outside to save
6. Re-run the Make.com scenario

**Solution 2: Update Make.com to Send Date Ranges**

If you need date ranges, modify the Make.com module:

**Before:**
```
Call Date: {{1.createdAt}}
```

**After:**
```
Call Date:
  Start: {{1.createdAt}}
  End: {{1.createdAt}}
```

Apply this fix to all date properties in:
- Module [4]: Create Call Record
- Module [8B-1]: Update Existing Client
- Module [8B-2]: Create New Client

---

## Transcript Extraction Errors

### Error: Transcript is incomplete or only captures first half

**Cause:** Fathom API might truncate long transcripts or Make.com variable limits.

**Solution:**

1. **Check transcript length in Fathom:**
   - Open the recording in Fathom
   - Verify the full transcript is available
   - If truncated in Fathom, this is a Fathom processing issue

2. **Increase Make.com text field limits:**
   - In Module [2] (Get Transcript), check the output size
   - If over 50KB, consider splitting into chunks
   - Or use Fathom's "Get Recording" endpoint which may have different limits

3. **Alternative: Store transcript URL instead:**
   - Instead of storing full transcript text
   - Store the Fathom recording URL
   - Access transcript when needed via Fathom interface

**Workaround for AI Processing:**

If transcript is truncated but you need to process it:

```json
"content": "Analyze this PARTIAL coaching call transcript (beginning only)...\n\nImportant: This transcript may be incomplete. Focus on content that is clearly present.\n\nTranscript:\n{{2.transcript}}"
```

Adjust AI prompts to acknowledge partial data.

---

## Notion Property Errors

### Error: `[400] Property not found`

**Cause:** Property name in Make.com doesn't exactly match Notion database property name.

**Solution:**

1. **Check property name spelling:**
   - Property names are case-sensitive
   - "Call Title" ≠ "call title" ≠ "Call title"

2. **Verify property exists:**
   - Open Notion database
   - Confirm property name matches exactly
   - Check for hidden spaces or special characters

3. **Refresh Make.com connection:**
   - In Make.com module, click "Show advanced settings"
   - Click "Refresh" next to property dropdown
   - This reloads property list from Notion

---

## Notion Relation Errors

### Error: `[400] Relation could not be created`

**Cause:** Trying to relate to a page that doesn't exist or integration doesn't have access.

**Solution:**

1. **Verify both databases share same integration:**
   - Open both databases in Notion
   - Check both show your integration under "Connections"
   - If not, click ••• → Add connections → Select your integration

2. **Verify page ID exists:**
   - In Make.com, verify `{{4.id}}` (call record) was created successfully
   - Check execution history - the Create Call Record step should show a valid page ID
   - If empty, the relation will fail

3. **Check relation property configuration:**
   - Open Content Ideas Bank database
   - Click "Source Call" relation property
   - Verify it points to "Call Intelligence Hub" database
   - Verify relation type allows linking

---

## OpenRouter API Errors

### Error: `[401] Unauthorized`

**Cause:** Invalid or missing API key.

**Solution:**

1. **Verify API key format:**
   - Should start with `sk-or-v1-...`
   - Copy fresh key from openrouter.ai/keys

2. **Check header formatting:**
   ```
   Header Name: Authorization
   Header Value: Bearer YOUR_API_KEY
   ```
   - Must include "Bearer " prefix
   - No extra quotes or spaces

3. **Regenerate key if needed:**
   - Go to openrouter.ai/keys
   - Create new key
   - Update all HTTP modules with new key

### Error: `[402] Payment Required`

**Cause:** Insufficient credits in OpenRouter account.

**Solution:**

1. Go to openrouter.ai/billing
2. Add $5-10 credit
3. Wait 1-2 minutes for credit to reflect
4. Re-run scenario

### Error: `[429] Rate limit exceeded`

**Cause:** Too many API requests in short time.

**Solution:**

1. **Add delays between requests:**
   - In Make.com, add "Sleep" module between HTTP requests
   - Set delay to 1-2 seconds

2. **Reduce concurrent execution:**
   - Lower the "Limit" in Fathom trigger (Module 1)
   - Process fewer recordings per run

---

## JSON Parsing Errors

### Error: `JSON parse failed` or `Unexpected token`

**Cause:** OpenRouter returned non-JSON response (often markdown code blocks or error messages).

**Solution:**

1. **Check AI response format:**
   - Click failed module in execution history
   - View `{{5A.data.choices[0].message.content}}`
   - If it contains ```json or markdown formatting, AI didn't follow instructions

2. **Update prompt to be more explicit:**

   **Add to end of prompt:**
   ```
   CRITICAL: Return ONLY valid JSON. No markdown code blocks, no explanations, no commentary. Start with [ or { character.
   ```

3. **Lower temperature:**
   - In HTTP request body, change `"temperature": 0.7` to `"temperature": 0.3`
   - Lower temperature = more consistent formatting

4. **Add JSON validation:**
   - Before JSON Parse module, add a "Text Parser" module
   - Use regex to strip markdown code blocks: `^```json\n|\n```$`

---

## Iterator Errors

### Error: `Iterator has no items to process`

**Cause:** JSON parsing didn't create an array, or array is empty.

**Solution:**

1. **Verify JSON structure:**
   - Check that OpenRouter returned an array: `[{...}, {...}]`
   - Not an object: `{ideas: [{...}]}`

2. **Update iterator array path:**
   - If response is `{ideas: [{...}]}`, use `{{6A.ideas}}` not `{{6A.array}}`

3. **Add fallback for empty arrays:**
   - Before iterator, add "Router" with filter
   - Route 1: `{{length(6A.array)}}` → Greater than → 0
   - Route 2: Fallback (skip iteration)

---

## Make.com Operations Limit

### Error: Scenario stopped, operations limit reached

**Cause:** Exceeded Make.com plan's operation limit.

**Solution:**

**Free Tier (1,000 ops/month):**
- Supports ~40 calls/month at 25 ops/call
- Reduce processing frequency (30 min instead of 15 min)
- Disable auto-drafting (skip Path A modules 10-11)

**Upgrade to Core ($9/mo, 10,000 ops):**
- Supports ~400 calls/month
- Recommended if processing 50+ calls/month

**Monitor usage:**
1. Make.com dashboard → Organization → Operations
2. Track consumption weekly
3. Set up low balance alerts

---

## Debugging Workflow

**General troubleshooting steps for any error:**

1. **Check execution history:**
   - Open scenario → Click "History" tab
   - Click most recent failed execution
   - Identify red (failed) module

2. **Inspect module data:**
   - Click failed module
   - Expand "Input" section (what data was sent)
   - Expand "Output" section (what error was returned)
   - Expand "Mapping" section (how variables were mapped)

3. **Test in isolation:**
   - Right-click module → "Run this module only"
   - Uses data from last successful execution
   - Helps identify if issue is with this module or previous ones

4. **Verify credentials:**
   - Fathom API key valid
   - OpenRouter API key valid and funded
   - Notion integration has access to all databases

5. **Check variable mapping:**
   - Verify `{{X.field}}` references correct module number
   - Module numbers can change if you add/remove modules
   - Use variable picker instead of typing manually

---

## Preventive Maintenance

**Weekly:**
- Review execution history for failures
- Check OpenRouter credit balance
- Monitor Make.com operations usage

**Monthly:**
- Test scenario with new recording
- Verify all 3 Notion databases populating correctly
- Check for any stale data or broken relations

**Quarterly:**
- Review and update AI prompts based on output quality
- Audit Notion property configurations
- Update Make.com modules if API versions change

---

## Getting Help

**If issue persists after trying solutions above:**

1. **Export execution log:**
   - Open failed execution in Make.com
   - Click ••• → Download execution data
   - Save JSON file for debugging

2. **Check integration status:**
   - Fathom status: status.fathom.video
   - OpenRouter status: status.openrouter.ai
   - Notion status: status.notion.so

3. **Review API documentation:**
   - Fathom API: fathom.video/api-docs
   - OpenRouter: openrouter.ai/docs
   - Notion: developers.notion.com

4. **Community resources:**
   - Make.com Community: community.make.com
   - Notion API Slack: notion.so/slack

---

## Error Code Quick Reference

| Error Code | Service | Common Cause | Quick Fix |
|------------|---------|--------------|-----------|
| 400 | Notion | Invalid property value or missing required field | Check property names and data types |
| 401 | OpenRouter | Invalid API key | Regenerate and update API key |
| 402 | OpenRouter | Insufficient credits | Add credits to account |
| 404 | Notion | Page or database not found | Verify database IDs |
| 429 | OpenRouter | Rate limit exceeded | Add delays between requests |
| 500 | Any | Server error | Wait and retry, check service status |

---

## Next Steps

- Return to `docs/04-testing-protocol.md` for systematic testing
- Review `docs/02-makecom-workflow-guide.md` for module configuration details
- Check `config/openrouter-prompts.md` for prompt optimization
