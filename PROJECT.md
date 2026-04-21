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
Sections in order:
1. Nav: Globe SVG logo + "Wandr" wordmark + "Curated travel" tagline on left. "Home" and "Why Wandr" links on right. Active page highlighted in gold. Sticky with blur backdrop.
2. Hero: Full-bleed travel photo with dark overlay. "Your trip. Planned around you." headline. No buttons.
3. Two-path cards section — side by side:
   - Left card: "Free" badge + "AI planner" tag + "Plan my trip" heading + 4 bullet points + gold "Plan my trip" button → planner.html
   - Right card: "First session free" gold badge + "Personal consultation" tag + "Work with a consultant" heading + 4 bullet points + ghost "Learn more" button → why.html + pricing note below button
4. Photo strip: 3 destination photos (Japan, Dubai, Spain) with location labels
5. Features (3 columns) — white cards with gold top border, no icons
6. How it works (4 steps)
7. CTA section — full-bleed travel photo with dark overlay
8. Footer — dark background

### planner.html — Questionnaire + itinerary
9-question flow (8 visible if destination = No):
1. Who are you travelling with? (single select) — Solo, With a partner, Family with kids, Family with parents / joint family, Group of friends, Work / Group travel
2. Where are you travelling from? (text)
3. Do you have a destination in mind? (yes/no)
4. Where would you like to go? (text — only shown if yes)
5. How long is your trip? (single select)
6. Which month are you travelling? (dropdown — next 6 months from current date, "More than 6 months away", "Not sure yet")
7. How do you like to travel? (single select)
8. What kind of experiences excite you? (multi select)
9. What is your travel budget like? (single select)

On Generate:
- Answers saved to Google Sheets Sheet 2 (no email yet)
- AI itinerary generated via Cloudflare Worker
- Markdown stripped from itinerary before display
- 4-line preview shown, Send it button disabled until itinerary loads
- window._recommendedDest stores the AI-recommended destination for Undecided users

On Send it (email submit):
- Full itinerary emailed via Cloudflare Worker
- Second row written to Sheet 2 with all answers + email + recommended destination (for Undecided users)
- Destination stored as .trim().toUpperCase()

### why.html — Consultant page
- Full-bleed hero photo with dark overlay
- Why Wandr intro
- Consultant bio (Ankur Bordoloi, 25 countries) with stat cards (25 countries, 5 continents)
- Instagram link in bio section ("Follow my travels") — links to Ankur's Instagram
- Destination tags by region (hover highlights in gold)
- What a session covers (6 cards)
- Who it's for (2 cards)
- Comparison table (DIY vs AI only vs Wandr consultation)
- CTA section — full-bleed travel photo
- Consultation booking form → Sheet 1

---

## Design
- **Theme:** Bright and airy — warm off-white background (#faf8f4), warm cream sections (#f3ede4), white cards
- **Accent:** Gold (#c8992a)
- **Text:** Deep warm brown (#1a1612), muted (#6b5f52, #a09485)
- **Cards:** White with gold top border (2px), subtle box shadow
- **Fonts:** Cormorant Garamond (serif) + Jost (sans)
- **Logo:** Globe SVG + Wandr wordmark + "Curated travel" tagline
- **Animations:** Scroll fade-in on all sections, hero image zoom on load
- **Fully mobile responsive**
- **Photos:** Unsplash (free for commercial use under Unsplash License) — hero, CTA, and photo strip. To be replaced with Ankur's own travel photos post-MVP.

---

## Cloudflare Worker (Final version)

Handles two endpoints:
- `POST /itinerary` — calls Anthropic API, returns AI-generated itinerary + recommended destination (if undecided)
- `POST /email` — sends full itinerary email via Resend

Security:
- CORS restricted to wandrplan.com origin only
- Rate limiting: 20 requests per hour per IP (Cloudflare cache-based)
- Origin header validation

Environment secrets:
- `ANTHROPIC_API_KEY`
- `RESEND_API_KEY`

---

## AI Prompt (Cloudflare Worker)

Inputs passed: traveller type, origin, destination (or undecided), duration, travel month, pace, vibes, budget

Key behaviours:
- Invalid/nonsense destination → returns exactly `INVALID_DESTINATION`
- Misspelled destination → infers and proceeds
- No destination → AI picks the single best fit and starts itinerary with "Based on your responses, your recommended destination is X."
- Unrecognised origin → ignored, no clarification asked

Formatting rules:
- Starts with 20-25 word warm summary
- 1-5 days: morning / afternoon / evening with specific timings
- 6 days to 2 weeks: single flowing paragraph per day
- 2 weeks to 1 month: themed day clusters (3-5 days each)
- 1 month+: grouped by week
- Always completes full itinerary — never cuts off
- Max 800 words
- No markdown formatting (no #, ##, **, *)
- Kid-friendly if travelling with kids
- Senior-friendly if travelling with parents / joint family

Budget breakdown:
- Per person in local currency
- Categories: food, transport, accommodation, activities
- Presented as a simple list, not a table
- Uses travel month for seasonal cost estimates; uses booking lead time to contextualise fares
- Includes rough return flight estimate from origin (knowledge-based, labelled as estimate) with note to check Google Flights or Skyscanner for live prices

Booking lead time logic (calculated in Worker):
- Under 3 weeks: "booking very last minute — fares will be at peak"
- 4-8 weeks: "booking short notice — fares will be above average"
- 2-5 months: "booking reasonably ahead — fares near typical average"
- 5+ months: "booking well in advance — fares likely at their lowest"
- "More than 6 months away": treated as well in advance

---

## Email (Cloudflare Worker via Resend)
- From: Wandr <hello@wandrplan.com>
- Subject: "Your [DEST] itinerary is ready" or "Your personalised itinerary is ready"
- Dark themed HTML email with full itinerary + consultation CTA
- CTA links to wandrplan.com/why.html#consultant

---

## Google Sheets
File: "Wandr Leads"
- **Sheet 1 — Consultation Leads:** bookings from planner.html and why.html consultation forms. Columns include: Name, Email, WhatsApp, Preferred Time, Destination, Notes, Form (source identifier)
- **Sheet 2 — Planner Responses:** two rows per user — first on Generate (no email), second on Send it (with email + recommended destination)

Form field values (used to identify source):
- `'Planner Responses'` — questionnaire submissions (Sheet 2)
- `'Planner - Consultation'` — consultation booking from planner.html (Sheet 1)
- `'Why Page - Consultation'` — consultation booking from why.html (Sheet 1)

Apps Script:
- Secret token: `wandr2026secure`
- Honeypot check
- Rate limiting: 20/hour
- Routes to correct sheet based on Form field

---

## Security Layers
1. Secret token on all Sheet submissions (`wandr2026secure`)
2. Honeypot field (`hpot`) on all forms
3. Browser-side rate limiting (60s cooldown via localStorage)
4. Apps Script rate limiting (20/hour)
5. Cloudflare Worker rate limiting (20 requests/hour per IP)
6. CORS restricted to wandrplan.com only
7. Invalid destination detection — friendly error with restart option

---

## DNS (Cloudflare)
- 4x A records → GitHub Pages IPs (185.199.108-111.153) — DNS only
- CNAME www → ankurjbordoloi.github.io — DNS only
- MX + TXT SPF → Resend send subdomain
- TXT DKIM → resend._domainkey
- TXT DMARC → _dmarc (p=none)

---

## Consultant Details
- Name: Ankur Bordoloi
- 25 countries: India, Nepal, Thailand, Malaysia, Singapore, China, Japan, Turkiye, UAE, Saudi Arabia, Oman, Bahrain, Egypt, Morocco, Spain, Portugal, France, Monaco, Czech Republic, Slovakia, Hungary, Austria, Azerbaijan, Armenia, USA
- Target: Solo travellers (primary), couples/honeymoons, open to all
- First session: free discovery call (15-30 min, WhatsApp/Phone/Meet)
- Packages: Essential ₹2-3k / Standard ₹4-6k / Full planning ₹8-12k
- Instagram linked on why.html

---

## Page Titles
- index.html: "Wandr — Your Trip, Planned Around You"
- planner.html: "Plan My Trip — Wandr"
- why.html: "Work With a Travel Consultant — Wandr"

---

## Operational Documents
- `wandr_discovery_call.docx` — printable call sheet with writing lines, includes booking form fields, all 4 sections, packages table, red flags, closing script
- `whatsapp_templates.md` — 6 WhatsApp templates: post-call summary, 3-day follow-up, confirm proceed, payment received, send itinerary, on-trip check-in

---

## MVP Status
- Live at wandrplan.com, closed group testing underway
- Early data (42+ submissions): 13 Undecided (31%), top destinations: Italy (4), Japan (2), Malaysia (2), Maldives (2), Singapore (2), Spain (2), Switzerland (2), Thailand (2). Domestic India trips also appearing (Delhi, Goa, Udaipur).

---

## Conversion Tracking
- Generate → Email: calculable from Sheet 2 (Row 2 count ÷ Row 1 count)
- Consultation source: tracked via Form field in Sheet 1 ('Planner - Consultation' vs 'Why Page - Consultation')

---

## Parked for Post-MVP
- **Own travel photos** — replace Unsplash placeholder images with Ankur's actual travel photos
- **Better logo** — current globe SVG is simple. Options: AI generator (Looka/Canva), hire designer on Fiverr (₹2-5k)
- **Brand rename** — "Wandr" is working name. pravasa.com registered but blank. Other options considered: Itinera, Rovana. Decision parked until after MVP validation.
- **Blog** — travel stories section. Warm earthy palette already suits editorial layout. Would live in a `blog/` folder with individual post pages.
- **Upgrade to Sonnet 4.6** — 3x more expensive than Haiku but supports web search. Relevant for live flight price lookups. Not needed for MVP.
- **Broader launch** — after MVP feedback collected

---

## Important Context
- Built entirely through Claude.ai with no prior coding knowledge by Ankur
- wandrplan.com domain bought on GoDaddy, DNS moved to Cloudflare to bypass GoDaddy's "Launching Soon" page
- Resend free tier only sends to verified email — fixed by verifying wandrplan.com domain on Resend
- Emails land in Promotions tab (not spam) — acceptable for MVP
- DKIM, SPF, DMARC all verified on wandrplan.com
- Unsplash images confirmed safe for commercial use under Unsplash License (free, no attribution required)

---

## Session Prompt
To continue work in a new session, paste the contents of this file along with the following message:

> Hi Claude! I'm continuing work on Wandr. Here's the full project context. Please continue helping me build and improve it.
