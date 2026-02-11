---
name: formcarry
description: Build and deploy forms without a backend using Formcarry as the form endpoint. Use this skill when the user mentions "formcarry," "form backend," "form endpoint," "contact form," "no backend form," "form without server," "collect form submissions," "form API," "HTML form submission," "form handling," "lead capture form," "feedback form," "survey form," "registration form," or wants to add a working form to any website, landing page, or app without building server-side code. Covers HTML forms, AJAX submissions (React, Vue, Fetch, Axios), file uploads, spam protection, webhooks, auto-responders, field validation, Stripe payments, and the Formcarry REST API.
license: MIT
---

# Formcarry — Forms Without a Backend

Formcarry is a form backend that collects submissions from HTML or JavaScript forms with minimal configuration. No server code needed. Setup takes ~3 minutes.

**Endpoint format:** `https://formcarry.com/s/{FORM_ID}`

## Before Building

1. **Does the user have a Formcarry account?** If not, they need one at formcarry.com
2. **What type of form?** Contact, feedback, registration, payment, file upload
3. **Submission method?** HTML (page redirect) or AJAX (no reload)
4. **Framework?** Plain HTML, React, Vue, Next.js, Astro, or other
5. **Features needed?** File uploads, spam protection, auto-responder, webhooks, payments

---

## Quick Start — 3 Steps

### Step 1: Create form in Formcarry dashboard
Dashboard → Create Form → Copy endpoint URL from Setup tab.

### Step 2: Point your form to the endpoint

```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <input type="text" name="name" required>
  <input type="email" name="email" required>
  <textarea name="message"></textarea>
  <button type="submit">Send</button>
</form>
```

### Step 3: Submit and check dashboard
Submissions appear in Formcarry dashboard. Email notifications sent to form owner.

**Critical rule:** Every `<input>`, `<textarea>`, and `<select>` MUST have a `name` attribute. Without it, data won't be collected.

---

## Submission Methods

### Method 1: HTML Form (Page Redirect)

Simplest approach. Form submits, user redirects to thank-you page.

```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <input type="text" name="name" placeholder="Name" required>
  <input type="email" name="email" placeholder="Email" required>
  <textarea name="message" placeholder="Message"></textarea>
  <button type="submit">Send</button>
</form>
```

### Method 2: Fetch API (No Reload)

```javascript
form.addEventListener("submit", async (e) => {
  e.preventDefault();

  const data = Object.fromEntries(new FormData(form));

  const response = await fetch("https://formcarry.com/s/{FORM_ID}", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json",
    },
    body: JSON.stringify(data),
  });

  const result = await response.json();

  if (result.code === 200) {
    // Success
  } else {
    // Error: result.message
  }
});
```

### Method 3: React

```tsx
const [status, setStatus] = useState<"idle" | "sending" | "sent" | "error">("idle");

async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
  setStatus("sending");

  const data = Object.fromEntries(new FormData(e.currentTarget));

  try {
    const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
      method: "POST",
      headers: { "Content-Type": "application/json", "Accept": "application/json" },
      body: JSON.stringify(data),
    });

    const result = await res.json();
    setStatus(result.code === 200 ? "sent" : "error");
  } catch {
    setStatus("error");
  }
}
```

For complete code examples (Vue, Axios, jQuery, XMLHttpRequest, file uploads), see [references/code-examples.md](references/code-examples.md).

---

## Special Hidden Fields

Control behavior with hidden inputs:

| Field | Purpose | Example |
|---|---|---|
| `_next` | Redirect URL after submit | `<input type="hidden" name="_next" value="https://site.com/thanks">` |
| `_gotcha` | Honeypot spam trap | `<input type="hidden" name="_gotcha">` |

**Note:** Dashboard thank-you page URL overrides `_next`. Leave dashboard field empty to use `_next`.

---

## Spam Protection

Three layers, use together for best results:

### 1. Built-in (automatic)
Formcarry's algorithm detects spam automatically. Spam is flagged, not deleted.

### 2. Honeypot field
```html
<input type="hidden" name="_gotcha">
```
Bots fill hidden fields. Humans don't. Submissions with `_gotcha` filled are flagged as spam.

### 3. Google reCAPTCHA
Configure reCAPTCHA secret key in dashboard or via API (`googleRecaptcha` parameter).

**Plan note:** Free plan counts spam toward submission limit. Paid plans don't.

---

## File Uploads

**Requirement:** `enctype="multipart/form-data"` on the form tag.
**Limit:** 10MB per file. Paid plans only.

```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST" enctype="multipart/form-data">
  <input type="text" name="name">
  <input type="file" name="attachment">
  <button type="submit">Send</button>
</form>
```

Multiple files: add `multiple` attribute or use multiple `<input type="file">` elements.

**AJAX file uploads:** Use `FormData` object, do NOT set `Content-Type` header (browser sets it automatically with boundary).

```javascript
const formData = new FormData(formElement);

const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
  method: "POST",
  headers: { "Accept": "application/json" },
  body: formData, // NOT JSON.stringify — send FormData directly
});
```

---

## Field Validations

Configure in dashboard. Six validators:

| Validator | Rule |
|---|---|
| `required` | Field cannot be empty |
| `number` | Must be a number |
| `email` | Must be valid email |
| `url` | Must be valid URL |
| `contains` | Must contain specified value |
| `notContains` | Must NOT contain specified value |

**Error response** (HTTP 422):
```json
{
  "code": 422,
  "status": "error",
  "title": "Validation Failed",
  "errors": {
    "email": { "message": "email is not a valid email", "rule": "email" }
  }
}
```

Handle validation errors by highlighting fields:
```javascript
if (result.code === 422) {
  Object.keys(result.errors).forEach(field => {
    document.querySelector(`[name="${field}"]`).classList.add("error");
  });
}
```

---

## Email Notifications

### Self notifications (to form owner)
Enabled by default. Configure email, subject, and template in dashboard.

### Auto-response (to submitter)
Enable in dashboard. Requires an `email` field in the form. Sends confirmation to the person who submitted.

### Custom email templates
Available on Growth/Premium plans. Use Mustache-style variables:

```
Hi {{name}},

Thanks for reaching out about {{subject}}.

{{#phone}}
We'll also call you at {{phone}}.
{{/phone}}

{{#fallback}}company | your company{{/fallback}}
```

**Variables:** `{{fieldName}}` — maps to form field `name` attributes.
**Conditionals:** `{{#field}}...{{/field}}` — renders only if field has value.
**Fallbacks:** `{{#fallback}}field | default text{{/fallback}}` — default when empty.
**Iteration:** `{{#fields}}{{key}}: {{value}}{{/fields}}` — loop through all fields.

---

## Webhooks

Configure webhook URL in dashboard or via API. Formcarry POSTs submission data to your URL in real-time.

Use cases: Slack notifications, CRM updates, database sync, custom processing.

---

## Stripe Payments

Collect payments alongside form submissions. Configure Stripe integration in dashboard.

---

## REST API

For programmatic form management and submission retrieval.

**Authentication:** Pass `api_key` header in every request. Find key in dashboard → Integrations.

For complete API reference (create/delete forms, list/filter submissions, pagination), see [references/api-reference.md](references/api-reference.md).

---

## Pricing Quick Reference

| Plan | Price | Forms | Submissions/mo | File Storage | Team |
|---|---|---|---|---|---|
| Baby (Free) | $0 | 1 | 50 | Yes | 1 |
| Starter | $5/mo | Unlimited | 500 | 1GB | 3 |
| Basic | $15/mo | Unlimited | 2,000 | 2GB | 5 |
| Premium | $80/mo | Unlimited | 30,000 | 30GB | 20 |
| Enterprise | Custom | Unlimited | Unlimited | Unlimited | Unlimited |

---

## Common Patterns

### Contact form with spam protection
```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <input type="text" name="name" required>
  <input type="email" name="email" required>
  <textarea name="message" required></textarea>
  <input type="hidden" name="_gotcha">
  <input type="hidden" name="_next" value="https://site.com/thanks">
  <button type="submit">Send Message</button>
</form>
```

### Newsletter signup (minimal)
```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <input type="email" name="email" placeholder="your@email.com" required>
  <button type="submit">Subscribe</button>
</form>
```

### Multi-step with JS (collect, then submit)
Collect data across steps in a JS object, then submit all at once via Fetch.

### React hook pattern
```tsx
function useFormcarry(formId: string) {
  const [status, setStatus] = useState<"idle" | "sending" | "sent" | "error">("idle");
  const [errors, setErrors] = useState<Record<string, string>>({});

  async function submit(data: Record<string, string>) {
    setStatus("sending");
    setErrors({});

    try {
      const res = await fetch(`https://formcarry.com/s/${formId}`, {
        method: "POST",
        headers: { "Content-Type": "application/json", "Accept": "application/json" },
        body: JSON.stringify(data),
      });

      const result = await res.json();

      if (result.code === 200) {
        setStatus("sent");
      } else if (result.code === 422) {
        setErrors(Object.fromEntries(
          Object.entries(result.errors).map(([k, v]: any) => [k, v.message])
        ));
        setStatus("error");
      } else {
        setStatus("error");
      }
    } catch {
      setStatus("error");
    }
  }

  return { status, errors, submit };
}
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Fields show as empty | Missing `name` attribute | Add `name` to every input |
| CORS error | Missing `Accept` header | Add `Accept: application/json` |
| File upload fails | Missing `enctype` | Add `enctype="multipart/form-data"` |
| File upload fails (AJAX) | Setting Content-Type manually | Don't set Content-Type, send FormData directly |
| `_next` redirect not working | Dashboard has custom thank-you URL | Clear dashboard thank-you page field |
| Spam not filtered | No protection configured | Add `_gotcha` hidden field + enable reCAPTCHA |
| 422 error | Field validation failed | Check `result.errors` for specific field issues |
