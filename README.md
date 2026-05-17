# The Reinvention Blueprint

> Info product on career and life reinvention. Cloudflare-native, Resend-sequenced, Worker-driven lead capture.

🌐 **Live:** [the-reinvention-blueprint.com](https://the-reinvention-blueprint.com) *(beta)*
🏗️ **Stack:** Cloudflare Pages + Workers + D1 + Resend Audiences
🔒 **Source:** private

---

## What this is

A digital product helping people in mid-career transitions reframe and execute a deliberate reinvention. Full funnel: landing page, lead magnet, multi-touch email sequence, paid offer, post-purchase fulfilment.

## The funnel

```
Organic + paid traffic
       │
       ▼
Landing page  →  Lead magnet (free PDF + video)
       │
       ▼
Resend Audience: "Reinvention Blueprint — Lead Magnet"
       │
       ▼
7-day sequence  →  Mid-funnel offer  →  Stripe
       │
       ▼
Post-purchase: full curriculum delivery + community access
       │
       ▼
Events posted to ops.tbot.trade  (lead → opened → clicked → purchased)
```

## Architecture

- **Landing + sales pages:** Cloudflare Pages, deployed via `wrangler pages deploy`
- **Lead capture:** Cloudflare Worker that validates form submission, dedupes against D1, POSTs to Resend Audience API, returns thank-you redirect
- **Email sequence:** Resend with audience-segmented broadcasts; sequence worker handles drip timing + conditional next-step logic
- **Analytics ingest:** every lead/open/click event POSTs to `events.tbot.trade/ingest` for unified rollups

## Key technical decisions

### 1. Resend Audiences instead of a CRM

Resend's Audience API gives me the email-list primitives I need (add contact, segment, broadcast, transactional send) without the overhead of a Mailchimp / ActiveCampaign / ConvertKit subscription. Direct API. No drag-and-drop builder I'd never use. Costs scale with sends, not list size.

### 2. D1 for deduplication, not as the CRM

The contact-of-record lives in Resend. D1 just holds the dedup index — `(email_hash, audience_id)` unique constraint. If a user submits the form twice, the worker silently no-ops on D1, then no-ops on Resend. Idempotent without complicated logic.

### 3. Sequence-worker as a separate concern

A second worker (`sequence-worker`) handles the multi-day email sequence — checks D1 for who's due for next-step, calls Resend `/emails` send, marks step complete. Runs on Cron Trigger every hour. Decoupled from the lead-capture worker so each can evolve independently.

### 4. Same ingest contract as every other product

The lead → opened → clicked → purchased events POST to the same `events.tbot.trade/ingest` endpoint as Canadian PR Mastery, T BOT, and the others. Unified dashboard. Single source of truth for "what's converting this week" across the whole portfolio.

## What I'd build next

1. **A/B testing harness** — Cloudflare Worker that splits visitors across landing-page variants by deterministic hash; results piped to the unified ops dashboard
2. **Re-engagement sequence** — for leads who opened but never bought after 30 days
3. **Affiliate tracking** — same pattern as the other products in the portfolio

## Why this is interesting

It's an end-to-end revenue product running on **two Cloudflare Workers + one D1 table + Resend**. No CMS, no CRM, no analytics SaaS. The entire stack fits in a few hundred lines of TypeScript and an hour to redeploy if I ever need to. Right-sized for a solo operator running a portfolio.

---

*Source code is private. Architecture and decisions above are the showcase. To discuss, [reach out](https://linkedin.com/in/kenmwara).*
