# Sliced.ai — Vibe Coding Build Guide

Sep 27, 2026 · @Anand Baraik

## Overview

Sliced.ai turns a topic, text, URL or YouTube video into a branded, platform-ready carousel, then exports or auto-publishes it. This guide applies the structured vibe coding workflow to that product, from idea to monitored production.

The golden rule: never go IDEA → AI → DEPLOY. Sliced.ai has five hard parts that punish shortcuts — AI output quality, a design engine, pixel-perfect export, OAuth with four social platforms, and background scheduling.

**The workflow**

1. Idea
2. Research
3. Define the user
4. PRD
5. Choose tech stack
6. Architecture
7. Design
8. Project rules
9. Task breakdown
10. Setup
11. Development (one vertical slice at a time)
12. Testing
13. Security review
14. Code review
15. Preview deployment
16. QA testing
17. Production deploy
18. Monitoring
19. Iteration

## Step 1: Define what you want to build

Ship a small, excellent carousel maker first; publishing and the bonus suite come later. Answer the five questions before opening any AI tool.

**1. What problem are you solving?**

Creators and marketers spend hours designing carousels by hand in general-purpose tools, then post manually to each platform in different sizes.

**2. Who is the user?**

- LinkedIn creators and personal brands (primary)
- Instagram educators and coaches
- Small marketing teams and agencies (later, for scheduling)

**3. What is the main outcome?**

Go from an idea to a branded, downloadable carousel in under five minutes.

**4. What is the MVP?**

- AI writing assistant: generate slides from a topic or pasted text
- Editor: add, delete, reorder slides; Text / Text + Image / Image slide types
- Templates with a small gallery (one or two per category)
- Branding: colors, font pairing, creator info block, slide counter
- Smart auto-resize so text always fits
- Platform format: LinkedIn 4:5 and Instagram feed
- Export PNG and PDF, with watermark on the free tier
- No-signup usage; account required to save drafts

**5. What is NOT in the first version?**

- URL and YouTube sources, Import Carousel
- Conversational edit bar
- Payments and Remove Watermark
- Carousel ad formats and Stories
- Entire Auto-Publishing Suite
- Bonus content tools and carousel-to-video

**Release phases**

1. **Phase 1 — MVP carousel maker:** the list above.
2. **Phase 2 — Power editing:** URL and YouTube sources, conversational edit bar, Import Carousel, Surprise Me, background effects, intro types, paid plan with Remove Watermark.
3. **Phase 3 — Publishing:** connect accounts, LinkedIn first, then Instagram and X; scheduling, content calendar, status tracking with retry.
4. **Phase 4 — Video and suite:** carousel-to-video, YouTube Shorts, bonus tools one at a time.

## Step 2: Research before coding

The riskiest research for Sliced.ai is platform API access, not features — verify it before promising auto-publishing to anyone.

**Users**

- Interview 5–10 LinkedIn creators: how they make carousels today, time spent, what they hate
- Which platforms do they post to, and how often?
- Would they pay to remove a watermark, or only for scheduling?

**Competitors**

Study aiCarousels and similar carousel makers, plus general design tools and schedulers. For each, note features, UI, pricing, export quality, onboarding friction, and what users complain about in reviews.

**Technical feasibility (check official docs only)**

- **LinkedIn:** which API product and scopes allow posting a PDF as a native document post, and whether app review is needed
- **Instagram:** Graph API carousel publishing rules, item limits, and the Business/Creator account requirement
- **X:** which paid API tier allows posting, and the per-post image limit (a 10-slide carousel becomes a thread)
- **YouTube:** upload quota, and whether an unverified app's uploads are forced private until an audit passes
- **YouTube as a source:** is there an official, allowed way to get transcripts of videos you don't own?
- **URL source:** robots and paywalls; how to extract clean article text
- **LLM cost:** estimated tokens per carousel × free-tier usage = monthly cost
- **Rendering:** can server-side export match the editor pixel for pixel?

Write findings into DECISIONS.md. Any "no" here reshapes Phase 3 and 4.

## Step 3: AI coding tool and tech stack

Use Cursor or Claude Code on a local repo: Sliced.ai is too large and integration-heavy for browser-only builders. Lovable or Bolt are fine for a throwaway editor UI prototype in week one.

**Recommended stack**

- **AI IDE:** Cursor (feature work) + Claude Code (repo-wide refactors, tests)
- **Frontend + backend:** Next.js (App Router) + TypeScript
- **Styling (app UI):** Tailwind CSS + a headless component library + ShadCn 
- **Editor state:** Zustand with undo/redo history
- **Drag and drop:** dnd-kit for slide reordering
- **Validation:** Zod for AI output, forms and API input
- **Database, auth, storage:** Supabase (Postgres, Auth, Storage, Row Level Security)
- **LLM:** one provider behind an internal `ai` service, structured JSON output
- **Export:** headless Chromium (Playwright) rendering the same slide components to PNG and PDF
- **Background jobs + scheduling:** a managed queue with cron and retries (e.g. Inngest or Trigger.dev)
- **Payments (Phase 2):** Stripe or Razorpay, depending on where customers are
- **Video (Phase 4):** Remotion or FFmpeg on a worker
- **Testing:** Vitest (unit), Playwright (E2E and visual snapshots)
- **Deployment:** Vercel (web) + a separate worker for export and video
- **Monitoring:** Sentry, product analytics, uptime checks

Why a separate worker: headless browser export and video rendering are too slow and heavy for serverless function limits.

## Step 4: Environment and project setup

Version control from minute one: AI will break the editor, and Git is your rollback.

**Install**

- Required: Cursor or VS Code, Git, Node.js LTS, npm or pnpm, GitHub account, Chrome
- Also needed here: Supabase CLI, Docker (local Supabase + the export worker), Vercel CLI
- Optional: GitHub CLI, Bruno or Postman, a Postgres client

**Verify**

```bash
node --version
npm --version
git --version
supabase --version
docker --version
```

**Create the project**

```bash
mkdir sliced-ai
cd sliced-ai
git init
```

Then create the GitHub repo, and create separate developer apps early on LinkedIn, Meta, X and Google — their reviews take time.

## Step 5: PRD.md

The PRD pins down WHAT and WHY; keep the full feature list in it, but mark every item with its phase so the AI never builds Phase 3 during Phase 1.

```markdown
# Product Requirements Document

## Product
Sliced.ai

## Problem
Creators spend hours hand-designing carousels and
re-sizing them for every platform.

## Target Users
LinkedIn creators, Instagram educators, small marketing teams.

## Goal
Idea to branded, downloadable carousel in under 5 minutes.

## Core Features (by phase)
P1: AI generation (topic, text), editor, templates, branding,
    auto-resize, LinkedIn 4:5 + Instagram feed, PNG/PDF export,
    watermark, no-signup use, save draft with account
P2: URL + YouTube sources, conversational edit bar,
    import carousel, Surprise Me, background effects,
    intro types, ad formats, paid plan (remove watermark)
P3: Account connections, scheduling, LinkedIn / Instagram / X
    auto-post, content calendar, status + retry
P4: Carousel-to-video, YouTube Shorts, bonus tools

## MVP (Phase 1)
- Generate 5-10 slides from a topic or pasted text
- Pick a template (with or without sample content)
- Edit text; add, delete, reorder slides
- Set colors, font pairing, creator info, counter
- Switch platform format
- Export PNG (zip) and PDF
- Sign up, save and reopen drafts

## Out of Scope (Phase 1)
- Payments, publishing, video, bonus tools
- URL/YouTube sources, edit bar, import

## Success Criteria
A user can:
1. Generate a carousel without signing up
2. Change template and colors without losing text
3. Reorder and delete slides
4. Export a PDF that matches the editor exactly
5. Sign up and find the draft saved
6. See a watermark on free exports
```

## Step 6: ARCHITECTURE.md

The core decision: a carousel is a JSON document, and ONE React slide renderer draws it in the editor, the preview and the export. Everything else hangs off that.

&#91;embedded content: Sliced.ai architecture · 7 components\]

The browser edits JSON; Next.js calls the LLM and saves to Supabase; exports and scheduled posts go through the queue to a worker, which renders slides and posts to each platform.

**Carousel document (simplified)**

```ts
type Carousel = {
  id: string;
  platform: 'linkedin_4_5' | 'instagram_feed' | 'instagram_story';
  templateId: string;
  theme: { palette: Palette; fontPair: string; alternateColors: boolean; background: string };
  creator: { show: boolean; name: string; title: string; avatarUrl?: string };
  counter: { show: boolean; corners: string };
  slides: Slide[];
};

type Slide =
  | { id: string; kind: 'intro'; variant: 'standard' | 'emoji' | 'headshot' | 'image'; title: string; subtitle?: string; cta?: string }
  | { id: string; kind: 'text'; heading: string; body: string }
  | { id: string; kind: 'text_image'; heading: string; body: string; imageUrl: string }
  | { id: string; kind: 'image'; imageUrl: string };
```

**Main tables**

- `profiles`, `brand_kits`
- `templates`
- `carousels` (JSON document + owner), `exports`
- `social_accounts` (encrypted tokens) — Phase 3
- `scheduled_posts`, `post_targets` (one row per platform, with status) — Phase 3
- `subscriptions`, `usage` — Phase 2

**Folder structure**

```
src/
├── app/                 # routes, layouts, server actions
├── components/          # app UI (buttons, dialogs)
├── renderer/            # slide components shared by editor + export
├── features/
│   ├── generate/
│   ├── editor/
│   ├── templates/
│   ├── export/
│   └── publish/         # Phase 3
├── services/            # ai, carousel, export, social
├── lib/                 # supabase clients, zod schemas
├── types/
└── utils/
worker/                  # export, publish, video jobs
```

**Architecture rules**

- The carousel JSON schema (Zod) is the single source of truth.
- The renderer is pure: props in, slides out; no fetching, no store access.
- UI components never call Supabase or the LLM directly; services do.
- LLM calls are server-side only, and every response is schema-validated.
- Each platform integration is an adapter behind one `SocialPublisher` interface.
- Anything slower than a few seconds runs as a queued job.

## Step 7: DESIGN.md

Sliced.ai needs two design systems in one file: the app's own UI, and the rules every carousel template must follow. Mixing them is the fastest way to get inconsistent output.

```markdown
# Design System

## Part A: App UI (editor, dashboard, calendar)
Style: modern, minimal, focused on the canvas
Font: Inter
Primary: #6366F1  Background: #F8FAFC
Text: #0F172A     Muted: #64748B
Radius: 12px cards, 8px inputs
Buttons: Primary, Secondary, Ghost, Destructive
Layout: left panel (slides), center canvas, right panel (design)
UX: loading, empty and error states everywhere;
    autosave indicator; undo/redo; keyboard shortcuts

## Part B: Carousel templates
Every template defines:
- Layouts for intro, text, text + image, image slides
- Safe zones per platform (no text near edges or counter)
- Min / max font sizes for auto-resize
- Slots: logo, creator info, counter, corners, CTA
- Default "Swipe ->" CTA on slide 1

Palettes by mood: Dark, Light, Vibrant, Pastel, Muted
  Each palette = background, text, accent (+ alt set
  for "Alternate Colors Between Slides")
Font pairings: 6-8 curated heading + body pairs
Contrast: text vs background meets WCAG AA

## Formats
LinkedIn 4:5, Instagram feed 4:5 and 1:1,
Instagram Stories 9:16, ad formats per platform spec
```

Keep template definitions as data (JSON), not hard-coded components, so adding a template never touches the renderer.

## Step 8: RULES.md

RULES.md is the AI's rulebook; add Sliced.ai-specific rules on top of the general ones.

```markdown
# Development Rules

## General
- TypeScript strict mode. No `any`.
- Reuse existing components and services.
- Keep functions small. Do not modify unrelated files.
- Only build features marked for the current phase in PRD.md.

## Before Coding
- Read PRD, ARCHITECTURE, DESIGN, MEMORY.
- Inspect existing code first; plan large changes before writing.

## Carousel + Renderer
- Never change the carousel schema without updating
  the Zod schema, a migration, and DECISIONS.md.
- Renderer components stay pure (no fetch, no store).
- Every visual change must pass the export snapshot tests.

## AI
- Prompts live in services/ai/prompts, versioned.
- Always request structured JSON and validate with Zod.
- Handle LLM failure: retry once, then show a clear error.

## UI
- Follow DESIGN.md. Responsive editor down to 1024px,
  read-only preview on mobile.
- Loading, empty and error states for every async action.

## Security
- No secrets in client code. Tokens encrypted at rest.
- Validate all input; check ownership server-side.

## Testing
- Tests for every service and critical flow; run before finishing.

## Git
- Small commits, conventional messages (feat:, fix:).
```

For Cursor, split these into scoped rule files:

```
.cursor/rules/
├── general.mdc
├── renderer.mdc     # applies to src/renderer/**
├── ai.mdc           # applies to src/services/ai/**
├── publish.mdc      # applies to src/features/publish/**, worker/**
└── testing.mdc
```

For Claude Code, put the same rules in a `CLAUDE.md` at the repo root.

## Step 9: TASKS.md

Build the renderer and export before AI generation: if export can't match the editor, nothing else matters. Each task ends with implement → test → review → commit.

**Phase 0: Setup**

- [ ] Init Next.js + TypeScript + Tailwind
- [ ] Local Supabase, env files, Git, CI (lint, typecheck, test)
- [ ] Carousel Zod schema + types

**Phase 1A: Renderer + export (the foundation)**

- [ ] Slide renderer: intro, text, text + image, image
- [ ] One template as JSON, rendered at LinkedIn 4:5
- [ ] Smart auto-resize (fit text between min and max size)
- [ ] Export worker: PNG per slide + PDF via Playwright
- [ ] Visual snapshot test: editor render = export render

**Phase 1B: Editor**

- [ ] Editor layout: slide list, canvas, design panel
- [ ] Edit text inline; add, delete, reorder (dnd-kit)
- [ ] Undo / redo
- [ ] Colors: mood palettes, custom pickers, alternate colors
- [ ] Font pairings
- [ ] Creator info, counter and corners, Swipe CTA
- [ ] Platform format switcher

**Phase 1C: AI generation**

- [ ] `ai` service: topic or text → validated Carousel JSON
- [ ] Generate screen with loading and error states
- [ ] Template gallery + Include Sample Content toggle

**Phase 1D: Accounts**

- [ ] Anonymous use with local draft, watermark on export
- [ ] Sign up / login; migrate the local draft on signup
- [ ] Save, list, open, delete drafts (RLS on carousels)
- [ ] Rate-limit anonymous generation

**Phase 2: Power editing** — URL source, YouTube source, conversational edit bar (slide / carousel scope), Import Carousel, Surprise Me, background effects, intro variants, ad formats, payments + Remove Watermark.

**Phase 3: Publishing** — social account connections, LinkedIn document post, scheduler + queue, status tracking + retry, content calendar, per-platform preview, then Instagram, then X.

**Phase 4: Video + suite** — carousel-to-video, YouTube Shorts, then bonus tools one per release.

## Step 10: Supporting docs

These five files keep the AI from re-deciding the architecture, forgetting progress, or shipping insecure publishing code.

**DECISIONS.md** — permanent decisions

```markdown
## ADR-001: Carousel is a JSON document
Reason: one schema drives editor, AI output, export and publishing.

## ADR-002: One shared renderer for editor and export
Reason: guarantees exported slides match what the user saw.

## ADR-003: Export and publishing run on a queued worker
Reason: headless browser and platform APIs exceed serverless limits
and need retries.

## ADR-004: LinkedIn is the first publishing platform
Reason: primary audience; carousels post as PDF documents.
```

**MEMORY.md** — current state

```markdown
## Current Status
Phase 1A: renderer done, export worker in progress.
## Current Task
TASK-008: PDF export
## Known Issues
- Emoji rendering differs in export (font not loaded)
## Next Step
Visual snapshot test for export
```

**TEST\_PLAN.md**

```markdown
## Generation
- Topic produces 5-10 valid slides
- Malformed LLM output shows an error, never a broken editor
## Editor
- Reorder, delete, undo/redo keep data intact
- Switching template or format keeps all text
- Long text auto-resizes and never overflows
## Export
- PNG and PDF match editor snapshots at each format
- Free export has watermark; paid does not
## Accounts
- Anonymous draft survives signup
- User cannot open another user's carousel
## Publishing (Phase 3)
- Scheduled post fires at the right time in the user's timezone
- Expired token marks post failed and asks to reconnect
- Retry does not create a duplicate post
## Responsive
- Editor at 1024px and 1440px; preview at 375px
```

**SECURITY.md**

```markdown
## Authorization
RLS on every user table; ownership checked server-side.
## Secrets
LLM and platform secrets server-only.
Social access and refresh tokens encrypted at rest.
## OAuth
State parameter + PKCE; request minimum scopes.
## Abuse and cost
Rate-limit generation per IP and per user; cap input length.
## URL source
Block private IPs and internal hosts (SSRF).
## Uploads (import, images)
Validate type, size, filename; scan PDFs; never trust client MIME.
## AI
Treat fetched page and transcript text as data, not instructions.
```

**.env.example**

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
LLM_API_KEY=
TOKEN_ENCRYPTION_KEY=
QUEUE_SIGNING_KEY=
LINKEDIN_CLIENT_ID=
LINKEDIN_CLIENT_SECRET=
META_APP_ID=
META_APP_SECRET=
X_CLIENT_ID=
X_CLIENT_SECRET=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
PAYMENTS_SECRET_KEY=
SENTRY_DSN=
```

Real values live in `.env.local` and the hosting dashboard, never in Git.

## Step 11: Building with AI

One task, one vertical slice, one structured prompt at a time. Start every session by loading context, not by asking for code.

**Kick-off prompt**

```
Read PRD.md, ARCHITECTURE.md, DESIGN.md, RULES.md,
TASKS.md and MEMORY.md. Do not modify anything yet.

1. Summarise the product and the current phase.
2. Summarise the architecture and the carousel schema.
3. List missing or conflicting information.
4. Explain the implementation plan for the next open task.

Do not write code yet.
```

**Vertical slices for Sliced.ai**

Don't build "the whole editor", then "the whole backend". Build complete user flows:

1. Hard-coded carousel JSON → renderer → export PDF
2. Edit a slide's text → re-render → export matches
3. Enter topic → AI JSON → editor → export
4. Sign up → save draft → reload → same carousel
5. (Phase 3) Connect LinkedIn → schedule → job runs → status = published

**Structured prompt (example: AI generation)**

```
CONTEXT
Sliced.ai carousel maker. Read PRD, ARCHITECTURE, RULES.

TASK
Implement generateCarousel(topic | text) in services/ai.

FILES
src/services/ai/, src/lib/schemas/carousel.ts

CONSTRAINTS
- Server-side only. Use the existing Carousel Zod schema.
- Do not touch the renderer or editor.
- Max input 5,000 characters.

ACCEPTANCE CRITERIA
- Returns 5-10 slides: 1 intro, body slides, 1 CTA slide.
- Output always passes schema validation.
- Invalid LLM output retries once, then returns a typed error.

TESTING
Unit tests with a mocked LLM (valid, invalid, timeout).
Run lint, typecheck, tests.

Report: files changed, what was done, tests run, open issues.
```

**Keep tasks small**

- Bad: "Build the carousel editor."
- Better: "Build slide reordering."
- Best: "Add drag-to-reorder in the slide list with dnd-kit. Update store order only. Keep undo working. Do not change the renderer."

**Debugging prompt**

```
ERROR: [paste]
EXPECTED: Exported PDF matches the editor.
ACTUAL: Headings wrap onto 3 lines in the PDF.
STEPS: Generate from topic → Bold template → Export PDF.
CONSTRAINT: Do not change the carousel schema.

Do not modify code yet. Find the root cause:
1. What is failing and why?
2. Which file is responsible?
3. What is the smallest fix?
4. How will we test it?
```

Then: "Implement the smallest fix. Do not refactor unrelated code. Run the relevant tests."

## Step 12: Testing and review

For Sliced.ai, visual snapshot tests of the renderer matter as much as unit tests: a 2px font change silently ruins every export.

**After every task**

```bash
npm run lint
npm run typecheck
npm test
npm run build
```

**Test layers**

- **Unit (Vitest):** schema validation, auto-resize maths, palette rotation, AI service with a mocked LLM
- **Integration:** services against local Supabase; RLS blocks cross-user access
- **Visual (Playwright):** each template × each format, editor screenshot vs export output
- **E2E (Playwright):** open site → generate → edit → reorder → export → sign up → draft saved → reload → delete
- **Publishing (Phase 3):** platform adapters tested against mocked APIs; one manual test account per platform

**Code review prompt**

```
Review the implementation against PRD, ARCHITECTURE,
DESIGN, RULES, TEST_PLAN and SECURITY.
Check: correctness, schema changes, renderer purity,
security, error handling, accessibility, responsiveness,
performance (editor re-renders, bundle size), duplication.
Do not modify anything yet. Report all issues first.
```

Then fix issues one at a time, re-running tests after each.

**Security review before each phase ships**

- Secrets not in Git or client bundle
- RLS on every new table
- Rate limits on generation and URL fetching
- OAuth tokens encrypted; minimum scopes
- Uploads validated

## Step 13: Deployment and production

Every change goes local → preview → QA → production, with separate Supabase projects and separate social developer apps for staging and production.

**Git flow**

```
main
├── feature/renderer
├── feature/export-worker
├── feature/ai-generate
└── fix/auto-resize-overflow
```

```bash
git add .
git commit -m "feat: add PDF export"
git push
```

**Pre-deploy checklist**

- [ ] Generate, edit, reorder, delete, export all work
- [ ] Loading, empty, error states present
- [ ] Editor at 1024px and 1440px; preview on mobile
- [ ] Keyboard navigation and accessible forms
- [ ] No secrets in Git; RLS verified; rate limits on
- [ ] Lint, typecheck, tests, build all pass

**Environments**

- Development: local Supabase, test LLM key, sandbox social apps
- Preview: staging Supabase, per-branch Vercel URL, staging worker
- Production: production Supabase, production worker, approved social apps

**Deploy**

1. Connect the GitHub repo to Vercel; set env vars per environment
2. Deploy the worker (container host) with its own env vars
3. Push a branch → test the preview URL
4. Merge to main → production (or `vercel --prod`)

**Production QA (on the live URL, not localhost)**

- Generate and export on every template and format
- Refresh the editor mid-edit; open a draft by direct URL
- Logged-out access to /dashboard is blocked
- Huge input, empty input, emoji, non-English text
- Slow network: loading states and autosave
- Phase 3: a real scheduled post to a test account per platform

**Monitoring**

- Error tracking (Sentry) on app and worker
- Analytics funnel: visit → generate → export → sign up
- Cost tracking: LLM tokens and export minutes per user
- Queue health: failed jobs, retries, publish failure rate
- Uptime checks and daily database backups

Then: collect feedback → fix → test → deploy again.

## Project structure and the loop

Start with PRD, RULES, TASKS, README and .env.example; add the rest as each phase needs it.

```
sliced-ai/
├── docs/
│   ├── PRD.md
│   ├── ARCHITECTURE.md
│   ├── DESIGN.md
│   ├── TEST_PLAN.md
│   ├── SECURITY.md
│   ├── DECISIONS.md
│   └── MEMORY.md
├── .cursor/rules/
├── src/
│   ├── app/
│   ├── components/
│   ├── renderer/
│   ├── features/
│   ├── services/
│   ├── lib/
│   ├── types/
│   └── utils/
├── worker/
├── templates/          # template JSON definitions
├── supabase/migrations/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── visual/
│   └── e2e/
├── public/
├── CLAUDE.md
├── RULES.md
├── TASKS.md
├── README.md
├── .env.example
└── package.json
```

**The loop, for every feature**

1. Read the docs
2. Understand the task
3. Plan
4. Implement
5. Test
6. Review
7. Fix
8. Commit
9. Update TASKS, MEMORY and DECISIONS

Then move to the next task. Documentation that describes an app that no longer exists misleads the AI.
