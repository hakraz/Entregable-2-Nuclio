# WEB_UPDATES_MINIMAL.md (English)

**Objective:** Update only 2 JS files to connect to N8N  
**Files to modify:** `assets/js/index.js` and `assets/js/newsletter.js`  
**Files that DON'T touch:** HTML, CSS, other JS  
**Total changes:** ~35 lines

---

## 📋 BEFORE YOU START

1. **Get N8N webhook URL:**
   - You need the workflow in N8N to be **active**
   - Copy the public URL from Webhook Trigger
   - Example: `https://your-n8n-instance.com/webhook/deliverable2`

2. **Validate JSON structure N8N expects:**
   ```json
   {
     "email": "user@example.com",
     "exam": "AZ-900",
     "source": "home-cta"
   }
   ```

3. **Define exam for each page:**
   - Home CTA: Default `"Deliverable 2 Signup"` or user-selected
   - Newsletter page: `"M365news Newsletter"`

---

## 🔧 CHANGE #1: `assets/js/index.js`

**Purpose:** Home CTA form must POST to N8N

**Current file (BEFORE):**
```javascript
document.getElementById('ticker-slot').outerHTML = renderTicker();
mountChrome('home');

document.getElementById('cta-form').addEventListener('submit', function(e) {
  e.preventDefault();
  var btn = document.getElementById('cta-btn');
  var input = document.getElementById('cta-email');
  var fine = document.getElementById('cta-fine');
  btn.textContent = '✓ Subscribed';
  btn.disabled = true;
  btn.style.background = 'var(--accent-a)';
  input.value = '';
  input.placeholder = 'Check your inbox for your discount code';
  fine.textContent = '✓ Check your inbox for the discount code.';
});
```

**REPLACE ENTIRE WITH:**

```javascript
document.getElementById('ticker-slot').outerHTML = renderTicker();
mountChrome('home');

document.getElementById('cta-form').addEventListener('submit', function(e) {
  e.preventDefault();
  
  var btn = document.getElementById('cta-btn');
  var input = document.getElementById('cta-email');
  var fine = document.getElementById('cta-fine');
  
  var email = input.value.trim();
  
  // Minimal validation
  if (!email || !email.includes('@')) {
    btn.textContent = '✕ Invalid email';
    btn.style.color = 'var(--fg)';
    setTimeout(() => {
      btn.textContent = 'Get code';
      btn.style.color = 'inherit';
    }, 2000);
    return;
  }
  
  // Change UI state to "sending"
  btn.textContent = 'Sending...';
  btn.disabled = true;
  
  // POST to N8N webhook
  fetch('https://YOUR_N8N_WEBHOOK_URL_HERE', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      email: email,
      exam: 'Deliverable 2 Signup',
      source: 'home-cta'
    })
  })
  .then(function(response) {
    if (!response.ok) {
      throw new Error('Network error: ' + response.status);
    }
    // Success
    btn.textContent = '✓ Subscribed';
    btn.style.background = 'var(--accent-a)';
    input.value = '';
    input.placeholder = 'Check your inbox for your discount code';
    fine.textContent = '✓ Check your inbox for the discount code.';
  })
  .catch(function(error) {
    console.error('Error:', error);
    btn.textContent = '✕ Error, retry';
    btn.disabled = false;
    setTimeout(() => {
      btn.textContent = 'Get code';
      btn.style.color = 'inherit';
    }, 3000);
  });
});
```

**Key changes:**
1. ✅ `fetch()` to N8N webhook (replace URL)
2. ✅ JSON with `email`, `exam`, `source`
3. ✅ Minimal email validation
4. ✅ States: "Sending..." → "✓ Subscribed" or "✕ Error"
5. ✅ Error handling with `.catch()`

**⚠️ IMPORTANT:**
Replace `https://YOUR_N8N_WEBHOOK_URL_HERE` with your real N8N URL.

---

## 🔧 CHANGE #2: `assets/js/newsletter.js`

**Purpose:** Both newsletter forms (hero + final) must POST to N8N

**Current file (BEFORE):**
```javascript
document.getElementById('ticker-slot').outerHTML = renderTicker();
mountChrome('newsletter');

function handleSubmit(inputId, btnId) {
  var input = document.getElementById(inputId);
  var btn = document.getElementById(btnId);
  btn.textContent = '✓ Subscribed';
  btn.disabled = true;
  btn.style.background = 'var(--accent-a)';
  input.value = '';
  input.placeholder = 'Check your inbox for your discount code';
}

document.getElementById('hero-form').addEventListener('submit', function(e) {
  e.preventDefault();
  handleSubmit('hero-email', 'hero-btn');
});

document.getElementById('final-form').addEventListener('submit', function(e) {
  e.preventDefault();
  handleSubmit('final-email', 'final-btn');
});
```

**REPLACE ENTIRE WITH:**

```javascript
document.getElementById('ticker-slot').outerHTML = renderTicker();
mountChrome('newsletter');

function handleSubmit(inputId, btnId) {
  var input = document.getElementById(inputId);
  var btn = document.getElementById(btnId);
  
  var email = input.value.trim();
  
  // Minimal validation
  if (!email || !email.includes('@')) {
    btn.textContent = '✕ Invalid email';
    btn.style.color = 'var(--fg)';
    setTimeout(() => {
      btn.textContent = 'Subscribe';
      btn.style.color = 'inherit';
    }, 2000);
    return;
  }
  
  // Change UI state to "sending"
  btn.textContent = 'Sending...';
  btn.disabled = true;
  
  // POST to N8N webhook
  fetch('https://YOUR_N8N_WEBHOOK_URL_HERE', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      email: email,
      exam: 'M365news Newsletter',
      source: 'newsletter-page'
    })
  })
  .then(function(response) {
    if (!response.ok) {
      throw new Error('Network error: ' + response.status);
    }
    // Success
    btn.textContent = '✓ Subscribed';
    btn.style.background = 'var(--accent-a)';
    input.value = '';
    input.placeholder = 'Check your inbox for your discount code';
  })
  .catch(function(error) {
    console.error('Error:', error);
    btn.textContent = '✕ Error, retry';
    btn.disabled = false;
    setTimeout(() => {
      btn.textContent = 'Subscribe';
      btn.style.color = 'inherit';
    }, 3000);
  });
}

document.getElementById('hero-form').addEventListener('submit', function(e) {
  e.preventDefault();
  handleSubmit('hero-email', 'hero-btn');
});

document.getElementById('final-form').addEventListener('submit', function(e) {
  e.preventDefault();
  handleSubmit('final-email', 'final-btn');
});
```

**Key changes:**
1. ✅ `handleSubmit()` now contains `fetch()`
2. ✅ Same JSON structure but `exam: 'M365news Newsletter'`
3. ✅ Email validation
4. ✅ States and error handling
5. ✅ Both forms (hero + final) use same logic

**⚠️ IMPORTANT:**
Replace `https://YOUR_N8N_WEBHOOK_URL_HERE` with your real N8N URL.

---

## 📝 INSTRUCTIONS FOR CLAUDE CODE

When you open Claude Code to make these changes:

```
Task: Update 2 JavaScript files to connect to N8N

Context:
- I have a static web at localhost:5050
- Web is complete (HTML/CSS/JS) and working
- I only need to update forms to POST to N8N webhook

Exact changes:

1. File: assets/js/index.js
   - Replace ENTIRE event listener for #cta-form
   - Add fetch() POST to webhook URL (see WEB_UPDATES_MINIMAL.md)
   - Add minimal email validation
   - Add error handling

2. File: assets/js/newsletter.js
   - Replace ENTIRE handleSubmit() function
   - Add fetch() POST to webhook URL (SAME URL as above)
   - Add email validation
   - Add error handling
   - Both forms (#hero-form and #final-form) use same function

Notes:
- DON'T touch HTML
- DON'T touch CSS
- DON'T add libraries (vanilla JS only)
- Use console.error() for debugging
- Webhook URL: [REPLACE WITH YOUR URL]

Give me the 2 complete updated files.
```

---

## 🧪 LOCAL TESTING (Without N8N yet)

Before connecting to real N8N, test structure:

### Test 1: Open web in browser

```bash
cd your-project
python3 -m http.server 5050
# Open http://localhost:5050
```

### Test 2: DevTools → Network

1. Open DevTools (F12)
2. Go to "Network" tab
3. Fill out a form (CTA or Newsletter)
4. You should see a POST request (will fail if webhook URL is placeholder)

### Test 3: Console logs

```javascript
// Before fetch(), add:
console.log('Sending to webhook:', JSON.stringify({...}));

// In catch(), already has:
console.error('Error:', error);
```

---

## 🔗 VARIABLES TO REPLACE

In both files, find and replace:

```
FIND:    https://YOUR_N8N_WEBHOOK_URL_HERE
REPLACE: https://your-n8n-domain.com/webhook/deliverable2
```

**Real examples:**
- Cloud N8N: `https://yourname.app.n8n.cloud/webhook/deliverable2`
- Self-hosted: `https://n8n.yourdomain.com/webhook/deliverable2`

---

## ✅ CHECKLIST BEFORE DELIVERY

- [ ] `index.js` has fetch() in cta-form
- [ ] `newsletter.js` has fetch() in handleSubmit()
- [ ] Both use correct webhook URL
- [ ] Both have email validation
- [ ] Both have error handling
- [ ] Console.error() without errors
- [ ] Network tab shows POST requests
- [ ] No changes to HTML/CSS
- [ ] No new libraries added

---

## 📊 DIFFERENCES BEFORE/AFTER

| Aspect | Before | After |
|--------|--------|-------|
| CTA form | No-op, UI only | POST to N8N |
| Newsletter form | No-op, UI only | POST to N8N |
| Validation | None | Minimal email |
| Error handling | None | try/catch |
| Code lines modified | 15-20 | ~35 |
| HTML/CSS touched | 0 | 0 |

---

## 🎯 END RESULT

After these changes:

1. **User fills form on web**
2. **JavaScript POSTs to N8N webhook**
3. **N8N workflow auto-executes:**
   - Saves to Google Sheets
   - Generates AI content
   - Creates Google Doc
   - Sends email
   - Notifies Slack
4. **User sees "✓ Subscribed" on web**
5. **User receives email with guide + code in inbox**

---

## 🚨 COMMON ERRORS

### Error 1: "Failed to fetch"
**Cause:** Webhook URL incorrect or N8N offline  
**Fix:** Verify URL, confirm N8N is active

### Error 2: "CORS error"
**Cause:** N8N doesn't allow requests from localhost  
**Fix:** In N8N webhook settings, enable CORS

### Error 3: "Network error: 405"
**Cause:** Webhook expects POST but receives GET  
**Fix:** Verify `method: 'POST'` in fetch

### Error 4: "email is not defined"
**Cause:** Input #cta-email doesn't exist in HTML  
**Fix:** Verify HTML has `id="cta-email"`

---

## 📞 NEXT STEPS

1. **Get webhook URL from N8N** (after creating workflow)
2. **Open Claude Code** with this document
3. **Request changes** in index.js and newsletter.js
4. **Local testing** → DevTools Network
5. **End-to-end testing** → Fill form → Verify Sheets/Docs/Email/Slack

