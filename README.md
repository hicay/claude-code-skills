# Claude Code Skills

A collection of practical skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — the CLI tool by Anthropic.

Skills are modular instruction packages that teach Claude how to handle specialized tasks. They load automatically when relevant, giving Claude domain expertise it doesn't have by default.

## Available Skills

### [ui-ux-design](./ui-ux-design)

Design and redesign interfaces using Dieter Rams' 10 principles of good design. Includes a complete design system (8pt grid, 3-tier typography, achromatic palette), component library with SwiftUI/React/CSS/Tailwind examples, and a thorough review checklist.

**Triggers on:** "redesign this screen," "improve the UI," "Dieter Rams," "design system," "visual hierarchy," "make this more readable," "fix the spacing"

**What's inside:**

| File | Lines | Content |
|---|---|---|
| `SKILL.md` | 258 | 10 principles, layout patterns, component anatomy, redesign process |
| `references/design-tokens.md` | 308 | Grid, color, typography, radius, timing + SwiftUI/CSS/Tailwind/React Native code |
| `references/component-library.md` | 820 | 15 components with full code (SwiftUI + React/CSS) |
| `references/review-checklist.md` | 129 | 60+ checkpoints for structure, color, typography, interaction, consistency |

---

### [formcarry](./formcarry)

Build and deploy working forms without writing any backend code. Uses [Formcarry](https://formcarry.com) as the form endpoint. Covers everything from a simple HTML contact form to AJAX submissions with React/Vue/Next.js, file uploads, spam protection, auto-responders, and the REST API.

**Triggers on:** "formcarry," "form backend," "contact form," "form without server," "collect form submissions," "lead capture form," "newsletter signup"

**What's inside:**

| File | Lines | Content |
|---|---|---|
| `SKILL.md` | 357 | Quick start, submission methods, spam protection, file uploads, validation, email templates, webhooks, Stripe, pricing |
| `references/code-examples.md` | 672 | 16 complete examples: HTML, Fetch, React, React Hook, Vue 3, Next.js, Astro, Axios, jQuery, XHR, file uploads |
| `references/api-reference.md` | 313 | Forms API, Submissions API, filtering, pagination, authentication, export patterns |

---

## Installation

### Claude Code (CLI)

Copy the skill folder into your project's `.claude/skills/` directory:

```bash
# Clone the repo
git clone https://github.com/hicay/claude-code-skills.git

# Copy the skill you want into your project
cp -r claude-code-skills/ui-ux-design /path/to/your-project/.claude/skills/
cp -r claude-code-skills/formcarry /path/to/your-project/.claude/skills/
```

Or install directly with a one-liner:

```bash
# UI/UX Design skill
git clone --depth 1 https://github.com/hicay/claude-code-skills.git /tmp/claude-skills && \
  mkdir -p .claude/skills && \
  cp -r /tmp/claude-skills/ui-ux-design .claude/skills/ && \
  rm -rf /tmp/claude-skills

# Formcarry skill
git clone --depth 1 https://github.com/hicay/claude-code-skills.git /tmp/claude-skills && \
  mkdir -p .claude/skills && \
  cp -r /tmp/claude-skills/formcarry .claude/skills/ && \
  rm -rf /tmp/claude-skills
```

After installation, the skill loads automatically when you use Claude Code in that project. No configuration needed.

### Claude.ai (Web)

1. Zip the skill folder: `zip -r ui-ux-design.zip ui-ux-design/`
2. Go to **Settings > Capabilities**
3. Upload the zip file
4. Enable the skill

### Verify Installation

Run Claude Code in your project and ask something that triggers the skill. You should see the skill name appear in the context.

---

## Example Prompts

### UI/UX Design

**Redesign a screen:**
```
Redesign my settings page. It's not very readable — the sections blend together and there's no visual hierarchy.
```

**Review existing UI:**
```
Review the ProfileView.swift file. Check if it follows our design system — consistent section headers, proper spacing, correct typography scale.
```

**Build a component:**
```
Create a SearchField component that follows the design system. It should have an icon, plain text field style, and bordered container.
```

**Design system audit:**
```
Audit all views in the Views/ folder. Find any inconsistencies in section header styles, spacing values, or color usage.
```

**Think like Rams:**
```
Think like Dieter Rams — how would you restructure the home screen? Don't remove any features, but rethink the information architecture.
```

**Two-tone layout:**
```
Make the sidebar grey and the content area white, like a Braun product. The control panel should recede, the content should be the focus.
```

---

### Formcarry

**Quick contact form:**
```
Add a contact form to my landing page. Use Formcarry so I don't need a backend. Include spam protection.
```

**React form with validation:**
```
Create a React contact form component that submits to Formcarry. Handle loading state, success message, and validation errors. Use the useFormcarry hook pattern.
```

**File upload form:**
```
I need a job application form with file upload for resumes. Use Formcarry. The form should accept name, email, position, and a PDF upload.
```

**Newsletter signup:**
```
Add a minimal newsletter signup form to the footer. Just an email field and submit button. Use Formcarry, no page reload.
```

**Astro + Formcarry:**
```
Add a contact form to my Astro site. I want it to work without JavaScript — pure HTML form submission with a redirect to /thanks.
```

**Form with auto-response:**
```
Set up a Formcarry form via the API that sends an auto-response email to the submitter confirming we received their message.
```

**Export submissions:**
```
Write a script that exports all Formcarry submissions from the last 30 days using the Submissions API. Filter out spam.
```

---

## How Skills Work

```
You ask a question
  → Claude checks skill descriptions (~100 words each, always in context)
  → If a skill matches, SKILL.md body loads (~300 lines)
  → If Claude needs details, it reads from references/ (on demand)
```

This progressive disclosure keeps context efficient — Claude only loads what it needs.

## Creating Your Own Skills

See Anthropic's [skill-creator guide](https://github.com/anthropics/skills/tree/main/skills/skill-creator) for the full specification.

Quick structure:

```
my-skill/
├── SKILL.md              # Required: frontmatter + instructions
├── LICENSE.txt            # License
└── references/            # Optional: detailed docs loaded on demand
    ├── guide.md
    └── examples.md
```

## License

MIT — see each skill's `LICENSE.txt` for details.
