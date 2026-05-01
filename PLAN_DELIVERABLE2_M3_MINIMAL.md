# PLAN_DELIVERABLE2_M3_MINIMAL.md

**Objective:** Connect existing static web (localhost:5050) to N8N to achieve 10 points  
**Web changes:** MINIMAL (only 2 form handlers)  
**Architecture:** Web frontend (HTML/CSS/JS) + N8N backend (5 nodes)

---

## 📊 THE 10 POINTS (Rubric M3)

```
1. Capture in Google Sheets          → 2 pts  (N8N Webhook → Google Sheets)
2. Generation with AI (guide)        → 2 pts  (N8N → Groq/Gemini/Claude)
3. Google Docs creation              → 2 pts  (N8N → Google Docs API)
4. Email sending                     → 2 pts  (N8N → Gmail/Outlook)
5. Business notification (Slack)     → 2 pts  (N8N → Slack webhook)
                                      ───────
                                      10 pts
```

---

## 🏗️ CURRENT ARCHITECTURE

```
Web (localhost:5050)
├── index.html (home + CTA form)
├── courses.html (catalog)
├── newsletter.html (signup)
└── about.html

Current JavaScript:
- Forms have client-only handlers
- Submit → "✓ Subscribed" in UI, nothing else
```

---

## 🔄 MINIMAL CHANGES NEEDED (VERY SMALL)

### Change #1: `index.js` (CTA form line)

**BEFORE (current):**
```javascript
document.getElementById('cta-form').addEventListener('submit', function(e) {
  e.preventDefault();
  var btn = document.getElementById('cta-btn');
  var input = document.getElementById('cta-email');
  btn.textContent = '✓ Subscribed';
  btn.disabled = true;
  input.value = '';
});
```

**AFTER (with N8N):**
```javascript
document.getElementById('cta-form').addEventListener('submit', function(e) {
  e.preventDefault();
  var btn = document.getElementById('cta-btn');
  var input = document.getElementById('cta-email');
  
  // Collect data
  var email = input.value;
  var exam = document.querySelector('[data-selected-exam]')?.dataset.selectedExam || 'Deliverable 2 Signup';
  
  // POST to N8N webhook
  fetch('https://your-webhook-n8n.com/webhook', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      email: email,
      exam: exam,
      source: 'home-cta'
    })
  })
  .then(response => {
    btn.textContent = '✓ Subscribed';
    btn.disabled = true;
    input.value = '';
    input.placeholder = 'Check your inbox';
  })
  .catch(error => {
    btn.textContent = '✕ Error, retry';
    console.error('Error:', error);
  });
});
```

### Change #2: `newsletter.js` (newsletter forms)

**BEFORE (current):**
```javascript
function handleSubmit(inputId, btnId) {
  var input = document.getElementById(inputId);
  var btn = document.getElementById(btnId);
  btn.textContent = '✓ Subscribed';
  btn.disabled = true;
  input.value = '';
}
```

**AFTER (with N8N):**
```javascript
function handleSubmit(inputId, btnId) {
  var input = document.getElementById(inputId);
  var btn = document.getElementById(btnId);
  var email = input.value;
  
  fetch('https://your-webhook-n8n.com/webhook', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      email: email,
      exam: 'M365news',
      source: 'newsletter-page'
    })
  })
  .then(response => {
    btn.textContent = '✓ Subscribed';
    btn.disabled = true;
    input.value = '';
    input.placeholder = 'Check your inbox';
  })
  .catch(error => {
    btn.textContent = '✕ Error, retry';
  });
}
```

---

## 🔗 N8N STRUCTURE (5 NODES = 10 POINTS)

```
NODE 1: Webhook Trigger
  ↓ (receives POST with email, exam, source)
  
NODE 2: Google Sheets
  └─ New row: [Timestamp | Email | Exam | Source]
  
NODE 3: AI (Groq/Gemini/Claude)
  └─ System Prompt: "You are Microsoft instructor. Generate guide with 3 tips for {exam}"
  └─ Output: content_generated
  
NODE 4: Google Docs
  └─ Create new Doc with:
     - Title: "{exam} - Guide for {email}"
     - Content: {content_generated}
     - Share publicly (read-only)
  └─ Output: doc_url
  
NODE 5a: Gmail (Email)
  └─ Send to: {email}
  └─ Subject: "Your personalized {exam} guide + discount code"
  └─ Body HTML: link to Google Docs + discount code
  
NODE 5b: Slack (Notification)
  └─ Channel: #leads or #support
  └─ Message: "🆕 Lead: {email} - {exam}"
```

---

## 📋 CHANGES BY FILE

| File | Changes | Lines |
|------|---------|-------|
| `index.html` | NONE | 0 |
| `courses.html` | NONE | 0 |
| `newsletter.html` | NONE | 0 |
| `about.html` | NONE | 0 |
| `shared.css` | NONE | 0 |
| `index.css` | NONE | 0 |
| `courses.css` | NONE | 0 |
| `newsletter.css` | NONE | 0 |
| `about.css` | NONE | 0 |
| `shared.js` | NONE | 0 |
| `courses.js` | NONE | 0 |
| `about.js` | NONE | 0 |
| `index.js` | **EDIT** | ~20 lines (CTA form) |
| `newsletter.js` | **EDIT** | ~15 lines (signup functions) |

**Total:** 2 files, ~35 lines of changes. **No HTML or CSS touched.**

---

## 🚀 WORKFLOW (PHASED)

### PHASE A: Setup N8N (FIRST)

1. **Create workflow in N8N:**
   - Webhook Trigger (expose public URL)
   - 5 nodes connected (Sheets → AI → Docs → Email → Slack)
   - Test with Postman

2. **Get public webhook URL:** `https://hook.n8n.cloud/webhook/abc123xyz`

3. **Document expected JSON structure:**
   ```json
   {
     "email": "user@example.com",
     "exam": "AZ-900",
     "source": "home-cta"
   }
   ```

### PHASE B: Update web (VERY FAST)

1. **Claude Code:** Open `index.js` and `newsletter.js`
2. **Replace:** Form handlers with POST to N8N webhook
3. **Test local:** `python http.server → localhost:5050`

### PHASE C: Integration testing

1. **Fill form on web** → POST to N8N
2. **N8N receives** → Creates Sheets row → Generates AI → Creates Doc → Sends email → Alerts Slack
3. **Verify:** Sheets has row, Email arrived, Docs created, Slack notified

### PHASE D: Video + Documentation

1. **Record Loom (5 min):**
   - Show form on web
   - Fill in data
   - Wait for email
   - Show Google Sheets row
   - Show Google Doc generated
   - Show Slack notification
   - Show N8N workflow complete

2. **Documentation:**
   - How to setup N8N
   - How to setup credentials (Google, Groq, Gmail, Slack)
   - Workflow structure
   - Changes to web

---

## 📝 FILES TO GENERATE/UPDATE

### Updates (minimal changes):
```
✏️ assets/js/index.js          (POST fetch in CTA form)
✏️ assets/js/newsletter.js     (POST fetch in signup forms)
```

### New delivery documents:
```
📄 DELIVERABLE2_README.md       (Complete setup instructions)
📄 N8N_WORKFLOW_SPEC.md         (Detail of 5 nodes)
📄 ERRORS_FIXED.md              (Bugs found + fixes)
📄 SETUP_GUIDE.md               (Step-by-step credentials)
```

---

## 🎯 CHECKLIST BEFORE DELIVERY

- [ ] N8N workflow works (manual test with Postman)
- [ ] Web POST goes to N8N webhook
- [ ] Google Sheets receives data
- [ ] AI generates personalized content
- [ ] Google Doc auto-created
- [ ] Email sent with link to Doc + code
- [ ] Slack notified of new lead
- [ ] Web at localhost:5050 works without errors
- [ ] `index.js` and `newsletter.js` updated
- [ ] Loom video (5 min) recorded
- [ ] README and documentation complete
- [ ] ERRORS_FIXED.md documented
- [ ] Lighthouse 85+ on web

---

## 🔐 CREDENTIALS NEEDED

| Service | Variable | Where |
|---------|----------|-------|
| Google Sheets | `SHEETS_ID` + `SHEETS_TAB` | N8N node config |
| Google Docs | `GOOGLE_SERVICE_ACCOUNT_KEY` | N8N node config |
| Gmail | `GMAIL_SERVICE_ACCOUNT_KEY` | N8N node config |
| Groq/Gemini | `API_KEY` | N8N node config |
| Slack | `WEBHOOK_URL` | N8N node config |

---

## ⏱️ ESTIMATED TIMELINE

| Phase | Time | Notes |
|-------|------|-------|
| A (N8N setup) | 2-3h | More if first time |
| B (Web update) | 30min | Claude Code → 2 files |
| C (Testing) | 1h | Validate complete flow |
| D (Video + docs) | 1-2h | Record Loom, write README |
| **TOTAL** | **4.5-6.5h** | Without blockers |

---

## 🎬 NEXT STEP

1. **Do you have access to N8N?** (Cloud or self-hosted)
2. **Do you have credentials ready?**
   - Google Workspace (Sheets, Docs, Gmail)
   - Groq/Gemini API key
   - Slack workspace + webhook

3. **If YES:** Start with **PHASE A (N8N workflow)**
4. **If NO:** First help with credential setup

---

## 📌 DIFFERENCES vs. PREVIOUS PLAN

| Before | Now |
|--------|-----|
| Rebuild in Next.js | Use existing web (minimal changes) |
| 8-12 hours | 4.5-6.5 hours |
| ~500 lines code | ~35 lines code |
| Custom backend | N8N (visual, proven) |
| Higher risk bugs | Minimal changes = fewer errors |

