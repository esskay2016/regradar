# Airtable Schema

## Base: RegRadar

### Table: Regulatory Updates

Complete field definitions and configuration.

| Field Name | Type | Configuration | Example |
|---|---|---|---|
| **Title** | Single line text | — | "CFPB Enforcement Action Against Major Bank" |
| **Source** | Single line text | — | "CFPB" |
| **Published Date** | Date | Typecast: ON | "2026-05-14" |
| **Summary** | Long text | — | "The CFPB announced a civil penalty and enforcement action..." |
| **Product Areas** | Single line text | — | "Payments, Deposits, Consumer Protection" |
| **Urgency** | Single line text | — | "Action Required" |
| **So What** | Long text | — | "Product teams must review compliance procedures..." |
| **Source URL** | URL | — | "https://example.com/announcement" |
| **Raw Content** | Long text | — | "Original RSS feed snippet" |

## Field Mapping from n8n

How n8n populates each field:

```javascript
Title → {{ $('Merge').item.json.title }}
Source → {{ $('Merge').item.json.source?.name || 'Unknown' }}
Published Date → {{ new Date($('Merge').item.json.pubDate).toISOString().split('T')[0] }}
Summary → [extracted from Claude output, SUMMARY section]
Product Areas → [extracted from Claude output, PRODUCT_AREAS section]
Urgency → [extracted from Claude output with keyword override logic]
So What → [extracted from Claude output, SO_WHAT section]
Source URL → {{ $('Merge').item.json.link }}
Raw Content → {{ $('Merge').item.json.contentSnippet }}
```

## Views (Optional)

Create these views in Airtable for easier browsing:

### View 1: "Action Required"
- Filter: Urgency = "Action Required"
- Sort: Published Date (newest first)
- Use to: Prioritize high-risk updates

### View 2: "By Product Area"
- Group by: Product Areas
- Sort: Published Date (newest first)
- Use to: See which areas are most active

### View 3: "By Source"
- Filter: Source (pick one)
- Sort: Published Date (newest first)
- Use to: Track specific regulator

## Typecast Setting (Important!)

When creating the **Published Date** field:

1. Click the field
2. Click the gear icon → "Customize field type"
3. Enable **"Typecast"** toggle
4. This allows n8n to automatically convert date strings to proper date format

If you don't enable this, dates may not save correctly.

## Sample Data

Here's what a populated record looks like:
Title: "CFPB Enforcement Action Against Major Bank for Deceptive Practices"
Source: "CFPB"
Published Date: 2026-05-14
Summary: "The Consumer Financial Protection Bureau announced a civil penalty and enforcement action against a major financial institution for deceptive marketing practices related to credit card fee disclosures."
Product Areas: "Payments, Card Issuance, Consumer Protection, Deceptive Practices"
Urgency: "Action Required" (overridden from Claude's "Monitor Closely" due to "enforcement action" keyword)
So What: "Product and compliance teams must immediately review current credit card disclosures and marketing materials. Ensure all fee information is clearly visible and avoid hiding conditions in fine print."
Source URL: "https://www.consumerfinance.gov/news/cfpb-enforcement-action..."
Raw Content: "The CFPB submitted a report to the Office of Management and Budget cataloging criminal regulatory offenses..."

## Capacity & Limits

- **Airtable Free Tier:** 1,200 records per base per month
- **With RegRadar:** ~5-10 records per day = ~150-300/month (well within limits)
- **Upgrade:** When you need more history or additional bases

## Backup Strategy

Airtable data is safe, but good practice:
1. Periodically export Regulatory Updates table as CSV
2. Store in GitHub repo (docs folder)
3. Set monthly reminder
