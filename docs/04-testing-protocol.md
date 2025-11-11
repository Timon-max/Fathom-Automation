# Testing Protocol & Troubleshooting

**Comprehensive testing checklist and debugging guide**

---

## Pre-Launch Testing Checklist

### Phase 1: Individual Module Testing

Test each module independently before connecting the full workflow.

#### Fathom Modules

- [ ] **Module 1: Watch New Recordings**
  - Trigger fires on schedule (check execution history after 15 min)
  - Returns at least 1 recent recording
  - All expected fields present (id, title, createdAt, duration, shareUrl)

- [ ] **Module 2: Get Transcript**
  - Returns non-empty transcript for test recording
  - Handles recordings without transcript gracefully (null/empty)
  - Text format is clean (no encoding issues)

- [ ] **Module 3: Get Summary**
  - Returns Fathom's AI summary
  - Summary is coherent and relevant to call
  - Handles recordings without summary

#### Notion Modules

- [ ] **Module 4: Create Call Record**
  - Successfully creates item in Call Intelligence Hub
  - All properties map correctly (check each field)
  - Select properties use valid option names
  - Date format is correct
  - Returns valid page ID in `{{4.id}}`

#### OpenRouter API Modules

- [ ] **Module 5A: Content Ideas Generator**
  - Returns valid JSON (no markdown code blocks)
  - JSON contains array of 5-10 ideas
  - Each idea has all required fields
  - Priority scores range from 1-10
  - Ideas are relevant to call content

- [ ] **Module 6A: Parse JSON**
  - Successfully converts JSON string to array
  - Array is iterable
  - Individual fields are accessible ({{6A.title}}, etc.)

- [ ] **Module 7A: Iterator**
  - Loops correct number of times (5-10)
  - Each iteration has access to idea fields

- [ ] **Module 8A: Create Content Idea**
  - Creates items in Content Ideas Bank
  - Relation to source call works ({{4.id}} links properly)
  - Multi-select arrays populate correctly
  - Number fields accept integer values

- [ ] **Module 9A: Filter**
  - Only high-priority ideas (8+) pass through
  - Lower priority ideas skip drafting

- [ ] **Module 10A: Draft Content**
  - Returns complete, formatted content
  - Length matches content type requirements
  - No meta-commentary or XML tags

- [ ] **Module 11A: Update with Draft**
  - Updates correct content idea record
  - Polished Draft field populates
  - Status changes to "Ready to Draft"

- [ ] **Module 5B: Intelligence Analyzer**
  - Returns valid JSON
  - Sentiment is one of 4 valid options
  - Client name extracted correctly
  - Arrays populated with relevant data

- [ ] **Module 7B: Search Client**
  - Finds existing client if present
  - Returns empty if client is new

- [ ] **Module 8B: Create/Update Client**
  - Creates new record if needed
  - Updates existing record without duplication
  - Multi-select fields add without removing old values

- [ ] **Module 5C: Executive Summary**
  - Returns formatted Markdown
  - Includes all required sections
  - No placeholder text (actual analysis)

- [ ] **Module 6C: Update Call with Summary**
  - Updates correct call record
  - Summary text displays properly in Notion
  - Processing Status changes to "Complete"

---

### Phase 2: End-to-End Testing

Test complete workflow with real recording.

#### Test Scenario 1: New Client Call

**Setup:**
1. Record 5-10 minute test call (or use existing recording)
2. Ensure transcript is available in Fathom
3. Note call title and expected client name

**Expected Results:**

- [ ] **Call Intelligence Hub receives:**
  - Call title matches Fathom
  - Transcript populated (>100 words)
  - Fathom Summary populated
  - Processing Status = "Complete"
  - Executive Summary formatted properly
  - All date/number fields correct

- [ ] **Content Ideas Bank receives:**
  - 5-10 new content idea records created
  - Each linked to source call (relation works)
  - Priority scores vary (not all 10 or all 5)
  - Platform Fit arrays populated
  - High-priority ideas (8+) have Polished Draft
  - Status set correctly

- [ ] **Client Intelligence receives:**
  - New client record created
  - Client name extracted correctly
  - Sentiment assigned (Improving/Stable/Watch/At Risk)
  - Pain Points populated
  - Recent Wins captured
  - Upsell Opportunities listed
  - Red Flags present if applicable

**Timing:**
- Total execution time: 2-5 minutes
- Operations used: 20-30

---

#### Test Scenario 2: Existing Client Follow-up Call

**Setup:**
1. Use same client name as Test Scenario 1
2. Record or use another call
3. Run workflow

**Expected Results:**

- [ ] **Client Intelligence updates (not duplicates):**
  - Last Call Date updated to new call
  - Sentiment may change based on call
  - Pain Points add to existing (not replace)
  - Recent Wins prepend new wins
  - Red Flags update if new concerns

- [ ] **No duplicate client records**
  - Only 1 record per client name
  - Search function found existing record

---

#### Test Scenario 3: High-Priority Content Draft

**Setup:**
1. Use call with clear, actionable content opportunities
2. Expect at least 1 idea to score 8+

**Expected Results:**

- [ ] **High-priority idea (8+):**
  - Has Polished Draft populated
  - Draft is complete (150-400 words depending on type)
  - Format matches content type
  - Status = "Ready to Draft"
  - Hook is compelling

- [ ] **Lower priority ideas (7 or below):**
  - No Polished Draft (empty field)
  - Status = "New Idea"

---

### Phase 3: Edge Case Testing

Test workflow resilience with unusual inputs.

#### Edge Case 1: Very Short Call (< 3 minutes)

**Expected Behavior:**
- Workflow completes without errors
- May generate fewer ideas (2-5 instead of 5-10)
- Intelligence analysis still provides sentiment
- Summary is shorter but complete

---

#### Edge Case 2: No Transcript Available

**Expected Behavior:**
- Workflow should handle gracefully
- Option A: Skip AI processing if transcript empty
- Option B: Use Fathom summary only (less detailed results)
- **Add error handling:** Check if `{{2.transcript}}` is empty before AI calls

**Fix if needed:**
Add filter after Module 3:
- Condition: `{{length(2.transcript)}}` > 100
- Fallback: Log error and skip AI processing

---

#### Edge Case 3: API Rate Limit Hit

**Expected Behavior:**
- OpenRouter: 429 error returned
- Make.com should retry automatically (if configured)

**Fix:**
1. Add error handler to HTTP modules
2. Set retry: 3 times with 2-second delay
3. Alert if all retries fail

---

#### Edge Case 4: Invalid JSON Response

**Scenario:** AI returns markdown code block instead of raw JSON

**Symptoms:**
- Module 6A (Parse JSON) fails
- Error: "Unexpected token" or "Invalid JSON"

**Fix:**
1. Update prompts to explicitly say "no markdown code blocks"
2. Add JSON cleaning step before parse:
   - Use Text Parser to remove ```json and ```
   - Regex: `replace(content; /```json\n?|```/g; "")`

---

#### Edge Case 5: Client Name Not Extracted

**Scenario:** Call title doesn't contain client name, transcript is vague

**Expected Behavior:**
- Client Intelligence still creates record
- Uses fallback name: "Client from [Call Title]"

**Fix if needed:**
Update intelligence prompt:
```
If client name is unclear, return:
"clientName": "Unknown - [First 30 chars of call title]"
```

---

## Troubleshooting Guide

### Common Errors & Solutions

#### Error: "Property Not Found"

**Full error message:**
```
Could not create database item: property [PropertyName] not found
```

**Cause:** Property name mismatch between Make.com and Notion

**Debug steps:**
1. Open Notion database
2. Check exact property name (case-sensitive, spaces matter)
3. Compare to Make.com module mapping
4. Update Make.com to match exactly

**Example:**
- Notion: "Call Title" (space)
- Make.com: "CallTitle" (no space) ❌
- Fix: Change to "Call Title" ✅

---

#### Error: "Invalid Database ID"

**Full error message:**
```
Could not find database: [database_id]
```

**Causes:**
1. Database ID is wrong
2. Integration doesn't have access to database

**Debug steps:**
1. Open database as full page
2. Copy URL: `notion.so/workspace/[DATABASE_ID]?v=...`
3. Extract 32-character ID (no dashes in URL version)
4. Verify integration has access:
   - Open database
   - Click ••• (top right)
   - Add connections → Find your integration

---

#### Error: "Relation Could Not Be Created"

**Full error message:**
```
Relation to [database] could not be created
```

**Causes:**
1. Source page ID ({{4.id}}) is empty or invalid
2. Both databases don't share same integration
3. Relation property isn't set up in Notion

**Debug steps:**
1. Check Module 4 output: `{{4.id}}` exists and is 32 characters
2. Verify both databases share integration access
3. In Content Ideas Bank, check "Source Call" property:
   - Type: Relation
   - Related database: Call Intelligence Hub
   - Relation type: Can link to multiple pages (or single, depending on setup)

---

#### Error: 401 Unauthorized (OpenRouter)

**Full error message:**
```
HTTP 401: Unauthorized
```

**Cause:** API key is invalid or missing

**Debug steps:**
1. Open Module 5A (or any HTTP module)
2. Check Headers section
3. Verify Authorization header:
   - Name: `Authorization`
   - Value: `Bearer sk-or-v1-...`
4. Test API key in OpenRouter dashboard
5. Regenerate key if needed

---

#### Error: 402 Payment Required (OpenRouter)

**Full error message:**
```
HTTP 402: Insufficient credits
```

**Cause:** OpenRouter account has $0 balance

**Fix:**
1. Go to openrouter.ai/account
2. Add credit ($5-10 recommended)
3. Retry scenario

---

#### Error: 429 Too Many Requests

**Full error message:**
```
HTTP 429: Rate limit exceeded
```

**Cause:** Too many API calls in short time

**Fix:**
1. Add delay between calls (Settings → Add delay: 2 seconds)
2. Reduce scenario frequency (30 min instead of 15 min)
3. Upgrade OpenRouter plan if hitting limits regularly

---

#### Error: JSON Parse Failed

**Full error message:**
```
Invalid JSON: Unexpected token '<' at position 0
```

**Cause:** AI returned HTML, markdown, or error message instead of JSON

**Debug steps:**
1. Check Module 5A output: `{{5A.data.choices[0].message.content}}`
2. If it contains ```json or markdown:
   - Update prompt: "Return ONLY valid JSON, no markdown code blocks"
   - Add text cleaning before parse
3. If it's an error message:
   - Check OpenRouter credit balance
   - Verify model is available
   - Test with simpler prompt

---

#### Error: Iterator Has No Items

**Symptom:** Module 7A shows 0 iterations

**Causes:**
1. JSON parsing failed (Module 6A has no array)
2. AI returned empty array `[]`
3. Wrong field mapped to iterator

**Debug steps:**
1. Check Module 6A output structure
2. Verify iterator input is `{{6A.array}}` not `{{6A}}`
3. If AI returned empty array:
   - Transcript may be too short
   - Prompt may need tuning
   - Test with different recording

---

#### Error: Module Skipped (Orange Icon)

**Symptom:** Module shows orange, says "Skipped"

**Cause:** Filter before this module didn't pass

**Expected behavior:** This is normal for:
- Module 10A (only runs for priority 8+)
- Module 8B-1 or 8B-2 (only one route runs)

**If unexpected:**
- Check filter condition
- Verify filter logic is correct
- Test with data that should pass filter

---

### Performance Issues

#### Scenario Takes >10 Minutes

**Normal duration:** 2-5 minutes

**Causes:**
1. OpenRouter API is slow (high load)
2. Transcript is very long (>20k words)
3. Too many content ideas generated (15+)

**Optimizations:**
1. Use faster models (GPT-3.5-turbo instead of GPT-4)
2. Truncate transcript to last 75% (most valuable content)
3. Limit ideas to 8 max in prompt

---

#### High Make.com Operations Usage

**Expected:** 20-25 ops per call

**If much higher (40+):**
- Check for loops creating too many items
- Verify iterator isn't running twice
- Look for failed modules that retry

**Optimization:**
- Reduce idea count to 5-8
- Combine API calls where possible
- Use bulk operations if available

---

## Monitoring & Maintenance

### Weekly Checks

- [ ] **Review execution history** (last 7 days)
  - Success rate should be >95%
  - Average execution time: 2-5 minutes
  - Operations per run: 20-30

- [ ] **Check Notion databases**
  - All recent calls processed
  - Content ideas linked correctly
  - Client intelligence updating (not creating duplicates)

- [ ] **Monitor costs**
  - OpenRouter usage (should be $2-5/week for 10-20 calls)
  - Make.com operations (should be under 1,000/month for free tier)

- [ ] **Quality spot-checks**
  - Pick 2-3 random content ideas - are they usable?
  - Check 1-2 executive summaries - accurate and helpful?
  - Verify client sentiment matches your assessment

---

### Monthly Maintenance

- [ ] **Prompt optimization**
  - Review content ideas - are they getting better or worse?
  - Check sentiment accuracy - any false positives?
  - Update prompts in `/config/openrouter-prompts.md`

- [ ] **Database cleanup**
  - Archive old calls (>90 days)
  - Remove duplicate client records if any
  - Update Select options if needed

- [ ] **Cost analysis**
  - Total spend: OpenRouter + Make.com
  - Cost per call processed
  - ROI: Time saved vs money spent

- [ ] **Feature requests**
  - What manual work remains?
  - What new insights would be valuable?
  - Prioritize Phase 2 features

---

## Testing Checklist Template

Use this for each scenario change or major update:

```
## Test: [Scenario Name] - [Date]

### Pre-Test Setup
- [ ] Scenario saved and activated
- [ ] All API keys valid
- [ ] Test recording prepared (ID: ______)
- [ ] Notion databases accessible

### Execution
- [ ] Scenario triggered (manual or scheduled)
- [ ] Start time: _______
- [ ] End time: _______
- [ ] Total duration: _______
- [ ] Operations used: _______

### Results
- [ ] Call record created in Call Intelligence Hub
- [ ] [X] content ideas generated
- [ ] [X] high-priority ideas have drafts
- [ ] Client intelligence created/updated
- [ ] Executive summary populated
- [ ] No errors in execution log

### Quality Check
- [ ] Content ideas are relevant and usable
- [ ] Sentiment analysis is accurate
- [ ] Executive summary captures key points
- [ ] All relations/links working

### Issues Found
[List any errors, unexpected behavior, or quality concerns]

### Next Steps
[What needs to be fixed or improved]
```

---

## Emergency Rollback Procedure

If scenario breaks production:

1. **Immediately disable scenario** (toggle OFF)
2. **Check execution history** - identify first failure
3. **Revert to last working version:**
   - Go to Scenario settings → History
   - Restore previous version
4. **Test restored version** with single call
5. **Re-enable** once confirmed working
6. **Debug issue** in separate test scenario

---

## Support Resources

**Make.com Help:**
- Documentation: make.com/en/help
- Community: community.make.com
- Support: make.com/en/help/contact-support

**OpenRouter:**
- Docs: openrouter.ai/docs
- Status: openrouter.ai/status
- Discord: openrouter.ai/discord

**Fathom:**
- API Docs: fathom.video/api-docs
- Support: support@fathom.video

**Notion:**
- API Docs: developers.notion.com
- Community: notion.so/help/community
- Status: status.notion.so
