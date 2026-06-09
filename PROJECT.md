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
- Newsletter popup shown 1.2s after success (email pre-filled) — only shown once per session via `_nlPopupShown` flag
Consultation section:
- Divider: "Not sure yet? Talk to a travel expert!"
- Badge: "First session free"
- On consultation submit: Sheet + Worker fire in parallel (Promise.all) with 10s timeout; success shown regardless
- Newsletter popup shown 1.2s after submit — only if not already shown this session
### why.html — Consultant page
- Full-bleed hero photo
- Consultant bio with Ankur's real photo (ankur.jpg), stat cards (25 countries)
- Instagram link (@ankurjbordoloi), destination tags by region
- What a session covers (6 cards), who it's for (2 cards)
- Comparison table, CTA photo section
- Consultation booking form → Sheet 1
- On submit: Sheet + Worker fire in parallel (Promise.all) with 10s timeout; success shown regardless
- Newsletter popup shown 1.2s after submit (email pre-filled)
- Consultant info body text: 12px (intentionally smaller than standard)
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
- `POST /email` — sends itinerary email via Resend (BCC: hello@wandrplan.com)
- `POST /consultation` — sends consultation confirmation + notification to ankur@wandrplan.com (BCC: hello@wandrplan.com)
- `POST /subscribe` — logs newsletter subscriber to Sheet 3
Security: CORS (wandrplan.com only), origin validation, 20 req/hour/IP rate limit
 
Secrets: `ANTHROPIC_API_KEY`, `RESEND_API_KEY`
 
Constant at top of worker: `const BCC = 'hello@wandrplan.com'`
 
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
Budget: per person, local currency, simple list (food, transport, accommodation, activities + Skyscanner note for flights)
 
Booking lead time:
- <3 weeks: peak fares
- 4-8 weeks: above average
- 2-5 months: near average
- 5+ months / "More than 6 months away": lowest
---
 
## Emails — Unified Light Theme
 
All emails: #faf8f4 bg, compass logo image + WANDR wordmark + gold tagline (shared LOGO_BLOCK constant in Worker), #6b5f52 body text, #c8992a CTA button.
BCC on all outgoing emails: hello@wandrplan.com
 
Logo in emails: hosted at https://raw.githubusercontent.com/ankurjbordoloi/ankurjbordoloi.github.io/main/wandr_logo_new_notext.png
 
### Itinerary email
- Subject: "Your [Dest] itinerary is ready — Wandr" (normal case, not all-caps) / "Your personalised itinerary is ready — Wandr"
- Title: "Your trip to [Dest]" / "Your trip to a destination we recommend for you"
- CTA → why.html#consultant
### Consultation confirmation email
- Subject: "Your consultation request is confirmed — Wandr"
- Title: "We've received your request, [Name]." (auto title-cased)
- What happens next + prep checklist + Wandr Team sign-off
- CTA → why.html
### Ankur notification email (internal)
- Fires on every consultation form submit (planner.html or why.html)
- Sent to: ankur@wandrplan.com (no BCC)
- Subject: "New consultation request — [Name]"
- Contains: name, email (clickable), WhatsApp, preferred time, destination, note
---
 
## Google Sheets — "Wandr Leads"
 
- Sheet 1 (Consultation Leads): from planner.html and why.html
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
- Popup copy: label "Wandr Newsletter", subtext explicitly mentions newsletter
- Dismiss: just closes, shows again on next visit (no localStorage)
- Source tracked: `footer`, `popup-planner`, `popup-why`, `popup-index`
- Sending tool: Mailchimp or Brevo free tier — to set up when first edition is ready
- Existing planner users (Sheet 2 emails) to be added at launch with a one-time welcome note
---
 
## Footer — All Pages
- Dark background (#1a1612)
- Left: Wandr wordmark + "Curated travel"
- Right: © year + "Built with intention." + Instagram link directly below (same text block)
- Instagram link: gold SVG icon + "Follow my travels" — right-aligned, below copyright
- Newsletter strip below footer inner (email input + Subscribe button)
---
 
## Email Setup (Gmail)
- hello@wandrplan.com → forwards to ankurjbordoloi@gmail.com via Cloudflare Email Routing
- ankur@wandrplan.com → forwards to ankurjbordoloi@gmail.com via Cloudflare Email Routing
- Both set up as "Send mail as" in Gmail via Gmail SMTP + App Password
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
- Itinerary Word doc template: Wandr-branded .docx with gold dividers, Georgia/Arial fonts, day labels in gold — generated per client
---
 
## Consultation Itinerary Work
- Ankur uses Claude to draft itineraries for real clients, then refines with personal knowledge
- Word docs generated in Wandr brand theme (WANDR header, gold dividers, bullet points with gold en-dash)
- First real client itinerary: Assam & Meghalaya (Guwahati → Shillong → Cherrapunji), 4 days
---
 
## MVP Status
- Live at wandrplan.com, closed group testing
- LinkedIn soft-launch post drafted — not yet posted
- 42+ submissions: 31% undecided, top dests: Italy, Japan, Malaysia, Maldives, Singapore, Spain, Switzerland, Thailand + domestic India
- Ankur's real photo live on why.html
- Newsletter subscriber capture live (footer + post-action popups)
- Instagram link live in all footers
- BCC on all outgoing emails (hello@wandrplan.com) for deliverability monitoring
- Email subject lines fixed — normal case + "— Wandr" suffix
---
 
## Post-MVP Backlog
- Own travel photos (replace Unsplash)
- Logo swap consideration
- Blog / travel stories
- Sonnet upgrade with web search for live flights — deferred
- Brand rename consideration (pravasa.com, Itinera, Rovana)
- First newsletter edition
---
 
## Instagram Launch — May 2026
 
### Profile
- Handle: @ankurjbordoloi — public, 614 followers (personal network, not travel audience yet)
- Bio: ✈️ 25 countries and counting... / 🪬 I plan trips people actually remember / 📍 Based in Dubai / 🌍 DM to plan yours
- Link: wandrplan.com
### Posting Strategy
- 2x per week — Wednesday + Saturday, 5pm UAE time
- Reels for reach, Carousels do NOT get distributed to non-followers on small accounts — reels only for growth
- Every 4th post ties back to Wandr without being salesy
- Editing tool: Edits by Instagram (no watermark, native boost)
### Audience (from insights)
- Age: 90.7% are 25-34
- Gender: ~55.8% women, 44.2% men
- Top countries: India 75.9%, USA 10.9%, UAE 6.1%
### Key Content Learnings
- Reels reach 55-63% non-followers consistently — algorithm distributing well
- Carousels reach 0% non-followers — avoid for growth
- Skip rate 38-54% across reels — hook quality is the #1 improvement lever
- 0 follows despite steady profile visits — need more content volume
- Never use AI generated covers
- Lead with most visually unusual/striking frame
- Captions should NOT repeat what's in the video
- Short personal voice outperforms descriptive listicle captions
- Soft CTAs work better than hard sells to cold audiences
- Tag real venues naturally in captions
### Posts Live
| Post | Format | Views | Non-followers | Saves | Follows |
|------|--------|-------|---------------|-------|---------|
| Launch carousel — Türkiye origin story | Carousel | 790 | 1.4% | 0 | 0 |
| Japan autumn reel — hot take format | Reel | 307 | 62.5% | 0 | 0 |
| Japan photo dump (Rap God trend) | Reel | 255 | 59.1% | 0 | 0 |
| Japan food reel | Reel | 269 | 53.4% | 1 | 0 |
| Japan city guide reel — 2 weeks | Reel | 309+ | 63.2% | 2 | 0 |
| Japan conbini carousel | Carousel | 104 | 0% | 1 | 0 |
 
### Content Pipeline
- Morioka / Grand Seiko Studio reel — in production (next to post, Wednesday)
- Europe content (Austria, France, Spain) — not started
---
 
## Context & Workflow
- Built via Claude.ai, zero prior coding knowledge
- PROJECT.md + WANDR_LAUNCH.md uploaded to Claude Project for persistent context — update at end of each session
- GoDaddy domain, DNS on Cloudflare
- Resend domain verified, DKIM/SPF/DMARC set
- Emails land in Promotions (acceptable for MVP)
---
 
## Session Prompt
This file is uploaded to the Claude Project. In new sessions just say:
> Hi Claude! I'm continuing work on Wandr. Please read the project file for full context.
 
For Instagram/marketing sessions:
> Hi Claude! I'm continuing work on Wandr marketing and Instagram. Please read WANDR_LAUNCH.md for full context. I'm currently working on [describe what you're doing].
