# RegRadar Setup Guide

Step-by-step instructions to get RegRadar running on your system.

## Prerequisites

Before you start, you'll need:

1. **n8n** (workflow automation)
   - Either: Local installation (requires Node.js v18+)
   - Or: n8n Cloud account (free tier available at https://app.n8n.cloud)

2. **Claude API Key**
   - Sign up at https://console.anthropic.com
   - Create an API key
   - Keep it secret (add to .env)

3. **Airtable Account**
   - Free account at https://airtable.com
   - You'll need: Personal Access Token + Base ID

4. **v0.dev Account** (optional, for dashboard)
   - Free account at https://v0.dev
   - Or deploy React component to Vercel/your own server

## Part 1: Set Up Airtable

### 1.1 Create a new Base

1. Log in to Airtable
2. Click **"Create"** → **"Start from scratch"**
3. Name it: **"RegRadar"**
4. Click **"Create base"**

### 1.2 Create the Table

1. In RegRadar base, rename "Table 1" to **"Regulatory Updates"**
2. Delete the default "Name" field

### 1.3 Add Fields

Click the **"+"** button to add these fields in order:

| Field Name | Type | Notes |
|---|---|---|
| Title | Single line text | |
| Source | Single line text | |
| Published Date | Date | Turn ON "Typecast" in advanced options |
| Summary | Long text | |
| Product Areas | Single line text | |
| Urgency | Single line text | |
| So What | Long text | |
| Source URL | URL | |
| Raw Content | Long text | |

Your table should now have 9 fields.

### 1.4 Get Your Airtable Credentials

1. Go to https://airtable.com/account/tokens
2. Click **"Create token"**
3. Name it: "RegRadar"
4. Grant scopes: 
   - `data.records:read`
   - `data.records:write`
   - `schema.bases:read`
5. Copy the token and save it (you'll need it for n8n)
6. Note your **Base ID**: 
   - Open RegRadar base
   - Look at the URL: `https://airtable.com/[BASE_ID]/...`
   - Copy the BASE_ID part

Save these credentials securely (in a .env file, password manager, etc.)

---

## Part 2: Set Up n8n

### 2.1 Access n8n

**Option A: n8n Cloud (Easiest)**
- Go to https://app.n8n.cloud
- Sign up with email
- You're ready to create workflows

**Option B: Local Installation**
```bash
npm install -g n8n
n8n start
# Access at http://localhost:5678
```

### 2.2 Import the RegRadar Workflow

1. In n8n, click **"Create new workflow"**
2. Click the three dots (**"..."**) in the top right
3. Select **"Import from file"**
4. Upload `n8n/workflow-export.json` from this repo
5. The workflow should load with all nodes intact

### 2.3 Add Credentials

The workflow uses 2 credential types:

**Claude API Credentials:**
1. Click any **"Message a Model"** node
2. In the "Anthropic" dropdown, select **"Create New Credential"**
3. Paste your Claude API key
4. Save

**Airtable Credentials:**
1. Click the **"Airtable - Create a Record"** node
2. In the "Airtable" dropdown, select **"Create New Credential"**
3. Paste your Airtable Personal Access Token
4. Save

### 2.4 Configure the Airtable Node

1. Click the **"Airtable - Create a Record"** node
2. Set **Base:** RegRadar (select from dropdown)
3. Set **Table:** Regulatory Updates (select from dropdown)
4. Verify all field mappings are correct (see airtable/SCHEMA.md)

### 2.5 Schedule Daily Execution

1. Click the **first node** (RSS Feed Trigger - CFPB)
2. Scroll down to **"Trigger on"**
3. Select: **"Every day"** at **"07:00 (7am)"**
4. Set timezone to your preferred time zone

### 2.6 Publish the Workflow

1. Click **"Save"** in the top left
2. Click **"Publish"** to activate it
3. The workflow now runs daily at 7am

---

## Part 3: Test the Workflow

### 3.1 Run a Manual Test

1. In your RegRadar workflow, find the **"Manual Trigger"** node at the top
2. Click **"Execute"** (or click the play button)
3. Watch the workflow execute in real-time
4. Check your Airtable base - you should see a new record

### 3.2 Verify the Record

1. Go to your Airtable RegRadar base
2. Open the "Regulatory Updates" table
3. You should see:
   - A new record with test data
   - Summary populated by Claude
   - Product Areas tagged
   - Urgency classified
   - So What filled in

If all fields are populated → **Setup is working!**

---

## Part 4: Deploy the Dashboard (Optional)

### 4.1 Get the Dashboard Code

The dashboard component is in `dashboard/dashboard.jsx`

### 4.2 Option A: Deploy to Vercel (Recommended)

1. Create a GitHub repo if you haven't (or push this repo to GitHub)
2. Go to https://vercel.com
3. Click **"New Project"** → select this repo
4. Vercel auto-detects it as a React project
5. Add environment variables:
REACT_APP_AIRTABLE_TOKEN=your_token_here
REACT_APP_AIRTABLE_BASE_ID=your_base_id_here
6. Click **"Deploy"**
7. Your dashboard is now live at `[project-name].vercel.app`

### 4.3 Option B: Deploy to v0.dev

1. Go to https://v0.dev
2. Create a new project
3. Paste the code from `dashboard/dashboard.jsx`
4. Add your Airtable credentials
5. v0 generates a live preview instantly

### 4.4 Access the Dashboard

Once deployed, you can:
- View all regulatory updates
- Filter by Agency, Product Area, Urgency
- Sort by Date, Urgency, Agency
- Click into records to see full details
- Dashboard auto-refreshes every 5 minutes

---

## Troubleshooting

### "Claude API error"
- Check your API key is valid
- Verify you have credits on your Anthropic account
- Check n8n error logs

### "Airtable connection failed"
- Verify your Personal Access Token (not regular API key)
- Check Base ID is correct
- Verify token scopes include `data.records:write`

### "No records created"
- Check n8n execution logs (click "Executions" tab)
- Verify RSS feeds are returning data
- Test with Manual Trigger first

### "Dashboard showing no data"
- Check Airtable token and Base ID are correct in .env
- Verify dashboard is pointing to correct table
- Check browser console for API errors

---

## Next Steps

1. ✅ Workflow running daily
2. ✅ Records populating in Airtable
3. ✅ Dashboard displaying updates
4. 📌 Customize keywords (edit in Airtable node Urgency field)
5. 📌 Add more regulatory sources (OCC, FinCEN, SEC)
6. 📌 Set up Slack integration for alerts

---

## Need Help?

- Check [docs/ARCHITECTURE.md](ARCHITECTURE.md) for system overview
- Review [n8n/WORKFLOW_SETUP.md](../n8n/WORKFLOW_SETUP.md) for workflow details
- Open an issue on GitHub
