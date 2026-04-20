# Wandr — AI Travel Planner

## What it is
An AI-powered travel planning website that takes user inputs through a questionnaire and generates personalised travel itineraries. Users can also book a one-on-one consultation with a human travel consultant (first session free).

## Live URLs
- Website: https://wandrplan.com (also https://ankurjbordoloi.github.io)
- Cloudflare Worker: https://wandr-backend.ankurjbordoloi.workers.dev

## Pages
- `index.html` — Landing page with two-path section (AI planner vs consultation)
- `planner.html` — AI questionnaire + itinerary result + consultation booking
- `why.html` — Why Wandr + expert profile + destinations + consultation booking form

## Tech stack
- **Hosting:** GitHub Pages (repo: ankurjbordoloi/ankurjbordoloi.github.io)
- **Custom domain:** wandrplan.com (GoDaddy, DNS managed via Cloudflare)
- **Backend:** Cloudflare Worker (wandr-backend)
- **AI:** Anthropic Claude Haiku 4.5 via Cloudflare Worker
- **Email:** Resend (domain: wandrplan.com, from: hello@wandrplan.com)
- **Form data:** Google Apps Script → Google Sheets (Wandr Leads)
- **Analytics:** Cloudflare Web Analytics (EU excluded)

## Cloudflare Worker
Handles two endpoints:
- `POST /itinerary` — calls Anthropic API and returns AI-generated itinerary
- `POST /email` — sends full itinerary email via Resend

Security:
- CORS restricted to wandrplan.com origin only
- Rate limiting: 20 requests per hour per IP (Cloudflare cache-based)

Environment secrets stored in Cloudflare:
- `ANTHROPIC_API_KEY`
- `RESEND_API_KEY`

## Google Sheets
File: Wandr Leads
- **Sheet 1 — Consultation Leads:** bookings from planner.html and why.html consultation forms
- **Sheet 2 — Planner Responses:** questionnaire answers submitted on Generate, plus a second row with email when user clicks Send it

Google Apps Script handles:
- Routing to correct sheet based on Form field
- Writing new rows with correct column headers
- Secret token validation (`wandr2026secure`)
- Honeypot spam check
- Rate limiting (20 submissions per hour)

## Questionnaire flow (planner.html)
Questions asked:
1. Who are you travelling with? (single select — includes "Family with parents / joint family")
2. Where are you travelling from? (text)
3. Do you have a destination in mind? (single select)
4. Where would you like to go? (text — only shown if destination = yes)
5. How long is your trip? (single select)
6. How do you like to travel? (single select)
7. What kind of experiences excite you? (multi select)
8. What is your travel budget like? (single select)

On completion:
- Answers submitted to Google Sheets Sheet 2
- AI itinerary generated via Cloudflare Worker
- Preview (4 lines) shown on page — Send it button disabled until itinerary loads
- Full itinerary emailed when user submits email
- Second row with all answers + email written to Sheet 2

## AI Prompt (Cloudflare Worker)
- Persona: experienced travel consultant, 25 countries
- Handles misspelled destinations, invalid destinations (returns INVALID_DESTINATION), no destination (AI recommends one)
- Starts with 20-25 word warm summary of the trip
- Formatting tiered by trip length:
  - 1-5 days: morning / afternoon / evening with timings
  - 6 days to 2 weeks: single flowing paragraph per day
  - 2 weeks to 1 month: themed day clusters (3-5 days each)
  - 1 month+: grouped by week
- Includes insider tips, real place names, local costs, transport tips
- Kid-friendly for families with kids
- Senior-friendly for family with parents / joint family
- Ends with budget breakdown per person in local currency
- No markdown formatting
- max_tokens: 3000
- Model: claude-haiku-4-5-20251001

## Email (Cloudflare Worker via Resend)
- From: Wandr <hello@wandrplan.com>
- Subject: "Your [DEST] itinerary is ready" or "Your personalised itinerary is ready"
- Dark themed HTML email with full itinerary + consultation CTA
- CTA links to wandrplan.com/why.html#consultant

## Security
- Secret token on all Google Sheets submissions (`wandr2026secure`)
- Honeypot field on all forms (`hpot`)
- Browser-side rate limiting (60 second cooldown via localStorage)
- Server-side rate limiting in Apps Script (20/hour)
- Cloudflare Worker rate limiting (20 requests/hour per IP)
- CORS restricted to wandrplan.com origin only
- Invalid destination detection — friendly error with restart option

## Design
- Dark background (#0f0f0f) with gold accents (#d4af37)
- Fonts: Cormorant Garamond (serif) + Jost (sans)
- Logo: Globe SVG icon + Wandr wordmark + "Curated travel" tagline
- Fully mobile responsive
- UI retheme (lighter, travel vibe) parked for post-MVP

## DNS (via Cloudflare)
- 4x A records → GitHub Pages IPs (185.199.108-111.153) — DNS only
- CNAME www → ankurjbordoloi.github.io — DNS only
- MX + TXT SPF → Resend (send subdomain)
- TXT DKIM → resend._domainkey
- TXT DMARC → _dmarc (p=none)

## Consultant
- Name: Ankur Bordoloi
- 25 countries visited: India, Nepal, Thailand, Malaysia, Singapore, China, Japan, Turkiye, UAE, Saudi Arabia, Oman, Bahrain, Egypt, Morocco, Spain, Portugal, France, Monaco, Czech Republic, Slovakia, Hungary, Austria, Azerbaijan, Armenia, USA
- Target clients: Solo travellers (primary), couples/honeymoons, open to all
- First session: free discovery call (15-30 min, WhatsApp / Phone / Meet)
- Paid packages: Essential (Rs 2-3k) / Standard (Rs 4-6k) / Full planning (Rs 8-12k)
- Instagram linked on why.html consultant section

## Operational documents
- `wandr_discovery_call.docx` — printable call sheet with writing lines
- `whatsapp_templates.md` — 6 WhatsApp follow-up message templates

## Current status
- MVP live at wandrplan.com
- Closed group testing underway

## Post-MVP backlog
- UI theme change — lighter, more travel vibe
- Better logo
- Brand rename (pravasa.com or similar being considered)
- About page if needed
