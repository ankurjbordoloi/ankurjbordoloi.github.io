# Wandr — AI Travel Planner

## What it is
An AI-powered travel planning website with two paths: a free AI itinerary generator and a paid human consultation service (first session free). Built from scratch by Ankur Bordoloi with no prior coding knowledge.

## Live URLs
- Website: https://wandrplan.com (also https://ankurjbordoloi.github.io)
- Cloudflare Worker: https://wandr-backend.ankurjbordoloi.workers.dev
- GitHub repo: ankurjbordoloi/ankurjbordoloi.github.io

---

## Tech Stack
- **Hosting:** GitHub Pages
- **Custom domain:** wandrplan.com (GoDaddy, DNS managed via Cloudflare)
- **Backend:** Cloudflare Worker (wandr-backend)
- **AI:** Anthropic Claude Haiku 4.5 (claude-haiku-4-5-20251001), max_tokens: 3000
- **Email:** Resend (domain: wandrplan.com, from: hello@wandrplan.com) — free plan, transactional only
- **Form data:** Google Apps Script → Google Sheets ("Wandr Leads")
- **Analytics:** Cloudflare Web Analytics (EU excluded)

---

## Pages

### index.html — Landing page
1. Nav: Compass logo (wandr_logo_new_notext.png) + "Wandr" wordmark + "Curated travel" tagline. Sticky with blur backdrop.
2. Hero: Full-bleed travel photo with dark overlay. "Your trip. Planned around you." No buttons.
3. Two-path cards: AI planner (free) + Personal consultation (first session free)
4. Photo strip: Japan, Dubai, Spain
5. Features (3 columns) — white cards with gold top border
6. How it works (4 steps)
7. CTA — full-bleed photo
8. Footer — dark background with newsletter strip + Instagram link
9. Favicon: wandr_logo_new_notext.png

### planner.html — Questionnaire + itinerary
9-question flow (8 if destination = No):
1. Who are you travelling with? (single select)
2. Where are you travelling from? (text)
3. Do you have a destination in mind? (yes/no)
4. Where would you like to go? (text — only if yes)
5. How long is your trip? (single select)
6. Which month are you travelling? (dropdown — next 6 months, "More than 6 months away", "Not sure yet")
7. How do you like to travel? (single select)
8. What kind of experiences excite you? (multi select)
9. What is your travel budget like? (single select)

Nav: Identical to index.html and why.html — compass logo + wordmark + nav links.
Progress bar: Thin 2px gold bar below the nav (separate from nav), disappears on Generate, reappears on restart.

On Generate:
- Answers logged to Sheet 2
- Progress bar hides
- If user typed destination then switched to "No", destination_name is cleared
- AI itinerary generated via Worker, markdown stripped
- Itinerary shown as paywall-style blur overlay — email input embedded inside glimpse card
- Send it button disabled until itinerary loads
- If AI generation fails → friendly error message shown ("our AI stepped out for chai") instead of generic Day 1/Day 2 placeholder

On Send it:
- Full itinerary emailed via Worker
- Second row written to Sheet 2 with email + recommended destination
- Email title for undecided: "Your trip to a destination we recommend for you"
- Newsletter popup shown 1.2s after success (email pre-filled) — only shown once per session

Consultation section:
- Divider: "Not sure yet? Talk to a travel expert!"
- Badge changed from "Premium" to "First session free"
- On consultation submit: Sheet + Worker fire in parallel (Promise.all) with 10s timeout; success shown regardless to avoid user-facing hangs
- Newsletter popup shown 1.2s after consultation submit — only if not already shown this session

### why.html — Consultant page
- Full-bleed hero photo
- Consultant bio with Ankur's real photo (ankur.jpg), stat cards (25 countries, 5 continents)
- Instagram link (@ankurjbordoloi), destination tags by region
- What a session covers (6 cards), who it's for (2 cards)
- Comparison table, CTA photo section
- Consultation booking form → Sheet 1
- On submit: Sheet + Worker fire in parallel (Promise.all) with 10s timeout; success shown regardless
- Newsletter popup shown 1.2s after submit (email pre-filled)
- Favicon: wandr_logo_new_notext.png

---

## Design
- **Theme:** Bright and airy — #faf8f4 bg, #f3ede4 sections, white cards
- **Accent:** Gold (#c8992a)
- **Text:** #1a1612, #6b5f52, #a09485
- **Cards:** White, 2px gold top border, subtle shadow
- **Fonts:** Cormorant Garamond (serif) + Jost (sans, weight 300)
- **Logo:** wandr_logo_new_notext.png — compass star mark with calligraphic W (from Adobe Firefly). Used in nav on all 3 pages and as favicon.
- **Nav:** Identical across all 3 pages — sticky, blur backdrop, compass logo + WANDR wordmark (21px Cormorant Garamond, letter-spacing 7px) + "Curated travel" tagline (gold, 8px Jost) + Home / Why Wandr links
- **Animations:** Scroll fade-in, hero zoom on load
- **Photos:** Unsplash (safe for commercial use, no attribution needed)

---

## Cloudflare Worker — Endpoints

- `POST /itinerary` — Anthropic API → returns itinerary + recommended_dest
- `POST /email` — sends itinerary email via Resend
- `POST /consultation` — sends consultation confirmation email + notification to ankur@wandrplan.com via Resend
- `POST /subscribe` — logs newsletter subscriber to Sheet 3 (Email, Source, Subscribed At)

Security: CORS (wandrplan.com only), origin validation, 20 req/hour/IP rate limit

Secrets: `ANTHROPIC_API_KEY`, `RESEND_API_KEY`

---

## AI Prompt

Inputs: traveller, origin, destination, duration, travel month, booking lead time, pace, vibes, budget

Behaviours:
- Nonsense destination → `INVALID_DESTINATION`
- Misspelled → infers and proceeds
- No destination → recommends one, starts with "Based on your responses, your recommended destination is X."
- Unknown origin → ignored

Formatting:
- 20-25 word warm summary first
- 1-5 days: morning/afternoon/evening with timings
- 6 days–2 weeks: flowing paragraph per day
- 2 weeks–1 month: themed clusters (3-5 days)
- 1 month+: weekly groups
- Max 800 words, no markdown, kid/senior friendly where applicable

Budget: per person, local currency, simple list (food, transport, accommodation, activities + flight estimate with Skyscanner note)

Booking lead time:
- <3 weeks: peak fares
- 4-8 weeks: above average
- 2-5 months: near average
- 5+ months / "More than 6 months away": lowest

---

## Emails — Unified Light Theme

Both emails: #faf8f4 bg, compass logo image + WANDR wordmark + gold tagline (shared LOGO_BLOCK constant in Worker), #6b5f52 body text, #c8992a CTA button.

Logo in emails: hosted at https://raw.githubusercontent.com/ankurjbordoloi/ankurjbordoloi.github.io/main/wandr_logo_new_notext.png

### Itinerary email
- Subject: "Your [DEST] itinerary is ready" / "Your personalised itinerary is ready"
- Title: "Your trip to [DEST]" / "Your trip to a destination we recommend for you"
- CTA → why.html#consultant

### Consultation confirmation email
- Subject: "Your consultation request is confirmed — Wandr"
- Title: "We've received your request, [Name]." (auto title-cased)
- What happens next + prep checklist + Wandr Team sign-off
- CTA → why.html

### Ankur notification email (internal)
- Fires instantly when consultation form is submitted (from planner.html or why.html)
- Sent to: ankur@wandrplan.com
- Subject: "New consultation request — [Name]"
- Contains: name, email (clickable), WhatsApp, preferred time, destination, note

---

## Google Sheets — "Wandr Leads"

- Sheet 1 (Consultation Leads): Consultation leads from planner.html and why.html
- Sheet 2 (Planner Responses): 2 rows per user — Generate + Send it
- Sheet 3 (Newsletter Subscribers): Email, Source, Subscribed At

Form field values:
- `'Planner Responses'` → Sheet 2
- `'Planner - Consultation'` → Sheet 1
- `'Why Page - Consultation'` → Sheet 1
- `'Newsletter'` → Sheet 3

Apps Script: secret token `wandr2026secure`, honeypot, 20/hour rate limit

---

## Newsletter

- Footer subscribe strip on all 3 pages — email field → "You're in. We'll be in touch."
- Newsletter popup on planner.html: fires after "Send it" and after consultation submit (email pre-filled, only once per session via `_nlPopupShown` flag)
- Newsletter popup on why.html: fires after consultation submit (email pre-filled)
- Popup label: "Wandr Newsletter" — copy makes newsletter explicit
- Dismiss behaviour: just closes, can show again on next page visit (no localStorage)
- Subscriber source tracked: `footer`, `popup-planner`, `popup-why`, `popup-index`
- Sending tool: to be set up when first edition is ready (Mailchimp or Brevo free tier)
- Existing planner users (Sheet 2 emails) to be added to list at newsletter launch with a one-time welcome note

---

## Footer — All Pages
- Dark background (#1a1612)
- Left: Wandr wordmark + "Curated travel"
- Right: © year + "Built with intention." + Instagram link (@ankurjbordoloi) below
- Instagram link: gold SVG icon + "Follow my travels", right-aligned below copyright
- Newsletter strip below footer content (email input + Subscribe button)

---

## Email Setup (Gmail)
- hello@wandrplan.com → forwards to ankurjbordoloi@gmail.com via Cloudflare Email Routing
- ankur@wandrplan.com → forwards to ankurjbordoloi@gmail.com via Cloudflare Email Routing
- Both addresses set up as "Send mail as" in Gmail via Gmail SMTP + App Password
- Gmail filter: emails to hello@wandrplan.com → labelled "Wandr"

---

## Security
1. Secret token on all Sheet submissions
2. Honeypot (`hpot`) on all forms
3. localStorage rate limiting (separate keys):
   - `wandr_last_email` — itinerary email
   - `wandr_last_booking` — planner consultation
   - `wandr_last_consult` — why.html consultation
4. Apps Script: 20/hour
5. Worker: 20/hour per IP
6. CORS: wandrplan.com only
7. Invalid destination → friendly error + restart

---

## Conversion Tracking
- Generate → Email: Sheet 2 Row 2 ÷ Row 1
- Consultation source: Form field in Sheet 1
- Newsletter source: Source field in Sheet 3

---

## DNS (Cloudflare)
- 4x A records → GitHub Pages (185.199.108-111.153)
- CNAME www → ankurjbordoloi.github.io
- MX + SPF + DKIM + DMARC → Resend
- Cloudflare Email Routing enabled for hello@ and ankur@

---

## Consultant
- Ankur Bordoloi, 25 countries: India, Nepal, Thailand, Malaysia, Singapore, China, Japan, Turkiye, UAE, Saudi Arabia, Oman, Bahrain, Egypt, Morocco, Spain, Portugal, France, Monaco, Czech Republic, Slovakia, Hungary, Austria, Azerbaijan, Armenia, USA
- Solo travellers (primary), couples, open to all
- First session free (15-30 min, WhatsApp/Phone/Meet)
- Packages: Essential ₹2-3k / Standard ₹4-6k / Full planning ₹8-12k

---

## Operational Documents
- `wandr_discovery_call.docx` — call sheet, packages, red flags, closing script
- `whatsapp_templates.md` — 6 templates: post-call, follow-up, confirm, payment, itinerary, on-trip

---

## MVP Status
- Live at wandrplan.com, closed group testing
- LinkedIn soft-launch post drafted (cryptic, no URL yet) — not yet posted, finalising a few things first
- Consultation confirmation email live
- Ankur notification email live (fires on every consultation request)
- 42+ submissions: 31% undecided, top dests: Italy, Japan, Malaysia, Maldives, Singapore, Spain, Switzerland, Thailand + domestic India
- Ankur's real photo live on why.html
- Newsletter subscriber capture live (footer + post-action popups on all pages)
- Instagram link live in all footers

---

## Post-MVP Backlog
- Own travel photos (replace Unsplash)
- Logo swap consideration — wandr_logo_new_notext.png currently in use, paper plane logo (Wandr_Logo.png) retired
- Blog / travel stories
- Sonnet 4.6 upgrade (web search for live flights) — deferred, Haiku performing well for MVP
- Brand rename consideration (pravasa.com, Itinera, Rovana)
- Broader LinkedIn launch
- Instagram Reels launch content (concepts explored: personal storytelling, product demo, Starbucks mug collection)
- First newsletter edition

---

## Context
- Built via Claude.ai, zero prior coding knowledge
- GoDaddy domain, DNS on Cloudflare (bypasses GoDaddy "Coming Soon")
- Resend domain verified, DKIM/SPF/DMARC set
- Emails land in Promotions (acceptable for MVP)

---

## Session Prompt
Paste this file in a new session with:
> Hi Claude! I'm continuing work on Wandr. Here's the full project context. Please continue helping me build and improve it.
