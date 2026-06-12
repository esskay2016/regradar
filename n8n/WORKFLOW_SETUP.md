# n8n Workflow Setup

How to import and configure the RegRadar n8n workflow.

## Files

- `workflow-export.json` - The complete RegRadar workflow (ready to import)

## Importing the Workflow

### Step 1: Access n8n

Go to:
- **Cloud:** https://app.n8n.cloud (login)
- **Local:** http://localhost:5678 (if self-hosted)

### Step 2: Import

1. Click **"Create new workflow"** or **"Import"** button
2. Select **"Import from file"**
3. Choose `workflow-export.json` from this folder
4. Click **"Import"**

The workflow loads with all nodes intact.

## Configuration

### Credentials Setup

The workflow uses 2 credential types that you need to add:

**1. Claude API Credential**
- Click any **"Message a Model"** node
- In the Anthropic dropdown, click **"Create New"**
- Paste your Claude API key from https://console.anthropic.com
- Save

**2. Airtable Credential**
- Click the **"Airtable - Create a Record"** node
- In the Airtable dropdown, click **"Create New"**
- Paste your Airtable Personal Access Token from https://airtable.com/account/tokens
- Save

### Workflow Configuration

**Set Your Base and Table:**
1. Open the **"Airtable - Create a Record"** node
2. In **"Base"** dropdown, select your **"RegRadar"** base
3. In **"Table"** dropdown, select **"Regulatory Updates"**

**Verify Field Mappings:**

The node should auto-populate these fields:
- Title, Source, Published Date, Summary, Product Areas
- Urgency, So What, Source URL, Raw Content

If any are missing, add them manually based on the Airtable schema.

### Schedule Daily Execution

1. Click the first **"RSS Feed Trigger - CFPB"** node
2. In the trigger settings, find **"Trigger on"**
3. Select: **"Every day"**
4. Set time: **"07:00"** (7am)
5. Set timezone: your preferred timezone

### Publish the Workflow

1. Click **"Save"** (top left)
2. Click **"Publish"** to activate it
3. Workflow now runs daily at 7am

## Testing

### Manual Test

1. Find the **"Manual Trigger"** node (first node, top of canvas)
2. Click **"Execute"** button
3. Watch the workflow run
4. Check your Airtable base for the new record

### Debug

If something fails:
1. Click **"Executions"** tab to see logs
2. Red nodes = errors; click to see details
3. Check credentials are valid
4. Verify Airtable base and table names match

## Nodes Explained

| Node | Purpose |
|---|---|
| RSS Feed Trigger (4x) | Fetch from CFPB, Fed, NACHA, PCISSC |
| Manual Trigger | Allows manual testing |
| Merge | Combine all 5 inputs into one stream |
| Filter | Remove blank items |
| Message a Model | Claude API analysis |
| Airtable - Create Record | Save to Airtable |

## Troubleshooting

**"Airtable connection failed"**
- Verify Personal Access Token (not API key)
- Check Base ID and Table name
- Confirm token scopes include `data.records:write`

**"Claude API error"**
- Check API key is valid
- Verify account has credits
- Check token hasn't expired

**"No records created"**
- Check execution logs (red = error)
- Try Manual Trigger first
- Verify RSS feeds are returning data

**"Wrong data in fields"**
- Check field mappings in Airtable node
- Verify Claude prompt in "Message a Model" node
- Review Airtable schema for field names
