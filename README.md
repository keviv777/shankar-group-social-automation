# 📱 AI Social Media Automation — Shankar Group

> End-to-end AI content generation + auto-posting to Instagram & LinkedIn, built with n8n · Gemini · Leonardo AI · Google Sheets

![Status](https://img.shields.io/badge/status-production--ready-brightgreen)
![Stack](https://img.shields.io/badge/stack-n8n%20%7C%20Gemini%20%7C%20Leonardo%20AI-blue)
![Platforms](https://img.shields.io/badge/platforms-Instagram%20%7C%20LinkedIn-purple)
![Cost](https://img.shields.io/badge/cost-free%20tier%20tools-orange)

---

## 📌 Overview

A fully automated social media content pipeline that:
- **Generates** platform-specific captions (Instagram, LinkedIn, Facebook, YouTube) via Gemini 1.5 Flash
- **Creates** AI images using Leonardo AI (Phoenix model)
- **Posts** directly to Instagram and LinkedIn via their respective APIs
- **Logs** every execution to Google Sheets (success + error tracking)
- **Handles errors** gracefully with separate error routing and Sheets logging

Triggered by a single webhook — one JSON payload generates and publishes a complete multi-platform content package.

---

## 🏗️ Architecture

```
[Webhook — topic + brief + brand_voice + target_audience]
        │
        ▼
[Validate Input]
        │
        ▼
[Gemini 1.5 Flash] ──── generates captions + image prompt ────► [Error Handler]
        │
        ▼
[Parse Gemini Output] ── strips markdown, validates JSON
        │
        ▼
[Leonardo AI — Create Image] ──────────────────────────────────► [Error Handler]
        │
        ▼
[Wait 25 seconds] ── polling delay for image generation
        │
        ▼
[Leonardo AI — Fetch Image] ── retrieves final image URL
        │
        ▼
[Extract Image URL]
        │
        ├──────────────────────────────────┐
        ▼                                  ▼
[Instagram Create Media]          [LinkedIn Post]
        │
        ▼
[Instagram Publish Post]
        │
        ▼
[Merge Results]  ◄──── [Log Errors to Sheets]
        │
        ▼
[Log to Google Sheets]
        │
        ▼
[Respond 200 OK]
```

---

## 📸 Live Demo

### n8n Workflow — Successful Execution (ID#692, 1m 7.272s)
![n8n Workflow](docs/workflow-execution.png)

### API Responses — Live Test Results
![API Responses](docs/api-responses.png)

### FastAPI Server — Running Logs (200 OK)
![FastAPI Logs](docs/fastapi-logs.png)

---

## 🛠️ Tech Stack

| Component | Tool | Cost |
|-----------|------|------|
| Workflow Orchestration | n8n (self-hosted or cloud) | Free |
| AI Caption Generation | Google Gemini 1.5 Flash | Free (15 req/min) |
| AI Image Generation | Leonardo AI — Phoenix model | Free (150 tokens/day) |
| Instagram Posting | Meta Graph API | Free |
| LinkedIn Posting | LinkedIn API (OAuth2) | Free |
| Logging | Google Sheets | Free |
| Hosting | Cloudflare Tunnel (for local n8n) | Free |

---

## 📁 Project Structure

```
shankar-group-social-automation/
├── workflow/
│   └── AI_Social_Media_Final_Clean.json     # Importable n8n workflow
├── docs/
│   ├── workflow-execution.png               # n8n execution screenshot
│   ├── api-responses.png                    # API test response screenshot
│   └── fastapi-logs.png                     # Server logs screenshot
├── .env.example
└── README.md
```

---

## 🔁 Workflow Nodes — Full Breakdown

| # | Node | Action |
|---|------|--------|
| 1 | Webhook — Topic Input | Receives POST with topic, brief, brand_voice, target_audience |
| 2 | Validate Input | Checks required fields are present |
| 3 | Gemini — Generate Content | HTTP Request to Gemini 1.5 Flash API |
| 4 | Parse Gemini Output | Strips markdown fences, parses JSON captions |
| 5 | Leonardo AI — Create Image | Submits image generation request |
| 6 | Wait for Image Generation | 25-second pause for rendering |
| 7 | Leonardo AI — Fetch Image | Retrieves completed image URL |
| 8 | Extract Image URL | Pulls CDN URL from response |
| 9 | Instagram — Create Media | Uploads image + caption to Instagram container |
| 10 | Instagram — Publish Post | Publishes the media container |
| 11 | LinkedIn — Publish Post | Posts image + caption to LinkedIn |
| 12 | Merge Results | Combines Instagram + LinkedIn status |
| 13 | Log to Google Sheets | Writes all data to Sheet1 |
| 14 | Log Errors to Sheets | Writes failures to Errors sheet |
| 15 | Respond Success 200 | Returns full JSON result to caller |
| 16 | Respond Error 500 | Returns error details on failure |

---

## 🚀 Setup Guide

### Step 1 — Get API Keys

**Gemini API** (free)
```
1. Go to https://aistudio.google.com/
2. Click Get API Key → Create API Key
3. Copy key (starts with AIza...)
```

**Leonardo AI** (free — 150 tokens/day)
```
1. Go to https://leonardo.ai/ → Sign up
2. Settings → API Access → Create New Key
3. Model ID used: aa77f04e-3eec-4034-9c07-d0f619684628 (Phoenix)
```

**Meta Graph API** (Instagram)
```
1. Create Meta Developer App at developers.facebook.com
2. Add Instagram Graph API product
3. Get Instagram Account ID + Long-Lived Access Token (60-day)
Required permissions: instagram_basic, instagram_content_publish,
                      pages_show_list, pages_read_engagement
```

**LinkedIn API**
```
1. Create app at linkedin.com/developers/apps
2. Request: Share on LinkedIn + Sign In with LinkedIn (OpenID)
3. Generate access token with scopes: w_member_social, openid, profile, email
4. Get Person ID from GET /v2/userinfo (sub field)
```

### Step 2 — Google Sheets Setup

Create a spreadsheet with two tabs:

**Sheet1** (success log) — columns:
```
Request ID | Timestamp | Topic | Caption Base | Hashtags | CTA |
Image Prompt | Image URL | Instagram Caption | LinkedIn Caption |
Instagram Status | Instagram Post ID | Instagram Error |
LinkedIn Status | LinkedIn Post ID | LinkedIn Error | Overall Status
```

**Errors** sheet — columns:
```
Request ID | Timestamp | Topic | Error Message | Failed Node | Status
```

### Step 3 — Import & Configure n8n Workflow

```
1. n8n → Workflows → Import from File → AI_Social_Media_Final_Clean.json
2. Replace placeholders in these nodes:
   - Gemini node: add your API key to the URL
   - Instagram nodes: replace INSTAGRAM_ACCOUNT_ID and META_ACCESS_TOKEN
   - LinkedIn node: replace ACCESS_TOKEN and PERSON_ID
   - Google Sheets nodes: replace YOUR_GOOGLE_SHEET_ID
3. Add credentials: Google Sheets OAuth2, Leonardo AI (Header Auth)
4. Activate workflow → copy Production Webhook URL
```

### Step 4 — Test

```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "AI tools for small businesses",
    "brief": "How AI can help SMBs save time and cost",
    "brand_voice": "educational and approachable",
    "target_audience": "small business owners in India"
  }'
```

### Expected Response
```json
{
  "success": true,
  "request_id": "REQ_1715165820000",
  "status": "ALL_SUCCESS",
  "topic": "AI tools for small businesses",
  "image_url": "https://cdn.leonardo.ai/...",
  "instagram": { "status": "SUCCESS", "post_id": "17841..." },
  "linkedin": { "status": "SUCCESS", "post_id": "urn:li:share:..." }
}
```

---

## 🔐 Environment Variables

```env
GEMINI_API_KEY=your_gemini_api_key
LEONARDO_API_KEY=your_leonardo_api_key
INSTAGRAM_ACCOUNT_ID=17841...
META_ACCESS_TOKEN=your_long_lived_meta_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
LINKEDIN_PERSON_ID=your_linkedin_sub_value
GOOGLE_SHEET_ID=your_sheet_id
```

---

## 🐛 Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Gemini returns invalid JSON | LLM wrapped output in markdown | Parse node handles this; lower temperature to 0.5 if persistent |
| Leonardo image not ready | Generation took >25s | Increase wait node to 40 seconds |
| Instagram 401 error | Meta token expired (60-day limit) | Regenerate long-lived token |
| LinkedIn 403 Forbidden | Missing `w_member_social` scope | Regenerate token with correct scopes |
| Sheets logging fails | OAuth expired or column mismatch | Re-authenticate + verify column headers exactly |
| Webhook not receiving | Workflow not activated / firewall | Activate workflow + use Cloudflare Tunnel |

---

## 📊 What Gets Generated Per Run

For each topic input, the workflow produces:
- Instagram caption (with hashtags + CTA)
- LinkedIn caption (professional tone)
- Facebook caption
- YouTube title + description
- AI-generated image (Leonardo Phoenix)
- Full log entry in Google Sheets

---

## 👤 Author

**Vivek Kumar Prajapati** — AI Automation Developer
- GitHub: [@keviv777](https://github.com/keviv777)
- Email: vivekprajapati97985@gmail.com
- LinkedIn: [linkedin.com/in/vivek-prajapati-6](https://linkedin.com/in/vivek-prajapati-6/)

> *Built for Shankar Group — n8n Automation Internship Final Selection Round — May 2026*
