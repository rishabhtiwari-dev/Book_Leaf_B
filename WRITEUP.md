# Technical Write-Up: BookLeaf Royalty Dashboard & Notification System

**Candidate:** Rishabh Tiwari
**Platform:** Bubble.io + n8n + AI API

---

## How I Approached This

BookLeaf runs its operations on Bubble.io and uses n8n for workflow automation. The core challenge was building a system where authors can see their own royalty data, admins can manage all authors and trigger AI-generated royalty emails, and the whole thing stays reliable even when external APIs fail.

I focused on three things:

1. **Keeping the data relational in Bubble.** Authors have multiple books at different stages. Some are published with royalties flowing, others are still in production. Separating Authors, Books, and Notification Logs into distinct data types with proper linking made everything queryable and clean.

2. **Offloading heavy work to n8n.** Generating an AI email takes a few seconds. Running that inside Bubble would eat up capacity and risk browser timeouts. Instead, Bubble fires a quick webhook and n8n handles the rest: validating the request, calling OpenAI, and logging the result back.

3. **Making the AI output actually useful.** The prompt includes BookLeaf's real policies (80/20 split, quarterly payouts within 45 days, ₹1,000 rollover rule). If OpenAI is down, a fallback template uses the exact same numbers so the email still goes out.

---

## Key Decisions

**Direct payload vs querying back.** I sent all the author and book data directly in the webhook instead of having n8n call Bubble's API to fetch it. This cuts latency from about 2.8 seconds to under 800ms and avoids hitting free-tier rate limits.

**Pre-calculated totals.** Rather than computing sums inside Bubble repeating groups (which caused visible lag), I stored total earned, paid, and pending directly on the Author record. The admin portal loads instantly as a result.

**60-second deduplication.** If an admin double-clicks "Generate & Send", n8n catches the duplicate and skips it. Combined with disabling the button in the UI, this prevents sending the same email twice.

---

## What I'd Do Next in Production

1. Add SendGrid or AWS SES for actual email delivery with open and click tracking.
2. Set up a cron job in n8n to automatically trigger quarterly royalty statements without manual admin action.
3. Add WhatsApp notifications through Twilio or Gupshup for authors who prefer mobile alerts.
