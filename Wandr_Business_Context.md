# Wandr — Business Context & Session Summary
*Use this document to continue work in a new Claude chat. Paste the entire contents as your opening prompt.*

---

## PROMPT FOR NEW CLAUDE CHAT

> I'm Ankur Bordoloi, building **Wandr** (wandrplan.com) — an AI-powered travel planning product with two paths: a free AI itinerary generator and a paid personal consultation service. I'm based in Dubai (UAE Golden Visa holder, NRI). I have 25 countries of travel experience. The product is built entirely through Claude — I have no coding background.
>
> Please read this entire context document before responding to anything. All decisions below are final unless I explicitly say otherwise.

---

## WHO I AM

- **Name:** Ankur Bordoloi
- **Location:** Dubai, UAE (Golden Visa holder)
- **Background:** NRI, corporate job, no coding background
- **Travel experience:** 25 countries solo and group travel since 2017
- **First solo trip:** Türkiye 2018, 2 weeks — this is the origin story of Wandr

---

## WHAT WANDR IS

A travel consultancy for Indian travellers who want to travel internationally but don't know where to start. Two revenue streams:

1. **Free AI itinerary generator** — wandrplan.com (built, live)
2. **Paid personal consultation** — Ankur plans trips personally for clients

**Positioning:** Not a travel agent. Not a booking agency. A travel consultant — expert advice, personalised planning, built around the client.

**Target audience:** Indian travellers (based in India or abroad) who want to go somewhere new but feel overwhelmed by the planning process — visas, flights, hotels, tours, money, safety.

---

## PACKAGES & PRICING

| Package | Price | Includes |
|---|---|---|
| Essential | ₹3,500 | Itinerary, hotel recs, visa overview, food tips, 1 revision. Booking assist +₹1,500 |
| Standard | ₹6,500 | Essential + tours, flight routing, 2 revisions, 7-day WhatsApp. Booking assist +₹1,500 |
| Full Planning | ₹12,000 | Standard + pre-trip call, unlimited revisions, 30-day WhatsApp. Booking assist FREE |

**First consultation is always free.**

---

## TECH STACK

- **Frontend:** GitHub Pages — index.html, planner.html, why.html (hand-coded, no framework)
- **Backend:** Cloudflare Workers
- **AI:** Anthropic Claude Haiku
- **Email:** Resend
- **Logging:** Google Sheets (two-sheet: Sheet 1 = consultations, Sheet 2 = AI itinerary generates)
- **Domain:** wandrplan.com

**Rate limiting:** Separate localStorage keys per action — `wandr_last_email`, `wandr_last_booking`, `wandr_last_consult`

---

## BRAND

### Visual identity
- **Primary font:** Cormorant Garamond (serif, display) — Bold for headlines, Regular for wordmark, Italic for intro/pre-text
- **Secondary font:** Jost (sans-serif) — Light for body, Medium for pills/labels
- **Colours:**
  - Dark background: `#1A1612`
  - Gold accent: `#C8992A`
  - Cream/off-white: `#FAF8F4`
  - Muted text: `#A09485`
  - Border: `#E8E0D5`
- **Logo:** Compass star mark (wandr_logo_new_notext.png) + "WANDR" wordmark (Cormorant Garamond Regular, letter-spacing 7px) + "CURATED TRAVEL" tagline (Jost Light, letter-spacing 3.5px, gold)
- **Feel:** Premium, editorial, warm — not corporate, not generic AI aesthetic

### Key brand rules
- Never say "5 continents" — only "25 countries"
- Always use "traveller" not "traveler" (British spelling)
- Voice is warm, personal, honest — not salesy
- No slide numbers on carousel slides

---

## INSTAGRAM LAUNCH CAROUSEL — FINAL VERSION

### Status
9 slides built in Python (PIL) and saved as JPGs. Final files are named `LaunchCarousel_Slide1.jpg` through `LaunchCarousel_Slide9.jpg`.

### Carousel story arc
The carousel tells Ankur's personal travel story and leads to Wandr as the solution.

| Slide | File | Photo | Story beat | Key copy |
|---|---|---|---|---|
| 1 | LaunchCarousel_Slide1.jpg | Osaka Castle, Japan (autumn, moody) | Hook | "I almost didn't go. / Best decision I ever made." |
| 2 | LaunchCarousel_Slide2.jpg | Wadi Shab, Oman (canyon, turquoise water) | The setup | "For years, I kept waiting for friends to travel with me. / Spoiler: they never came. / So I went anyway." |
| 3 | LaunchCarousel_Slide3.jpg | Cappadocia balloon selfie, Türkiye | First solo trip pt 1 | "In 2018, I booked a solo trip to Türkiye. / 2 weeks. Alone. First time outside India without anyone I knew. / I was terrified." |
| 4 | LaunchCarousel_Slide4.jpg | Cappadocia fairy chimneys, Türkiye | First solo trip pt 2 | "And completely unprepared for how hard it would be... / Visas. Flights. Hotels. Tours. Money. Food. Getting around. Staying safe. / Nobody teaches you this." |
| 5 | LaunchCarousel_Slide5.jpg | Eiffel Tower at night, Paris | The learning | "You just figure it out. Trip by trip. Mistake by mistake. / 25 countries later — it's just a science to me now. / Daunting at first. Simple with the right process." |
| 6 | LaunchCarousel_Slide6.jpg | Chefchaouen phone wall, Morocco | The friends problem | "Now every time a friend plans a trip, they call me. / 'Where should I go?' 'How do I plan this?' 'Is this even possible?' / I love getting these calls. Honestly." |
| 7 | LaunchCarousel_Slide7.jpg | Rockefeller Center, NYC | The decision | "So I made it official. / Wandr is for every Indian traveller who wants to go somewhere new — / But doesn't know where to start." |
| 8 | LaunchCarousel_Slide8.jpg | New York Café, Budapest (ornate gold interior) | The invitation | "Solo. Couple. Group. / I've been there. I'll help you get there." |
| 9 | LaunchCarousel_Slide9.jpg | Barcelona skyline from above | CTA | "Your next trip starts with a conversation. / DM me 'PLAN' or visit / wandrplan.com" |

### Slide design specs (PIL/Python)
All slides: **1080 × 1350px**, JPEG quality 95

**Nav band (every slide):**
- White band top: `#FAF8F4` at opacity 225/255, height ~109px
- Bottom border: `#E8E0D5` at opacity 160
- Logo: compass PNG (black background removed via numpy mask), height = combined text block height (~65px)
- Gap between logo and text: 14px
- WANDR: Cormorant Garamond Regular 42px, `#1A1612`, letter-spacing 14px (drawn char-by-char)
- CURATED TRAVEL: Jost Light 16px, `#C8992A`, letter-spacing 7px (drawn char-by-char)

**Text positioning:**
- Slides 1: bottom-left
- Slides 2, 3, 4, 5: top-left (below nav band, 52px gap)
- Slides 6, 7, 8, 9: bottom-left

**Overlay styles:**
- `top` — dark at top fading to lighter mid, darker at bottom
- `bottom` — transparent top, dark gradient from 55% down
- `light` — very subtle throughout, heavier only at bottom
- `city` — light through mid (preserves cityscape), dark at bottom

**Top-left vignette:** Applied to slides 2 and 3 — radial darkening from top-left corner to improve italic text readability on bright photos

**Gold pill treatment:** For accent text lines on bright photos:
- Black background at opacity 130, tight padding (14px horizontal, 6px vertical)
- Measured using `textbbox` for pixel-perfect fit
- Text vertically centred using bbox top/bottom offsets
- Used on: "So I went anyway." (slide 2), "I was terrified." (slide 3), "Nobody teaches you this." (slide 4), "Simple with the right process." (slide 5), "But doesn't know where to start." (slide 7)

**Gold rule:** Rectangle 52px wide × 2px tall, `#C8992A`, used as section divider

**Headline hierarchy:**
- Pre-text / intro: Cormorant Garamond Italic, 34–38px, white at 73% opacity
- Headline: Cormorant Garamond Bold, 64–90px, white 100%
- Body: Jost Light, 21–26px, white at 84% opacity
- Gold accent: Jost Medium, 22–24px, `#C8992A`
- URL: Cormorant Garamond Bold, 42px, `#C8992A`

---

## KEY DECISIONS MADE THIS SESSION

### Carousel design
- **No slide numbers** on any slide
- **SVGs not required** — JPGs only for posting
- **Text positioning:** Top-left for slides with bright photos where face is in bottom half; bottom-left for darker photos or where face is in top half
- **Gold text on bright areas** gets a semi-transparent black pill background (opacity 130) instead of plain gold text — ensures readability on any photo
- **Photo rotation:** iPhone HEIC/JPG files often stored landscape — must rotate -90° and re-crop before use
- **Cover scaling** (not fit scaling) used for all photo crops — ensures no black bars at edges
- **Budapest photo (slide 8):** Required cover scaling at `max(scale_by_w, scale_by_h)` to eliminate vertical black bars

### Copy decisions
- **Slide 2 rewrite:** Original "my friends had a reason they couldn't make it" felt like attacking friends. Changed to "For years, I kept waiting for friends to travel with me. Spoiler: they never came. So I went anyway." — warm, self-aware, no blame
- **Slide 6 rewrite:** Changed from "they don't even know what questions to ask" (judgmental) to questions-in-quotes format ("Where should I go?" etc.) — warm, relatable, matches phone wall photo
- **Slide 9:** Removed "First consultation is free" line; kept "DM me PLAN or visit / wandrplan.com" with larger font
- **"But doesn't know where to start"** — capital B (not lowercase)
- **"Türkiye"** — always use ü (Unicode), not Turkey

### Instagram account
- Converting existing @ankurjbordoloi account to public (not creating a new brand account)
- Name field: "Ankur | Travel Consultant"
- Link: wandrplan.com
- Switch to Creator account, category: Travel & Tourism
- Best posting days: Tuesday–Thursday, 8–10pm IST

### Legal/financial (decided earlier, still pending)
- Operating informally for now
- Options under consideration: UAE Freelancer Permit (Shams ~AED 5,750/yr), Estonia OÜ (~€600-1,200/yr), Wyoming LLC (~$150-300/yr)
- Dubai activity code: 7020033 "Tourism and Recreation Consultants"

---

## FILES CREATED THIS SESSION

### Carousel JPGs (final, ready to post)
- `/mnt/user-data/outputs/wandr_carousel/LaunchCarousel_Slide1.jpg` through `LaunchCarousel_Slide9.jpg`

### Business documents (created in earlier sessions)
- `wandr_invoice_template.docx`
- `wandr_discovery_call_v2.docx`
- `wandr_packages.docx`
- `wandr_packages.pptx`
- `wandr_whatsapp_templates_v2.docx`

---

## PENDING / NEXT STEPS

- [ ] Post Instagram carousel (upload slides 1–9 in order)
- [ ] Update Instagram bio, name, profile photo, link
- [ ] Switch Instagram account to public
- [ ] Post carousel with caption (draft caption below)
- [ ] Website: remove any remaining "5 continents" references in why.html
- [ ] LinkedIn launch post
- [ ] Legal entity decision: Estonia OÜ vs UAE Freelancer Permit vs Wyoming LLC
- [ ] Google review link for WhatsApp Template 11

---

## INSTAGRAM CAPTION (DRAFT)

```
I almost didn't go to Turkiye.

Then I did. And it changed everything.

8 years and 25 countries later, I've realised something — 
planning a trip isn't hard once you know how.

But nobody teaches you the process.
Visas. Flights. Hotels. Getting around. Staying safe.
You just figure it out, trip by trip.

I did. And now every time a friend plans a trip, they call me.

So I made it official.

Wandr is for every Indian traveller who wants to go somewhere new but doesn't know where to start.

Solo, couple, or group — I've been there. I'll help you get there.

First consultation is free. DM me "PLAN" or visit wandrplan.com

#wandr #travelconsultant #indiantraveller #solotravel #travelplanning #internationaltravelfromIndia #traveltips #wandrplan
```

---

## PYTHON BUILD ENVIRONMENT

The slides were built using:
- Python 3 + PIL (Pillow)
- Font files converted from woff2 to TTF using fonttools + brotli
- TTF files stored at `/home/claude/fonts_ttf/`:
  - `cormorant-bold.ttf`
  - `cormorant-regular.ttf`
  - `cormorant-italic.ttf`
  - `jost-light.ttf`
  - `jost-regular.ttf`
  - `jost-medium.ttf`
- Background images stored at `/home/claude/wandr_slides/`
- Logo: `/mnt/user-data/uploads/wandr_logo_new_notext.png` (black bg removed via numpy)
- Build script: `/home/claude/build_all_slides.py`

**Key technique — removing black logo background:**
```python
logo_raw = Image.open('wandr_logo_new_notext.png').convert('RGBA')
logo_arr = np.array(logo_raw)
black_mask = (logo_arr[:,:,0] < 50) & (logo_arr[:,:,1] < 50) & (logo_arr[:,:,2] < 50)
logo_arr[black_mask, 3] = 0
logo_clean = Image.fromarray(logo_arr)
```

**Key technique — cover scaling (no black bars):**
```python
scale = max(W / iw, H / ih)
nw, nh = int(iw * scale), int(ih * scale)
resized = img.resize((nw, nh), Image.LANCZOS)
left = (nw - W) // 2
top = (nh - H) // 2
crop = resized.crop((left, top, left + W, top + H))
```

**Key technique — gold pill with tight bounds:**
```python
def draw_gold_pill(draw, x, y, text, fnt):
    bbox = draw.textbbox((0,0), text, font=fnt)
    b_left, b_top, b_right, b_bottom = bbox
    pad_x, pad_y = 14, 6
    draw.rectangle([(x-pad_x, y+b_top-pad_y), (x+(b_right-b_left)+pad_x, y+b_bottom+pad_y)], fill=(0,0,0,130))
    draw.text((x, y), text, font=fnt, fill=GOLD)
    return (b_bottom - b_top) + pad_y * 2
```
