# Formcarry — Complete Code Examples

Every framework and method, copy-paste ready. Replace `{FORM_ID}` with your actual form ID from the Formcarry dashboard.

## Table of Contents

1. [HTML (No JavaScript)](#html)
2. [Fetch API](#fetch-api)
3. [React](#react)
4. [React with File Upload](#react-file-upload)
5. [React Hook (Reusable)](#react-hook)
6. [Vue 3 (Composition API)](#vue-3)
7. [Next.js (App Router)](#nextjs)
8. [Astro](#astro)
9. [Axios](#axios)
10. [jQuery](#jquery)
11. [XMLHttpRequest](#xmlhttprequest)
12. [File Upload (HTML)](#file-upload-html)
13. [File Upload (AJAX)](#file-upload-ajax)
14. [Contact Form (Full Example)](#contact-form)
15. [Newsletter Signup](#newsletter)
16. [Multi-field with Validation Handling](#validation-handling)

---

## HTML

No JavaScript required. Form submits and redirects.

```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <label for="name">Name</label>
  <input type="text" id="name" name="name" required>

  <label for="email">Email</label>
  <input type="email" id="email" name="email" required>

  <label for="message">Message</label>
  <textarea id="message" name="message" required></textarea>

  <!-- Honeypot spam protection -->
  <input type="hidden" name="_gotcha">

  <!-- Custom redirect after submit -->
  <input type="hidden" name="_next" value="https://yoursite.com/thanks">

  <button type="submit">Send</button>
</form>
```

---

## Fetch API

Native browser API — no dependencies.

```javascript
const form = document.getElementById("contact-form");

form.addEventListener("submit", async (e) => {
  e.preventDefault();

  const formData = new FormData(form);
  const data = Object.fromEntries(formData);

  try {
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
      form.reset();
      // Show success message
    } else {
      // Show error: result.message
    }
  } catch (error) {
    // Network error
  }
});
```

---

## React

Functional component with status tracking.

```tsx
import { useState, FormEvent } from "react";

export function ContactForm() {
  const [status, setStatus] = useState<"idle" | "sending" | "sent" | "error">("idle");

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setStatus("sending");

    const formData = new FormData(e.currentTarget);
    const data = Object.fromEntries(formData);

    try {
      const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Accept": "application/json",
        },
        body: JSON.stringify(data),
      });

      const result = await res.json();

      if (result.code === 200) {
        setStatus("sent");
        e.currentTarget.reset();
      } else {
        setStatus("error");
      }
    } catch {
      setStatus("error");
    }
  }

  if (status === "sent") {
    return <p>Thank you! We'll be in touch.</p>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="name" placeholder="Name" required />
      <input type="email" name="email" placeholder="Email" required />
      <textarea name="message" placeholder="Message" required />
      <input type="hidden" name="_gotcha" />

      <button type="submit" disabled={status === "sending"}>
        {status === "sending" ? "Sending..." : "Send"}
      </button>

      {status === "error" && <p>Something went wrong. Please try again.</p>}
    </form>
  );
}
```

---

## React File Upload

Uses `FormData` directly — do NOT set `Content-Type` header.

```tsx
import { useState, useRef, FormEvent } from "react";

export function FileUploadForm() {
  const [status, setStatus] = useState<"idle" | "sending" | "sent" | "error">("idle");
  const formRef = useRef<HTMLFormElement>(null);

  async function handleSubmit(e: FormEvent) {
    e.preventDefault();
    if (!formRef.current) return;
    setStatus("sending");

    const formData = new FormData(formRef.current);

    try {
      const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
        method: "POST",
        headers: { "Accept": "application/json" },
        // Do NOT set Content-Type — browser sets it with multipart boundary
        body: formData,
      });

      const result = await res.json();
      setStatus(result.code === 200 ? "sent" : "error");
    } catch {
      setStatus("error");
    }
  }

  return (
    <form ref={formRef} onSubmit={handleSubmit} encType="multipart/form-data">
      <input type="text" name="name" required />
      <input type="email" name="email" required />
      <input type="file" name="attachment" />
      <button type="submit" disabled={status === "sending"}>
        {status === "sending" ? "Uploading..." : "Submit"}
      </button>
    </form>
  );
}
```

---

## React Hook

Reusable hook for any form.

```tsx
import { useState, useCallback } from "react";

interface FormcarryResult {
  status: "idle" | "sending" | "sent" | "error";
  errors: Record<string, string>;
  submit: (data: Record<string, any>) => Promise<void>;
  reset: () => void;
}

export function useFormcarry(formId: string): FormcarryResult {
  const [status, setStatus] = useState<FormcarryResult["status"]>("idle");
  const [errors, setErrors] = useState<Record<string, string>>({});

  const submit = useCallback(async (data: Record<string, any>) => {
    setStatus("sending");
    setErrors({});

    try {
      const res = await fetch(`https://formcarry.com/s/${formId}`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Accept": "application/json",
        },
        body: JSON.stringify(data),
      });

      const result = await res.json();

      if (result.code === 200) {
        setStatus("sent");
      } else if (result.code === 422) {
        const fieldErrors: Record<string, string> = {};
        for (const [key, val] of Object.entries(result.errors)) {
          fieldErrors[key] = (val as any).message;
        }
        setErrors(fieldErrors);
        setStatus("error");
      } else {
        setStatus("error");
      }
    } catch {
      setStatus("error");
    }
  }, [formId]);

  const reset = useCallback(() => {
    setStatus("idle");
    setErrors({});
  }, []);

  return { status, errors, submit, reset };
}

// Usage:
// const { status, errors, submit } = useFormcarry("YOUR_FORM_ID");
// await submit({ name: "John", email: "john@example.com", message: "Hello" });
```

---

## Vue 3

Composition API with reactive status.

```vue
<script setup lang="ts">
import { ref, reactive } from "vue";

const status = ref<"idle" | "sending" | "sent" | "error">("idle");
const form = reactive({ name: "", email: "", message: "" });

async function handleSubmit() {
  status.value = "sending";

  try {
    const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json",
      },
      body: JSON.stringify(form),
    });

    const result = await res.json();

    if (result.code === 200) {
      status.value = "sent";
      Object.assign(form, { name: "", email: "", message: "" });
    } else {
      status.value = "error";
    }
  } catch {
    status.value = "error";
  }
}
</script>

<template>
  <div v-if="status === 'sent'">
    <p>Thank you! We'll be in touch.</p>
  </div>

  <form v-else @submit.prevent="handleSubmit">
    <input v-model="form.name" type="text" name="name" placeholder="Name" required />
    <input v-model="form.email" type="email" name="email" placeholder="Email" required />
    <textarea v-model="form.message" name="message" placeholder="Message" required />

    <button type="submit" :disabled="status === 'sending'">
      {{ status === "sending" ? "Sending..." : "Send" }}
    </button>

    <p v-if="status === 'error'">Something went wrong. Please try again.</p>
  </form>
</template>
```

---

## Next.js

App Router with Server Action pattern.

```tsx
"use client";

import { useState, FormEvent } from "react";

export function ContactForm() {
  const [status, setStatus] = useState<"idle" | "sending" | "sent" | "error">("idle");

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setStatus("sending");

    const formData = new FormData(e.currentTarget);
    const data = Object.fromEntries(formData);

    try {
      const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Accept": "application/json",
        },
        body: JSON.stringify(data),
      });

      const result = await res.json();
      setStatus(result.code === 200 ? "sent" : "error");
    } catch {
      setStatus("error");
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="name" placeholder="Name" required />
      <input type="email" name="email" placeholder="Email" required />
      <textarea name="message" placeholder="Message" required />
      <input type="hidden" name="_gotcha" />
      <button type="submit" disabled={status === "sending"}>
        {status === "sending" ? "Sending..." : "Send"}
      </button>
      {status === "sent" && <p>Sent!</p>}
      {status === "error" && <p>Error. Try again.</p>}
    </form>
  );
}
```

---

## Astro

Works with any Astro island framework, or plain HTML.

```astro
---
// src/components/ContactForm.astro
// For static sites, use plain HTML form (no JS needed)
---

<form action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <input type="text" name="name" placeholder="Name" required />
  <input type="email" name="email" placeholder="Email" required />
  <textarea name="message" placeholder="Message" required></textarea>
  <input type="hidden" name="_gotcha" />
  <input type="hidden" name="_next" value="https://yoursite.com/thanks" />
  <button type="submit">Send</button>
</form>
```

For AJAX in Astro, use the React or Vue component with `client:load` directive:
```astro
---
import { ContactForm } from "../components/ContactForm";
---
<ContactForm client:load />
```

---

## Axios

```javascript
import axios from "axios";

// Do NOT set Content-Type header — axios handles it
async function submitForm(data) {
  try {
    const { data: result } = await axios.post(
      "https://formcarry.com/s/{FORM_ID}",
      data,
      { headers: { "Accept": "application/json" } }
    );

    if (result.code === 200) {
      // Success
    }
  } catch (error) {
    // Error
  }
}
```

---

## jQuery

```javascript
$("#contact-form").on("submit", function (e) {
  e.preventDefault();

  var data = {};
  $(this).serializeArray().forEach(function (field) {
    data[field.name] = field.value;
  });

  $.ajax({
    url: "https://formcarry.com/s/{FORM_ID}",
    method: "POST",
    data: JSON.stringify(data),
    contentType: "application/json",
    headers: { "Accept": "application/json" },
    success: function (result) {
      if (result.code === 200) {
        // Success
      }
    },
    error: function () {
      // Error
    },
  });
});
```

---

## XMLHttpRequest

For environments without Fetch support.

```javascript
var form = document.getElementById("contact-form");

form.addEventListener("submit", function (e) {
  e.preventDefault();

  var formData = new FormData(form);
  var data = {};
  formData.forEach(function (value, key) {
    data[key] = value;
  });

  var xhr = new XMLHttpRequest();
  xhr.open("POST", "https://formcarry.com/s/{FORM_ID}");
  xhr.setRequestHeader("Content-Type", "application/json");
  xhr.setRequestHeader("Accept", "application/json");

  xhr.onload = function () {
    var result = JSON.parse(xhr.responseText);
    if (result.code === 200) {
      form.reset();
      // Success
    } else {
      // Error
    }
  };

  xhr.onerror = function () {
    // Network error
  };

  xhr.send(JSON.stringify(data));
});
```

---

## File Upload HTML

```html
<!-- Single file -->
<form action="https://formcarry.com/s/{FORM_ID}" method="POST" enctype="multipart/form-data">
  <input type="text" name="name" required>
  <input type="email" name="email" required>
  <input type="file" name="resume">
  <button type="submit">Submit</button>
</form>

<!-- Multiple files (one input) -->
<form action="https://formcarry.com/s/{FORM_ID}" method="POST" enctype="multipart/form-data">
  <input type="file" name="documents" multiple>
  <button type="submit">Upload</button>
</form>

<!-- Multiple files (separate inputs) -->
<form action="https://formcarry.com/s/{FORM_ID}" method="POST" enctype="multipart/form-data">
  <input type="file" name="photo">
  <input type="file" name="resume">
  <input type="file" name="cover_letter">
  <button type="submit">Submit</button>
</form>
```

---

## File Upload AJAX

```javascript
const form = document.getElementById("upload-form");

form.addEventListener("submit", async (e) => {
  e.preventDefault();

  const formData = new FormData(form);

  // IMPORTANT: Do NOT set Content-Type header
  // Browser automatically sets it with multipart boundary
  const response = await fetch("https://formcarry.com/s/{FORM_ID}", {
    method: "POST",
    headers: { "Accept": "application/json" },
    body: formData,
  });

  const result = await response.json();
  // Handle result
});
```

---

## Contact Form

Complete contact form with spam protection, validation, loading state, and success feedback.

```html
<form id="contact" action="https://formcarry.com/s/{FORM_ID}" method="POST">
  <div>
    <label for="name">Name</label>
    <input type="text" id="name" name="name" required>
  </div>

  <div>
    <label for="email">Email</label>
    <input type="email" id="email" name="email" required>
  </div>

  <div>
    <label for="subject">Subject</label>
    <select id="subject" name="subject">
      <option value="general">General Inquiry</option>
      <option value="support">Support</option>
      <option value="feedback">Feedback</option>
      <option value="partnership">Partnership</option>
    </select>
  </div>

  <div>
    <label for="message">Message</label>
    <textarea id="message" name="message" rows="5" required></textarea>
  </div>

  <!-- Spam protection -->
  <input type="hidden" name="_gotcha">

  <!-- Redirect after submit -->
  <input type="hidden" name="_next" value="https://yoursite.com/thanks">

  <button type="submit">Send Message</button>
</form>
```

---

## Newsletter

Minimal email capture.

```html
<form action="https://formcarry.com/s/{FORM_ID}" method="POST" style="display:flex;gap:8px;">
  <input type="email" name="email" placeholder="your@email.com" required>
  <input type="hidden" name="_gotcha">
  <button type="submit">Subscribe</button>
</form>
```

---

## Validation Handling

Handle Formcarry's 422 validation errors in the UI.

```javascript
async function submitWithValidation(form) {
  const data = Object.fromEntries(new FormData(form));

  // Clear previous errors
  form.querySelectorAll(".error").forEach((el) => el.classList.remove("error"));
  form.querySelectorAll(".error-message").forEach((el) => el.remove());

  const res = await fetch("https://formcarry.com/s/{FORM_ID}", {
    method: "POST",
    headers: { "Content-Type": "application/json", "Accept": "application/json" },
    body: JSON.stringify(data),
  });

  const result = await res.json();

  if (result.code === 200) {
    form.reset();
    return true;
  }

  if (result.code === 422 && result.errors) {
    for (const [field, error] of Object.entries(result.errors)) {
      const input = form.querySelector(`[name="${field}"]`);
      if (input) {
        input.classList.add("error");
        const msg = document.createElement("span");
        msg.className = "error-message";
        msg.textContent = error.message;
        input.parentNode.appendChild(msg);
      }
    }
  }

  return false;
}
```

```css
.error {
  border-color: #d1031c;
}

.error-message {
  color: #d1031c;
  font-size: 12px;
  margin-top: 4px;
  display: block;
}
```
