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
- **Email:** Resend (domain: wandrplan.com, from: hello@wandrplan.com)
- **Form data:** Google Apps Script → Google Sheets ("Wandr Leads")
- **Analytics:** Cloudflare Web Analytics (EU excluded)

---

## Pages

### index.html — Landing page
1. Nav: Globe SVG logo + "Wandr" wordmark + "Curated travel" tagline. Fixed with blur backdrop, shadow on scroll.
2. Hero: Full-bleed travel photo with dark overlay. "Your trip. Planned around you." No buttons.
3. Two-path cards: AI planner (free) + Personal consultation (first session free)
4. Photo strip: Japan, Dubai, Spain
5. Features (3 columns) — white cards with gold top border. Feature body in burnt brown (#6b3f2a)
6. How it works (4 steps)
7. CTA — full-bleed photo
8. Footer — dark background

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

On Generate:
- Answers logged to Sheet 2
- If user typed destination then switched to "No", destination_name is cleared
- AI itinerary generated via Worker, markdown stripped, 4-line preview shown
- Send it button disabled until itinerary loads

On Send it:
- Full itinerary emailed via Worker
- Second row written to Sheet 2 with email + recommended destination
- Email title for undecided: "Your trip to a destination we recommend for you"

### why.html — Consultant page
- Full-bleed hero photo
- Consultant bio with stat cards (25 countries, 5 continents)
- "A" avatar placeholder — replace with photo post-MVP
- Instagram link, destination tags by region
- What a session covers (6 cards), who it's for (2 cards)
- Comparison table, CTA photo section
- Consultation booking form → Sheet 1

---

## Design
- **Theme:** Bright and airy — #faf8f4 bg, #f3ede4 sections, white cards
- **Accent:** Gold (#c8992a)
- **Text:** #1a1612, #6b5f52, #a09485; feature body #6b3f2a
- **Cards:** White, 2px gold top border, subtle shadow
- **Fonts:** Cormorant Garamond (serif) + Jost (sans, weight 300)
- **Logo:** Globe SVG + wordmark (Wandr_Logo.png paper plane ready for post-MVP)
- **Animations:** Scroll fade-in, hero zoom on load
- **Photos:** Unsplash (safe for commercial use, no attribution needed)

---

## Cloudflare Worker — Endpoints

- `POST /itinerary` — Anthropic API → returns itinerary + recommended_dest
- `POST /email` — sends itinerary email via Resend
- `POST /consultation` — sends consultation confirmation email via Resend

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

Both emails: #faf8f4 bg, dark brown logo, gold tagline, #6b5f52 body text, #c8992a CTA button

### Itinerary email
- Subject: "Your [DEST] itinerary is ready" / "Your personalised itinerary is ready"
- Title: "Your trip to [DEST]" / "Your trip to a destination we recommend for you"
- CTA → why.html#consultant

### Consultation confirmation email
- Subject: "Your consultation request is confirmed — Wandr"
- Title: "We've received your request, [Name]." (auto title-cased)
- What happens next + prep checklist + Wandr Team sign-off
- CTA → why.html

---

## Google Sheets — "Wandr Leads"

- Sheet 1: Consultation leads (from planner.html and why.html)
- Sheet 2: Planner responses (2 rows per user — Generate + Send it)

Form field values:
- `'Planner Responses'` → Sheet 2
- `'Planner - Consultation'` → Sheet 1
- `'Why Page - Consultation'` → Sheet 1

Apps Script: secret token `wandr2026secure`, honeypot, 20/hour rate limit

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

---

## DNS (Cloudflare)
- 4x A records → GitHub Pages (185.199.108-111.153)
- CNAME www → ankurjbordoloi.github.io
- MX + SPF + DKIM + DMARC → Resend

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
- LinkedIn soft-launch post drafted (cryptic, no URL yet)
- Consultation confirmation email live
- 42+ submissions: 31% undecided, top dests: Italy, Japan, Malaysia, Maldives, Singapore, Spain, Switzerland, Thailand + domestic India

---

## Post-MVP Backlog
- Own travel photos (replace Unsplash)
- Ankur's photo in why.html bio
- Logo swap to Wandr_Logo.png
- Blog / travel stories
- Sonnet 4.6 upgrade (web search for live flights)
- Brand rename consideration (pravasa.com, Itinera, Rovana)
- Broader launch

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
