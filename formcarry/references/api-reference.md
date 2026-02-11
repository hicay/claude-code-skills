# Formcarry REST API Reference

Programmatic form management and submission retrieval.

## Authentication

All API requests require an `api_key` header. Find your key in Dashboard → Integrations.

```
Header: api_key: YOUR_API_KEY
```

### Verify authentication

```
GET https://formcarry.com/api/auth
```

```json
// Response 200
{
  "status": "success",
  "message": "You are successfully authenticated, please pass your api_key in every requests header."
}
```

---

## Forms API

### Create a form

```
PUT https://formcarry.com/api/form
```

**Required parameters:**

| Parameter | Type | Description |
|---|---|---|
| `name` | string | Form name/identifier |

**Optional parameters:**

| Parameter | Type | Description |
|---|---|---|
| `email` | string | Notification recipient(s), comma-separated |
| `returnUrl` | string | Redirect URL after successful submission |
| `failUrl` | string | Redirect URL after failed submission |
| `returnParams` | boolean | Include submission data in redirect query string |
| `googleRecaptcha` | string | Google reCAPTCHA secret key |
| `webhook` | string | Webhook URL for real-time submission forwarding |
| `retention` | boolean | Save submissions to database (default: true) |

**Thank You Page parameters:**

| Parameter | Type | Values |
|---|---|---|
| `thankYouPage_theme` | string | `"Fancy"`, `"Formal"`, `"Sweet"` |
| `thankYouPage_headline` | string | Custom headline |
| `thankYouPage_message` | string | Custom message body |
| `thankYouPage_mode` | string | `"Dark"`, `"Light"`, `"System"` |
| `thankYouPage_returnText` | string | Button text (default: "Return") |
| `thankYouPage_returnUrl` | string | Button destination URL |

**Self Email parameters (notifications to form owner):**

| Parameter | Type | Description |
|---|---|---|
| `selfEmail_theme` | string | `"Fancy"`, `"Formal"`, `"Custom"` |
| `selfEmail_code` | string | HTML template (when theme is "Custom") |
| `selfEmail_from` | string | Sender email address |
| `selfEmail_subject` | string | Email subject line |
| `selfEmail_sendOnSpam` | boolean | Send notification for spam submissions |

**Auto-Response parameters (email to submitter):**

| Parameter | Type | Description |
|---|---|---|
| `autoResponse` | boolean | Enable auto-response |
| `autoResponse_from` | string | Sender email address |
| `autoResponse_theme` | string | `"Fancy"`, `"Formal"`, `"Custom"` |
| `autoResponse_subject` | string | Subject line |
| `autoResponse_message` | string | Message (Fancy/Formal themes) |
| `autoResponse_code` | string | HTML template (Custom theme) |

**Example:**

```javascript
const res = await fetch("https://formcarry.com/api/form", {
  method: "PUT",
  headers: {
    "api_key": "YOUR_API_KEY",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Contact Form",
    email: "team@company.com",
    returnUrl: "https://company.com/thanks",
    webhook: "https://hooks.slack.com/services/xxx",
  }),
});
```

**Response:**

```json
{
  "code": 200,
  "title": "Created a new form",
  "message": "Your form was successfully created",
  "type": "success",
  "formUrl": "https://formcarry.com/s/XXXXXXX"
}
```

---

### Delete a form

```
DELETE https://formcarry.com/api/form/{formId}
```

```javascript
await fetch("https://formcarry.com/api/form/FORM_ID", {
  method: "DELETE",
  headers: { "api_key": "YOUR_API_KEY" },
});
```

**Response:**

```json
{
  "code": 200,
  "title": "Good job mate",
  "message": "Your form was successfully deleted.",
  "type": "success"
}
```

---

## Submissions API

### List submissions

```
GET https://formcarry.com/api/form/{formId}/submissions
```

**Query parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `limit` | number | 25 | Submissions per page (max: 50) |
| `page` | number | 1 | Page number |
| `sort` | string | `createdAt:-1` | Sort field and order. Format: `field:order` (1=asc, -1=desc) |
| `filter` | string | — | Comma-separated filters (see below) |

**Available filters:**

| Filter | Format | Description |
|---|---|---|
| Date range | `date:N` | Submissions from last N days |
| Has attachments | `attachments:true` | Only submissions with files |
| No attachments | `attachments:false` | Only submissions without files |
| Is spam | `spam:true` | Only spam-flagged submissions |
| Not spam | `spam:false` | Only legitimate submissions |
| Field contains | `fieldname_contains:value` | Custom field contains value |
| Field excludes | `fieldname_notcontains:value` | Custom field doesn't contain value |

**Example:**

```javascript
// Get last 7 days of non-spam submissions, sorted newest first
const res = await fetch(
  "https://formcarry.com/api/form/FORM_ID/submissions?limit=50&page=1&filter=date:7,spam:false&sort=createdAt:-1",
  {
    headers: { "api_key": "YOUR_API_KEY" },
  }
);

const data = await res.json();
```

**Response:**

```json
{
  "form": "FORM_ID",
  "results": 25,
  "pagination": {
    "current_page": 1,
    "previous_page": null,
    "next_page": 2,
    "total_page": 5,
    "total_submissions": 100
  },
  "submissions": [
    {
      "_id": "SUBMISSION_ID",
      "form": "FORM_ID",
      "createdAt": "2025-01-15T10:30:00.000Z",
      "updatedAt": "2025-01-15T10:30:00.000Z",
      "fields": [
        { "name": "name", "value": "John Doe" },
        { "name": "email", "value": "john@example.com" },
        { "name": "message", "value": "Hello!" }
      ]
    }
  ]
}
```

**Pagination fields:**

| Field | Description |
|---|---|
| `current_page` | Current page number |
| `previous_page` | Previous page (null if first) |
| `next_page` | Next page (null if last) |
| `total_page` | Total pages |
| `total_submissions` | Total submission count |

---

## Common API Patterns

### Create form + set up everything programmatically

```javascript
const API_KEY = "YOUR_API_KEY";

async function createForm(config) {
  const res = await fetch("https://formcarry.com/api/form", {
    method: "PUT",
    headers: {
      "api_key": API_KEY,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(config),
  });

  return res.json();
}

// Contact form with webhook + auto-response
const form = await createForm({
  name: "Website Contact",
  email: "team@company.com, ceo@company.com",
  returnUrl: "https://company.com/thanks",
  webhook: "https://hooks.slack.com/services/xxx",
  autoResponse: true,
  autoResponse_from: "noreply@company.com",
  autoResponse_subject: "Thanks for reaching out!",
  autoResponse_message: "We received your message and will get back within 24 hours.",
  thankYouPage_theme: "Fancy",
  thankYouPage_headline: "Message Received",
  thankYouPage_message: "We'll be in touch soon.",
});

console.log(form.formUrl); // Use this as your form action
```

### Export all submissions

```javascript
async function exportAllSubmissions(formId) {
  const allSubmissions = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const res = await fetch(
      `https://formcarry.com/api/form/${formId}/submissions?limit=50&page=${page}&filter=spam:false`,
      { headers: { "api_key": API_KEY } }
    );

    const data = await res.json();
    allSubmissions.push(...data.submissions);

    hasMore = data.pagination.next_page !== null;
    page++;
  }

  return allSubmissions;
}
```

### Search submissions by field value

```javascript
// Find all submissions where email contains "@company.com"
const res = await fetch(
  `https://formcarry.com/api/form/FORM_ID/submissions?filter=email_contains:@company.com`,
  { headers: { "api_key": API_KEY } }
);
```

---

## Error Responses

| Code | Meaning |
|---|---|
| 200 | Success |
| 401 | Invalid or missing API key |
| 404 | Form not found |
| 422 | Validation failed (see `errors` field) |
| 429 | Rate limited |
| 500 | Server error |
