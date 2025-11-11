# OpenRouter Setup & Configuration

**Complete guide to setting up OpenRouter for AI processing**

---

## What is OpenRouter?

OpenRouter is an AI gateway that provides unified access to multiple AI models (OpenAI GPT-4, Anthropic Claude, Meta Llama, etc.) through a single API.

**Benefits:**
- **Cost savings:** 20-40% cheaper than direct OpenAI API
- **Unified interface:** One API key for all models
- **Flexibility:** Switch models without code changes
- **Better rate limits:** Higher throughput than direct APIs
- **Fallback support:** Auto-failover if primary model is down

---

## Account Setup

### Step 1: Create Account

1. Go to: https://openrouter.ai
2. Click **"Sign up"** (top right)
3. Sign up with:
   - Email + Password, or
   - Google / GitHub account
4. Verify email if required

---

### Step 2: Add Credits

**OpenRouter is pay-as-you-go (no subscriptions)**

1. Go to: https://openrouter.ai/account
2. Click **"Credits"** tab
3. Click **"Purchase Credits"**
4. Recommended starting amount: **$10**
   - Processes ~50-100 calls depending on transcript length
   - Never expires
   - Refundable if unused

**Payment methods:**
- Credit/debit card
- PayPal (in some regions)

**Cost per call estimate:**
- Content Ideas (GPT-4o-mini): ~$0.10-0.15
- Intelligence Analysis (Claude 3.5): ~$0.25-0.35
- Executive Summary (GPT-4o-mini): ~$0.10-0.15
- **Total per call: $0.45-0.65**

**Monthly at 20 calls:** $9-13

---

### Step 3: Generate API Key

1. Go to: https://openrouter.ai/keys
2. Click **"Create Key"**
3. Configure:
   - **Name:** "Fathom Automation Production"
   - **Limit (optional):** Set monthly spend limit ($20 recommended)
   - **Allowed models (optional):** Leave empty (all models)
4. Click **"Create"**
5. Copy API key:
   - Format: `sk-or-v1-xxxxxxxxxxxxxxxxxxxxxxxxxxxx`
   - **Save immediately** - shown only once!
6. Save to `/config/.env`:
   ```
   OPENROUTER_API_KEY=sk-or-v1-your-key-here
   ```

---

### Step 4: Create Test Key (Optional)

For development/testing:

1. Create second key: "Fathom Automation Test"
2. Set limit: $2
3. Use cheaper models (GPT-3.5-turbo)
4. Save separately in `.env.test`

---

## Model Selection

### Recommended Models

**For Content Generation (Creative Writing):**
- **Primary:** `openai/gpt-4o-mini`
  - Cost: $0.15 / 1M input tokens
  - Speed: Fast (2-4 seconds)
  - Quality: Excellent for content drafting

- **Alternative:** `anthropic/claude-3-haiku`
  - Cost: $0.25 / 1M input tokens
  - Speed: Very fast (1-2 seconds)
  - Quality: Good, more concise style

**For Intelligence Analysis (Deep Reasoning):**
- **Primary:** `anthropic/claude-3.5-sonnet`
  - Cost: $3.00 / 1M input tokens
  - Speed: Moderate (4-6 seconds)
  - Quality: Excellent for nuanced analysis

- **Alternative:** `openai/gpt-4o`
  - Cost: $2.50 / 1M input tokens
  - Speed: Moderate (4-5 seconds)
  - Quality: Excellent, slightly more verbose

**For Summarization (Fast & Cheap):**
- **Primary:** `openai/gpt-4o-mini`
  - Cost: $0.15 / 1M input tokens
  - Speed: Fast
  - Quality: Very good for structured output

---

### Model Comparison

| Model | Best For | Cost/1M Tokens | Speed | Output Quality |
|-------|----------|----------------|-------|----------------|
| GPT-4o-mini | Content, summaries | $0.15 | Fast | ⭐⭐⭐⭐ |
| GPT-4o | Complex analysis | $2.50 | Medium | ⭐⭐⭐⭐⭐ |
| Claude 3.5 Sonnet | Intelligence, reasoning | $3.00 | Medium | ⭐⭐⭐⭐⭐ |
| Claude 3 Haiku | High-volume tasks | $0.25 | Very Fast | ⭐⭐⭐⭐ |
| GPT-3.5-turbo | Testing, simple tasks | $0.50 | Fast | ⭐⭐⭐ |

---

## API Configuration

### Authentication Headers

**Every API request needs these headers:**

```json
{
  "Authorization": "Bearer YOUR_API_KEY",
  "Content-Type": "application/json",
  "HTTP-Referer": "https://your-domain.com",
  "X-Title": "Fathom Automation"
}
```

**Required:**
- `Authorization`: Your API key with "Bearer " prefix
- `Content-Type`: Always "application/json"

**Optional but recommended:**
- `HTTP-Referer`: Your website/app URL (for analytics)
- `X-Title`: App name (appears in OpenRouter dashboard)

---

### Request Format

**Basic structure:**

```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "User's prompt here"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 2000
}
```

**Parameters explained:**

**model** (required)
- Format: `provider/model-name`
- Examples: `openai/gpt-4o-mini`, `anthropic/claude-3.5-sonnet`

**messages** (required)
- Array of conversation messages
- System message: Sets AI behavior/persona
- User message: The actual prompt

**temperature** (optional, default: 1.0)
- Range: 0.0 to 2.0
- Lower = more deterministic/focused
- Higher = more creative/random
- Recommended:
  - 0.3 for analysis (intelligence)
  - 0.7 for content generation
  - 0.5 for summaries

**max_tokens** (optional)
- Maximum response length
- Default: Varies by model
- Set to prevent runaway costs
- Recommended: 2000 (sufficient for most content)

**top_p** (optional, default: 1.0)
- Alternative to temperature
- Range: 0.0 to 1.0
- Don't use both temperature and top_p

**response_format** (optional)
- Force specific output format
- `{"type": "json_object"}` for JSON responses
- Useful for structured data extraction

---

### Response Format

**Successful response:**

```json
{
  "id": "gen-xxxxxxxxxxxxx",
  "model": "openai/gpt-4o-mini",
  "object": "chat.completion",
  "created": 1704931200,
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The AI's response text here"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 150,
    "completion_tokens": 200,
    "total_tokens": 350
  }
}
```

**Access response text:**
```
{{response.data.choices[0].message.content}}
```

**Access token usage:**
```
{{response.data.usage.total_tokens}}
```

---

### Error Handling

**Common error codes:**

**401 Unauthorized**
```json
{"error": {"code": 401, "message": "Invalid API key"}}
```
- **Fix:** Check API key in headers
- Verify key hasn't been deleted in OpenRouter dashboard

**402 Payment Required**
```json
{"error": {"code": 402, "message": "Insufficient credits"}}
```
- **Fix:** Add credits to OpenRouter account

**429 Rate Limit**
```json
{"error": {"code": 429, "message": "Rate limit exceeded"}}
```
- **Fix:** Add delay between requests (2-3 seconds)
- Consider upgrading to higher tier

**503 Service Unavailable**
```json
{"error": {"code": 503, "message": "Model temporarily unavailable"}}
```
- **Fix:** Use fallback model or retry after 30 seconds
- Check OpenRouter status page

---

## Make.com Integration

### HTTP Module Configuration

**In Make.com HTTP "Make a request" module:**

**URL:**
```
https://openrouter.ai/api/v1/chat/completions
```

**Method:**
```
POST
```

**Headers:**
```
Name: Authorization
Value: Bearer {{YOUR_API_KEY}}

Name: Content-Type
Value: application/json

Name: HTTP-Referer
Value: https://make.com

Name: X-Title
Value: Fathom Automation
```

**Body type:**
```
Raw
```

**Content type:**
```
JSON (application/json)
```

**Request content:**
```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "System prompt here"},
    {"role": "user", "content": "{{your.variable.here}}"}
  ],
  "temperature": 0.7
}
```

**Parse response:**
- Use JSON parse module on: `{{httpModule.data.choices[0].message.content}}`

---

## Cost Monitoring

### Dashboard Analytics

1. Go to: https://openrouter.ai/activity
2. View:
   - **Requests:** Total API calls
   - **Tokens:** Input + output token usage
   - **Cost:** Breakdown by model
   - **Timeline:** Usage over time

**Review weekly:**
- Total spend vs budget
- Cost per request (should be $0.40-0.65 per call)
- Identify expensive requests (outliers)

---

### Setting Budget Alerts

1. Go to: https://openrouter.ai/keys
2. Edit your API key
3. Set **Limit:** $20 per month
4. When limit reached:
   - API key stops working
   - Email notification sent
   - Prevents surprise bills

**Recommended limits:**
- Production key: $20-50/month
- Test key: $2-5/month

---

### Cost Optimization

**Reduce costs without sacrificing quality:**

**1. Use cheaper models where appropriate:**
- Summarization: GPT-3.5-turbo ($0.50) vs GPT-4o-mini ($0.15) = 70% savings
- Only use Claude 3.5 for complex intelligence analysis

**2. Reduce token usage:**
- Truncate long transcripts (keep last 75%)
- Use Fathom summary instead of full transcript where possible
- Set max_tokens limits

**3. Cache common requests:**
- Don't re-analyze same call twice
- Store AI outputs in Notion
- Only update when new data available

**4. Batch processing:**
- Process multiple short calls in one API request
- Combine related prompts

**Example savings:**
- Before: 3 API calls per call ($0.65)
- After optimization: 2 API calls ($0.40)
- Savings: 38% at 20 calls/month = $5/month

---

## Testing Setup

### Test in OpenRouter Playground

Before using in Make.com, test prompts:

1. Go to: https://openrouter.ai/playground
2. Select model: `openai/gpt-4o-mini`
3. Add system message: Your system prompt
4. Add user message: Sample transcript (use fake data)
5. Click **"Run"**
6. Review output:
   - Is format correct (JSON if expected)?
   - Is quality sufficient?
   - How many tokens used?
7. Iterate on prompt
8. Copy final prompt to Make.com

---

### Test API Key

**Test with cURL:**

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "openai/gpt-4o-mini",
    "messages": [
      {"role": "user", "content": "Say hello!"}
    ]
  }'
```

**Expected response:**
```json
{
  "choices": [
    {"message": {"content": "Hello! How can I help you today?"}}
  ]
}
```

**If error:**
- 401: API key invalid
- 402: No credits
- Check OpenRouter dashboard for details

---

## Advanced Features

### Model Fallback

**Automatically use backup model if primary fails:**

```json
{
  "models": ["openai/gpt-4o-mini", "anthropic/claude-3-haiku"],
  "messages": [...]
}
```

OpenRouter tries first model, falls back to second if unavailable.

---

### Streaming Responses

**For real-time output (not needed for Make.com):**

```json
{
  "model": "openai/gpt-4o-mini",
  "messages": [...],
  "stream": true
}
```

Returns response incrementally (useful for UI applications).

---

### Provider Routing

**Let OpenRouter choose cheapest available model:**

```json
{
  "model": "openrouter/auto",
  "messages": [...]
}
```

Uses cheapest model that meets your requirements.

---

## Security Best Practices

### API Key Security

**Do:**
- ✅ Store in environment variables (`.env`)
- ✅ Use separate keys for prod/test
- ✅ Set spending limits on keys
- ✅ Rotate keys every 3-6 months
- ✅ Delete unused keys immediately

**Don't:**
- ❌ Commit keys to Git
- ❌ Share keys via email/Slack
- ❌ Use same key across projects
- ❌ Hardcode keys in Make.com scenarios

---

### Rate Limiting

**Protect your key from abuse:**

1. Set per-key spending limits
2. Monitor usage daily for anomalies
3. Use Make.com's built-in rate limiting
4. Add delays between bulk operations

---

## Troubleshooting

### Issue: Response is markdown instead of JSON

**Symptom:**
```
```json
{"key": "value"}
```
```

**Fix:**
Update prompt: "Return ONLY valid JSON with no markdown code blocks. Do not include ```json or ```. Output raw JSON only."

---

### Issue: Response is cut off mid-sentence

**Symptom:** Content ends abruptly

**Cause:** Hit max_tokens limit

**Fix:**
- Increase max_tokens to 2000-4000
- Or use model with longer context (Claude 3.5 supports 200k tokens)

---

### Issue: Inconsistent output format

**Symptom:** Sometimes JSON, sometimes text

**Fix:**
1. Add `"response_format": {"type": "json_object"}` to request
2. Make prompt more explicit: "You must respond with valid JSON only"
3. Lower temperature (0.3 instead of 0.7)

---

### Issue: High costs per request

**Symptom:** Spending >$1 per call

**Debug:**
1. Check token usage in OpenRouter activity log
2. Identify which prompt uses most tokens
3. Common causes:
   - Very long transcripts (>10k words)
   - Multiple retries due to errors
   - Using expensive model unnecessarily

**Fix:**
- Truncate transcripts to 5000 words max
- Use cheaper models where quality sufficient
- Add error handling to prevent retries

---

## Support Resources

**OpenRouter Documentation:**
https://openrouter.ai/docs

**Model documentation:**
- OpenAI: https://platform.openai.com/docs/models
- Anthropic: https://docs.anthropic.com/claude/docs

**OpenRouter Discord:**
https://discord.gg/openrouter

**Status page:**
https://status.openrouter.ai

---

## Next Steps

Once OpenRouter is set up:

1. ✅ API key created and saved to `.env`
2. ✅ Credits added ($10+)
3. ✅ Test API key with cURL
4. ✅ Models selected (GPT-4o-mini + Claude 3.5)
5. ✅ Budget alerts configured
6. → Move to: `docs/02-makecom-workflow-guide.md`
