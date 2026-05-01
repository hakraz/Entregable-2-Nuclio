# N8N_WORKFLOW_SPEC.md (English)

**Workflow name:** `Deliverable2_M3_LeadMagnet`  
**Objective:** 5 integrated nodes (Sheets + AI + Docs + Email + Slack) = 10 points  
**Input:** POST webhook with `{email, exam, source}`  
**Output:** Row in Sheets + Google Doc + Email + Slack notification

---

## 🎯 EXECUTIVE SUMMARY

```
[Webhook Trigger] 
  ↓ (POST: email, exam, source)
[Google Sheets Node] → Saves row
  ↓
[AI Node] → Generates personalized guide
  ↓
[Google Docs Node] → Creates document
  ↓
[Gmail Node] → Sends email
  ↓
[Slack Node] → Internal notification
```

---

## 📍 NODE 1: WEBHOOK TRIGGER

**Type:** `Webhook`  
**Method:** `POST`  
**Resulting URL:** `https://your-n8n.com/webhook/deliverable2` (N8N provides this)

**Expected input (JSON):**
```json
{
  "email": "user@example.com",
  "exam": "AZ-900",
  "source": "home-cta"
}
```

**Configuration:**
- ✅ `POST` (method)
- ✅ Path: `/webhook/deliverable2` (custom)
- ✅ Authenticate: OFF (public)

**Output variable names for use below:**
- `$json.email`
- `$json.exam`
- `$json.source`

---

## 📍 NODE 2: GOOGLE SHEETS

**Type:** `Google Sheets`  
**Operation:** `Append`  
**Credential:** Google Workspace account (with Sheets permissions)

**Configuration:**

| Field | Value |
|-------|-------|
| Spreadsheet ID | `YOUR_SHEETS_ID` (from URL: `/spreadsheets/d/{ID}`) |
| Sheet | `Leads` (or your tab name) |
| Columns | Map below ↓ |

**Column mapping (A, B, C, D):**
```
A. Timestamp          → {{ new Date().toISOString() }}
B. Email             → {{ $json.email }}
C. Exam              → {{ $json.exam }}
D. Source            → {{ $json.source }}
```

**Output for next node:**
- `$json` contains row_id if needed

---

## 📍 NODE 3: AI (CONTENT GENERATION)

**Type:** `OpenAI` or `Groq` or `Google Gemini` (choose one)  
**Credential:** API key from your provider

### Option A: Groq (recommended, cheaper)

**Type:** `Groq`  
**Model:** `mixtral-8x7b-32768`  
**Temperature:** 0.7  
**Max tokens:** 1500

**System Prompt:**
```
You are an expert Microsoft certification instructor.

The user is preparing for the exam: {EXAM_NAME}

Generate a quick guide (maximum 400 words) with:
1. Three key topics you must study
2. One specific tip for each topic
3. Resource recommendation (mention the Udemy course)
4. Discount code: SPRINGISHERE26 (50% off)

Format in clean Markdown.
```

**User Message (dynamic):**
```
Generate guide for: {{ $json.exam }}
User email: {{ $json.email }}
```

**Output:**
- `$json.choices[0].message.content` → save to variable `content_generated`

### Option B: OpenAI

**Type:** `OpenAI`  
**Model:** `gpt-4-turbo` (or gpt-3.5-turbo if budget)  
**Temperature:** 0.7

(Structure identical, change credential)

### Option C: Google Gemini

**Type:** `Google Gemini`  
**Model:** `gemini-1.5-pro`

(Structure identical)

---

## 📍 NODE 4: GOOGLE DOCS

**Type:** `Google Docs`  
**Operation:** `Create document`  
**Credential:** Google Workspace (same as Sheets)

**Configuration:**

| Field | Value |
|-------|-------|
| Document Title | `{{ $json.exam }} - Guide for {{ $json.email }}` |
| Folder | (leave empty = root, or select specific folder) |
| Content | (see below) |

**Content (formatted):**
```html
{{ $json.exam }} - Personalized Guide

Generated for: {{ $json.email }}
Date: {{ new Date().toISOString() }}

---

{{ $node["AI Node"].json.choices[0].message.content }}

---

Discount code: SPRINGISHERE26
Valid until: end of month
Applicable to: All courses on Udemy
```

**Output variables for next node:**
- `$json.webViewLink` → Public URL of Doc (important for email)
- `$json.id` → Doc ID

**Important permissions:**
- Verify Doc is "Viewer access" (public read without login)
- If needed, add: `anyone with link can view`

---

## 📍 NODE 5a: GMAIL (EMAIL)

**Type:** `Gmail`  
**Operation:** `Send email`  
**Credential:** Gmail account (with app-password permissions)

**Configuration:**

| Field | Value |
|-------|-------|
| To | `{{ $json.email }}` |
| Subject | `Your personalized {{ $json.exam }} guide + discount code` |
| Body Type | `HTML` |
| HTML Body | (see below) |

**HTML Body Template:**

```html
<h2>Hello {{ $json.email }},</h2>

<p>Here is your personalized guide for the <strong>{{ $json.exam }}</strong> exam.</p>

<p>
  <a href="{{ $node["Google Docs"].json.webViewLink }}" style="
    display: inline-block;
    padding: 12px 24px;
    background: #0ea5e9;
    color: white;
    text-decoration: none;
    border-radius: 6px;
    font-weight: bold;
  ">
    Open your guide →
  </a>
</p>

<hr>

<h3>Your discount code</h3>
<p style="font-size: 20px; font-weight: bold; color: #0ea5e9;">
  SPRINGISHERE26
</p>
<p>50% off any course. Valid until end of month.</p>

<hr>

<p>Questions? Reply to this email.</p>

<p>
  — The certxpro.net team<br>
  <em>Not affiliated with Microsoft Corporation</em>
</p>
```

**Output:**
- Success if `status: 200`

---

## 📍 NODE 5b: SLACK

**Type:** `Slack`  
**Operation:** `Send message to channel`  
**Credential:** Slack webhook URL (or Slack app OAuth)

**Configuration:**

| Field | Value |
|-------|-------|
| Channel | `#leads` (or your channel) |
| Text | (see below) |

**Message Text:**

```
🆕 New lead captured

📧 Email: {{ $json.email }}
📚 Exam: {{ $json.exam }}
📍 Source: {{ $json.source }}
⏰ Timestamp: {{ new Date().toISOString() }}

→ View in Sheets: [Link to your Sheets]
```

**Output:**
- Message sent to Slack

---

## 🔗 NODE CONNECTION (Diagram)

```
[1. Webhook]
    ↓
[2. Sheets (Append)] → Saves row
    ↓
[3. AI (Groq/OpenAI)] → Generates content_generated
    ↓
[4. Google Docs (Create)] → Creates Doc, output webViewLink
    ↓
    ├─→ [5a. Gmail] → Sends email
    └─→ [5b. Slack] → Notifies
```

**Important:** Nodes 5a and 5b can execute in **parallel** (no dependency on each other).

---

## 🧪 TESTING THE WORKFLOW

### Test 1: Webhook only

```bash
curl -X POST https://your-n8n.com/webhook/deliverable2 \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "exam": "AZ-900",
    "source": "test"
  }'
```

**Expected:** 200 OK + workflow executes

### Test 2: Validate Sheets

Open your Google Sheets → Verify new row with:
- Timestamp, email, exam, source

### Test 3: Validate Google Docs

Search Google Drive for new document: `AZ-900 - Guide for test@example.com`

### Test 4: Validate Email

Check inbox of `test@example.com` → Should have email with:
- Correct subject
- Link to Google Docs
- Discount code visible

### Test 5: Validate Slack

Check Slack channel → Should have message with lead data

---

## 🔒 PERMISSIONS & CREDENTIALS

| Service | Permission | Where to get |
|---------|-----------|--------------|
| Google Sheets | `Editor` on Sheets | Share URL → Access |
| Google Docs | `Editor` on Docs | Create folder/permissions |
| Gmail | `Send as` in Gmail | OAuth token |
| Groq/OpenAI | API Key | console.groq.com / openai.com |
| Slack | Webhook URL | api.slack.com → Webhooks |

**Note:** All credentials go **in N8N Credentials**, NOT hardcoded.

---

## 🚨 COMMON ERRORS

### Error 1: "Cannot read property 'email' of undefined"
**Cause:** Webhook didn't receive POST correctly  
**Fix:** Verify web is doing `fetch()` with correct JSON structure

### Error 2: "Google Sheets append failed"
**Cause:** SHEETS_ID incorrect or insufficient permissions  
**Fix:** Verify ID in URL, add N8N account as Editor

### Error 3: "OpenAI API key invalid"
**Cause:** API key expired or no permissions  
**Fix:** Regenerate at openai.com, verify quota

### Error 4: "Gmail: Permission denied"
**Cause:** Gmail app-password not configured  
**Fix:** Generate app-password in Google Account Security

### Error 5: "Slack webhook invalid"
**Cause:** URL expired or miscopied  
**Fix:** Regenerate webhook at api.slack.com

---

## 📊 VARIABLE MAPPING (QUICK REFERENCE)

```javascript
// NODE 1 (Webhook): Raw input
$json.email
$json.exam
$json.source

// NODE 2 (Sheets): No transformation, just pass data
// Output: $json (same)

// NODE 3 (AI): Generates content
$node["Groq"].json.choices[0].message.content  // Important variable

// NODE 4 (Docs): Creates Doc, returns URLs
$node["Google Docs"].json.webViewLink  // URL for email (important)
$node["Google Docs"].json.id  // Doc ID

// NODE 5a (Gmail): Sends
// Use webViewLink from Docs

// NODE 5b (Slack): Notifies
// Use email, exam, source from Webhook
```

---

## 🎬 QUICK SETUP CHECKLIST

- [ ] Webhook Trigger created, URL copied
- [ ] Google Sheets connected, SHEETS_ID verified
- [ ] AI connected (Groq/OpenAI/Gemini), API key validated
- [ ] Google Docs connected, permissions "Anyone with link" ON
- [ ] Gmail connected, app-password configured
- [ ] Slack connected, webhook URL valid
- [ ] Nodes connected in order (1→2→3→4→5a, 5b)
- [ ] Variables mapped correctly (`$json.email`, etc)
- [ ] Manual test successful (curl + verifications)
- [ ] Error handling: each node has try/catch or error branch

---

## 📚 RESOURCES

- N8N Docs: https://docs.n8n.io/
- Google Sheets node: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheetsapiupdate/
- OpenAI node: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.openai/
- Gmail node: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.gmail/
- Slack node: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.slack/

