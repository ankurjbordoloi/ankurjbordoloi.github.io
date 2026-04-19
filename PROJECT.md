# Wandr — AI Travel Planner

## What it is
An AI-powered travel planning website that takes user inputs through a questionnaire and generates personalised travel itineraries. Users can also book a one-on-one consultation with a human travel consultant.

## Live URLs
- Website: https://wandrplan.com (also https://ankurjbordoloi.github.io)
- Cloudflare Worker: https://wandr-backend.ankurjbordoloi.workers.dev

## Pages
- `index.html` — Landing page
- `planner.html` — AI questionnaire + itinerary result + consultation booking
- `why.html` — Why Wandr + expert profile + consultation booking form

## Tech stack
- **Hosting:** GitHub Pages (repo: ankurjbordoloi/ankurjbordoloi.github.io)
- **Backend:** Cloudflare Worker (wandr-backend)
- **AI:** Anthropic Claude Haiku via Cloudflare Worker
- **Email:** Resend (domain: wandrplan.com, from: hello@wandrplan.com)
- **Form data:** Google Apps Script → Google Sheets (Wandr Leads)
- **Domain:** wandrplan.com (GoDaddy)

## Cloudflare Worker
Handles two endpoints:
- `POST /itinerary` — calls Anthropic API and returns AI-generated itinerary
- `POST /email` — sends full itinerary email via Resend

Environment secrets stored in Cloudflare:
- `ANTHROPIC_API_KEY`
- `RESEND_API_KEY`

## Google Sheets
File: Wandr Leads
- Sheet 1 — Consultation Leads (from planner.html and why.html booking forms)
- Sheet 2 — Planner Responses (questionnaire answers + email)

Google Apps Script handles:
- Writing new rows
- Updating email column when user submits email on result page
- Secret token validation (`wandr2026secure`)
- Honeypot spam check
- Rate limiting (20 submissions per hour)

## Questionnaire flow (planner.html)
Questions asked:
1. Who are you travelling with? (single select)
2. Where are you travelling from? (text)
3. Do you have a destination in mind? (single select)
4. Where would you like to go? (text — only shown if yes to above)
5. How long is your trip? (single select)
6. How do you like to travel? (single select)
7. What kind of experiences excite you? (multi select)
8. What is your travel budget like? (single select)

On completion:
- Answers submitted to Google Sheets (Sheet 2)
- AI itinerary generated via Cloudflare Worker
- Preview (4 lines) shown on page
- Full itinerary emailed when user submits their email

## Security
- Secret token on all Google Sheets form submissions
- Honeypot field on all forms
- Browser-side rate limiting (60 second cooldown)
- Server-side rate limiting in Apps Script (20/hour)
- Cloudflare Worker restricts requests to wandrplan.com origin only
- Invalid destination detection — returns friendly error message

## Design
- Dark background (#0f0f0f) with gold accents (#d4af37)
- Fonts: Cormorant Garamond (serif) + Jost (sans)
- Logo: Globe SVG icon + Wandr wordmark + "Curated travel" tagline
- Fully mobile responsive

## Still to do
- About / Contact page
- Server-side rate limiting already in Apps Script (Layer 3 done)
- Rename brand from Wandr (pravasa.com or similar — parked for now)
- Connect Cloudflare Worker + Resend for AI email when ready to scale

## Consultant
- Name: Ankur Bordoloi
- 25 countries visited across Asia, Middle East, Europe, North Africa, Americas
- Target clients: Solo travellers and couples/honeymoons
- Consultation booking collects: name, email, WhatsApp, preferred time, destination, notes
