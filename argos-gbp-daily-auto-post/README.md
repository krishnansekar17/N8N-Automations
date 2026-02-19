# 🚀 Argos GBP Daily Auto-Post — n8n Workflow

**Fully automated Google Business Profile posting system** that publishes product images with AI-generated captions twice daily. Zero manual effort after setup.

![n8n](https://img.shields.io/badge/n8n-Workflow-orange?logo=n8n)
![Google Business Profile](https://img.shields.io/badge/Google%20Business%20Profile-API-blue?logo=google)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4O--MINI-green?logo=openai)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## What It Does

This n8n workflow automatically:

1. **Picks the next unpublished image** from a Google Sheet content calendar
2. **Generates a unique AI caption** using OpenAI (GPT-4O-MINI) with seasonal awareness, mythology references, and brand voice
3. **Uploads the photo** to the GBP Photos gallery (permanent)
4. **Creates a "What's New" post** with the AI caption + Learn More CTA button
5. **Updates the sheet** with PUBLISHED status, timestamp, and the generated caption

Runs at **10 AM + 6 PM daily** on autopilot.

---

## Workflow Architecture

```
[Schedule Trigger] → [Google Sheets: Get Pending Row] → [IF: Has Content?]
    → TRUE  → [OpenAI: Generate Caption] → [HTTP: Upload Photo to GBP]
             → [HTTP: Create GBP Post] → [Google Sheets: Mark Published]
    → FALSE → (stop — no more content)
```

**7 nodes total** | **625 images** | **312 days of automated content** | **$0 platform fees**

---

## Node Breakdown

| # | Node | Type | What It Does |
|---|------|------|-------------|
| 1 | Daily 10AM & 6PM Post | Schedule Trigger | Cron-based dual daily trigger |
| 2 | Get Next Pending Row | Google Sheets | Fetches first row where status = PENDING |
| 3 | Has Pending Image? | IF | Checks if content exists (with type conversion) |
| 4 | Generate AI Caption | OpenAI | Blog-style caption with fragrance notes + seasonal context |
| 5 | Upload Photo to GBP | HTTP Request | POSTs image to GBP Photos gallery via API |
| 6 | Create GBP Post | HTTP Request | Creates What's New post with CTA link |
| 7 | Mark as Published | Google Sheets | Updates status, timestamp, caption, API response |

---

## Prerequisites

Before importing this workflow, you need:

### Google Cloud Console
- A Google Cloud project with these **APIs enabled**:
  - Google My Business API
  - My Business Account Management API
  - My Business Business Information API
  - My Business Lodging API
  - My Business Place Actions API
  - My Business Q&A API
  - My Business Verifications API
  - Google Sheets API
  - Google Drive API

### GBP API Quota (Critical!)
- **Default quota is ZERO.** You must request access at:
  [https://support.google.com/business/contact/api_default](https://support.google.com/business/contact/api_default)
- Approval takes 1-3 business days

### OAuth 2.0 Credentials
- Create OAuth 2.0 Client ID (Web Application type)
- Add your n8n callback URL as authorized redirect URI:
  `https://your-n8n-instance.com/rest/oauth2-credential/callback`
- Also add `https://developers.google.com/oauthplayground` (needed to find Account/Location IDs)

### Account & Location IDs
Use [OAuth 2.0 Playground](https://developers.google.com/oauthplayground) with your credentials to call:
```
GET https://mybusinessaccountmanagement.googleapis.com/v1/accounts
GET https://mybusinessbusinessinformation.googleapis.com/v1/accounts/{ACCOUNT_ID}/locations
```

### OpenAI API Key
- Account at [platform.openai.com](https://platform.openai.com)
- GPT-4O-MINI costs ~$0.001-0.003 per caption (~$0.15/month at 2 posts/day)

---

## Setup Instructions

### 1. Import the Workflow
- Download `Argos GBP Daily Auto-Post.json`
- In n8n: **Settings** → **Import from File** → Select the JSON

### 2. Create 3 Credentials in n8n

| Credential | Type | Used By |
|-----------|------|---------|
| GBP OAuth2 | Google OAuth2 API (generic) | Upload Photo + Create Post nodes |
| Google Sheets | Google Sheets OAuth2 (built-in) | Get Pending Row + Mark Published nodes |
| OpenAI | OpenAI API | Generate AI Caption node |

### 3. Update These Values
After importing, you **must** update these with your own:

- **Nodes 5 & 6 (HTTP URLs):** Replace the Account ID and Location ID in the API URLs
- **Node 2 & 7:** Select your Google Sheet document
- **Schedule Trigger:** Adjust timezone and times as needed

### 4. Prepare Your Content Calendar
Create a Google Sheet with tab name `Image Queue` and these columns:

| Column | Header | Description |
|--------|--------|-------------|
| A | row_id | Sequential number |
| B | fragrance_name | Product name |
| C | image_filename | File name |
| D | drive_file_id | Google Drive file ID |
| E | image_url | `https://drive.google.com/uc?export=download&id={FILE_ID}` |
| F | post_caption | Existing notes (AI uses as context) |
| G | category | PRODUCT |
| H | post_type | BOTH |
| I | status | PENDING |
| J | published_date | (filled by workflow) |
| K | gbp_response | (filled by workflow) |
| L | cta_url | Product page URL |

### 5. Test & Activate
- Test each node individually first (click node → Test step)
- Once all nodes pass, toggle **Active** and save

---

## Key Technical Fixes

Issues I solved during the build (so you don't have to):

| Problem | Solution |
|---------|----------|
| IF node "Wrong type" error | Enable **"Convert types where required"** toggle |
| AI captions break JSON with quotes/newlines | Wrap the Create Post body in **`JSON.stringify()`** instead of raw JSON |
| Apps Script timeout with 600+ images | Use **batch `setValues()`** instead of row-by-row writing |
| Can't find GBP Account/Location IDs | Use **OAuth 2.0 Playground** with custom credentials (not API Explorer) |

---

## Content Preparation Scripts

I used Google Apps Scripts (Extensions → Apps Script in the Sheet) to prepare the content:

1. **`listAllFragranceImages()`** — Scans Drive subfolders, populates sheet with file IDs and download URLs (batch operation)
2. **`updateUrlsAndCaptions()`** — Fuzzy-matches folder names to product database, fixes product URLs, generates template captions
3. **`shuffleByFragrance()`** — Round-robin interleaves images so the same product doesn't post 20 days in a row

---

## AI Caption Prompt Highlights

The OpenAI system prompt generates blog-style captions with:
- Mythology-inspired storytelling (Greek myths behind each fragrance)
- Seasonal awareness (date passed dynamically)
- Anti-AI writing rules (no em dashes, no overused words like "delve", "tapestry", "elevate")
- 1400 character limit (GBP allows 1500)
- Rotating opening styles: questions, bold statements, sensory descriptions

---

## Results

- ✅ 625 images queued across 20+ products
- ✅ 2 posts/day = 312 days (~10 months) of automated content
- ✅ Each caption uniquely AI-generated
- ✅ Photos added to permanent GBP gallery + expiring Updates feed
- ✅ ~$0.15/month OpenAI cost
- ✅ Zero ongoing maintenance

---

## Adapting for Your Business

This workflow works for **any business with a Google Business Profile**. To adapt:

1. Replace the Google Sheet data with your products/images
2. Update the OpenAI system prompt for your brand voice
3. Update the GBP Account ID and Location ID in Nodes 5 & 6
4. Adjust the schedule trigger times for your timezone

---

## License

MIT — Use it, modify it, share it. A star ⭐ on the repo is appreciated!

---

## Built By

**Krishnan S** — Senior Digital Marketing Executive & AI Automation Specialist

🔗 [YouTube: AI with Krishnan](https://youtube.com/@aiwithkrishnan)

Built with Claude AI assistance for architecture planning, debugging, and documentation.
