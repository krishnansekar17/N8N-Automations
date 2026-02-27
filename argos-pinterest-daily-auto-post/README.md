# 📌 Argos Pinterest Daily Auto-Post - n8n Workflow

**Fully automated Pinterest posting system** that publishes product images with AI-generated SEO titles, descriptions, hashtags, and alt text four times daily. Zero manual effort after setup.

![n8n](https://img.shields.io/badge/n8n-Workflow-FF6D5A) ![Pinterest](https://img.shields.io/badge/Pinterest-API_v5-BD081C) ![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991) ![License](https://img.shields.io/badge/License-MIT-green)

---

## What It Does

This n8n workflow automatically:

1. **Picks the next unpublished image** from a Google Sheet content queue
2. **Matches it to a product database** with real fragrance notes, style, and occasion data using fuzzy matching
3. **Generates a unique AI pin** using OpenAI (GPT-4o) with SEO-optimized title, description with hashtags, and alt text
4. **Posts the pin** to the correct Pinterest board via Pinterest API v5
5. **Updates the sheet** with PUBLISHED status, pin ID, AI content, and timestamp

Runs at **12 AM + 6 AM + 12 PM + 6 PM daily** on autopilot.

## Workflow Architecture

```
[Schedule Trigger] → [Google Sheets: Get Pending Row] → [Google Sheets: Get Fragrance DB]
  → [Code: Match Fragrance] → [IF: Has Pending Row?]
    → TRUE  → [OpenAI: Generate Pin Content] → [Code: Parse AI Response]
             → [HTTP: Create Pinterest Pin] → [Code: Check Result]
             → [Google Sheets: Mark Published]
    → FALSE → (stop — no more content)
```

10 nodes total | 625 images | 22 boards | ~156 days of automated content | $0 platform fees

## Node Breakdown

| # | Node | Type | What It Does |
|---|------|------|-------------|
| 1 | Run 4x Daily | Schedule Trigger | Cron-based trigger every 6 hours |
| 2 | Get Next Pending Row | Google Sheets | Fetches first row where status = PENDING |
| 3 | Get Fragrance Database | Google Sheets | Loads all 22 products with real notes |
| 4 | Match Fragrance | Code | Fuzzy matches queue row to product DB with apostrophe normalization |
| 5 | Has Pending Row? | IF | Checks if content exists (uses .first() for paired item fix) |
| 6 | Generate Pin Content | OpenAI | SEO title + description + alt text via GPT-4o |
| 7 | Parse AI Response | Code | Extracts and validates JSON from AI output |
| 8 | Create Pinterest Pin | HTTP Request | POSTs pin to Pinterest API v5 with Bearer token |
| 9 | Check Pin Result | Code | Validates API response, sets PUBLISHED or ERROR |
| 10 | Mark as PUBLISHED | Google Sheets | Updates status, pin_id, title, description, timestamp |

## Prerequisites

Before importing this workflow, you need:

### Pinterest Developer Account
- Pinterest Business Account with claimed/verified domain
- Developer App at [developers.pinterest.com](https://developers.pinterest.com)
- **Standard Access approved** (Trial only works with sandbox — real pins need Standard)
- OAuth token with scopes: `boards:read`, `boards:write`, `pins:read`, `pins:write`, `user_accounts:read`
- Token expires every **30 days** — set a calendar reminder to refresh

### Google Sheets
One spreadsheet with two tabs:

**Tab 1 — Pinterest Queue** (one row per image):

| Column | Description |
|--------|-------------|
| row_id | Unique ID: PIN-0001, PIN-0002, etc. |
| fragrance_name | Product name as in catalog |
| image_filename | Original filename from Drive |
| drive_file_id | Google Drive file ID |
| image_url | Public URL: `https://drive.google.com/uc?export=download&id=FILE_ID` |
| board_name | Target Pinterest board name |
| board_id | Pinterest board ID (from GET /v5/boards) |
| pin_title | _(auto-filled by AI)_ |
| pin_description | _(auto-filled by AI)_ |
| alt_text | _(auto-filled by AI)_ |
| product_url | Shopify product page URL |
| status | Set all to `PENDING` |
| pin_id | _(auto-filled after posting)_ |
| published_date | _(auto-filled after posting)_ |

**Tab 2 — Fragrance Database** (one row per product):

| Column | Description |
|--------|-------------|
| fragrance_name | Full official product name |
| short_name | Comma-separated aliases for matching |
| top_notes | Top/opening notes |
| middle_notes | Heart/middle notes |
| base_notes | Base/dry-down notes |
| style | Style keywords |
| best_for | Recommended occasions |
| description | Brand description (AI context, not copied) |

### Other Requirements
- **OpenAI API key** with GPT-4o access
- **Google Drive** images shared as "Anyone with the link"
- **n8n instance** (self-hosted or cloud)

## Pinterest Standard Access (Critical!)

Trial access cannot post real pins. Apply for upgrade with a **60-90 second demo video** showing:

1. Full OAuth flow (authorization URL → consent → redirect with code → token exchange)
2. Working n8n integration (workflow execution → successful sandbox pin creation)
3. Google Sheet updating to PUBLISHED

Upload to YouTube (unlisted) and submit through the developer portal. Review takes 1-5 business days.

## AI Content System

The v3 prompt uses **3 random variables** per execution for creative variety:

- **7 content angles** — Gift, Seasonal, Occasion, Lifestyle, Discovery, Sensory, Mythology
- **10 title formulas** — Question, Scenario, Problem-Solution, Sensory, Listicle, Seasonal, Gift, Mythology, Bold Claim, Curiosity
- **5 description patterns** — Story Opener, Sensory Scene, Direct Confident, Question Answer, For Who

**= 350 possible combinations** so no two pins feel the same.

### Anti-Generic Rules
- Never starts with "Discover", "Experience", "Exude", or "Celebrate"
- 26 banned AI words (delve, tapestry, captivating, embark, etc.)
- Uses ONLY real product notes from database — never invents
- Title first 30 chars = hook | Description 220-232 chars sweet spot | 3-4 hashtags

### Sample Output

| Field | Example |
|-------|---------|
| Title | `Tired of Ordinary Scents? Meet Perseus Triumphant` |
| Description | `Born from the myth of Perseus, this bold fragrance opens with bergamot and saffron. Rose and leather wrap your spirit for unforgettable evenings. Shop now at argosfragrances.com` + hashtags |
| Alt Text | `A square bottle of Argos Perseus Triumphant with dark leather texture and elegant design` |

## Setup

1. Import the `.json` file in n8n: **Import from File**
2. Add your **Google Sheets** credential (OAuth)
3. Add your **OpenAI** credential (API key)
4. Update the **Pinterest Bearer token** in the HTTP Request node
5. Update the **Google Sheets document ID** in all Sheets nodes
6. Paste the **system prompt and user prompt** into the OpenAI node
7. Test with one manual execution
8. Toggle the workflow **Active**

## License

MIT — Free to use, modify, and share.
