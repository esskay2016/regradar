# RegRadar Architecture

## System Overview

RegRadar is an Agentic AI system that automatically monitors federal regulatory sources, analyzes them with Claude AI, and surfaces priority items through an interactive dashboard.

## Data Flow
RSS Feeds (4 sources)
↓
n8n Workflow Trigger (daily 7am)
↓
Merge Node (combine all feeds)
↓
Filter Node (remove blanks)
↓
Claude API (AI analysis)
↓
Keyword Override Logic (urgency check)
↓
Airtable (store structured data)
↓
v0 Dashboard (display & interact)

## Components

### 1. RSS Feed Ingestion (n8n)
- **CFPB:** https://www.consumerfinance.gov/activity-log/feed/
- **Federal Reserve:** https://www.federalreserve.gov/feeds/press_all.xml
- **NACHA:** https://www.nacha.org/rss.xml
- **PCISSC:** https://www.pcisecuritystandards.org/feed/

Triggers daily at 7am EST via n8n scheduler.

### 2. Data Filtering (n8n Filter Node)
Removes items where:
- Title is empty AND content is empty
- Prevents empty records from reaching Claude (saves API cost)

### 3. AI Analysis (Claude API)
Claude analyzes each regulatory update and returns:

```json
{
  "summary": "Executive summary of the update (3-5 sentences)",
  "product_areas": "Payments, Deposits, Lending, etc.",
  "urgency": "Action Required | Monitor Closely | Informational",
  "so_what": "Implications for product teams (2 sentences)"
}
```

### 4. Urgency Override Logic (n8n Function Node)
Keyword checking happens in the Airtable Urgency field:

Keywords that trigger "Action Required":
- enforcement action, civil penalty, final rule
- consent order, cease and desist
- violation, supervisory action, corrective action
- UDAAP, deceptive marketing, unfair practices
- examination findings, deficiencies
- unsafe or unsound practices, compliance failures

If any keyword found in Claude's analysis → override to "Action Required"

### 5. Data Storage (Airtable)
Table: "Regulatory Updates"

Fields:
- **Title** (text) - Press release headline
- **Source** (text) - CFPB, Federal Reserve, etc.
- **Published Date** (date) - ISO format, typecast ON
- **Summary** (long text) - Claude summary
- **Product Areas** (text) - Comma-separated list
- **Urgency** (text) - Action Required, Monitor Closely, Informational
- **So What** (long text) - Product team implications
- **Source URL** (URL) - Link to original announcement
- **Raw Content** (long text) - Original feed content

### 6. Frontend Dashboard (v0.dev)
React component that:
- Fetches data from Airtable API
- Displays records as interactive cards
- Filters by: Agency, Product Area, Urgency
- Sorts by: Date (newest/oldest), Urgency (high-low), Agency (A-Z)
- Auto-refreshes every 5 minutes
- Navy blue banking theme

## n8n Workflow Nodes (in order)

1. **RSS Feed Triggers (4x)**
   - Each connects to different RSS source
   - Manual Trigger (for testing)
   
2. **Merge Node**
   - Mode: Append
   - Inputs: 5 (4 RSS + 1 manual)
   
3. **Filter Node**
   - Condition: title is not empty OR contentSnippet is not empty
   
4. **Message a Model (Claude)**
   - Model: Claude (Anthropic)
   - System prompt: [see claude/system-prompt.md]
   
5. **Airtable - Create a Record**
   - Base: RegRadar
   - Table: Regulatory Updates
   - Field mapping: [see airtable/SCHEMA.md]

## Execution Flow

### Daily Automated Run (7am EST)

n8n Scheduler triggers
→ All 4 RSS feeds fetch latest items
→ Merge combines into single stream
→ Filter removes blanks
→ Claude analyzes each item (API call per item)
→ Keyword override logic applied
→ Records saved to Airtable
→ Dashboard queries Airtable, displays updates

### Manual Test Run
Manual Trigger sends test data
→ Same flow as above (skips RSS feeds)
→ Useful for testing Claude prompt and Airtable mapping

## Performance & Costs

- **n8n:** Self-hosted or free cloud tier
- **Claude API:** ~$0.01-0.05 per item (depends on content length)
- **Airtable:** Free tier includes ~1,200 records/month
- **v0.dev:** Free tier sufficient for dashboard
- **Execution time:** ~30-60 seconds per run (4 feeds, ~5-10 items per feed)

## Error Handling

- Empty feed items are filtered before Claude analysis
- Airtable field mapping includes fallbacks for missing data
- Dashboard handles API rate limits with exponential backoff
- n8n logs all executions for debugging

## Future Architecture Enhancements

1. **Add Scraper-Based Sources** - OCC, FinCEN, SEC (for sources without RSS)
2. **Historical Archive Lookup** - User-triggered searches for past 3-24 months
3. **Email Digest** - Summarize daily updates and send to stakeholders
4. **Slack Integration** - Post high-urgency items to team Slack
5. **Custom Keywords** - UI to let users configure their own urgency keywords
6. **Multi-Language Support** - Translate summaries for global teams

