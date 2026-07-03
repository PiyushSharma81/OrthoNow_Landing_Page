# Namoza Developer Assignment — OrthoNow

Submission for Developer – Position 1 (Client Web + Martech).

## Repo structure

```
task-1-gtm-schema/
  gtm-event-schema.md        → full event table, booking-form dataLayer JSON, Google Ads conversion pick
task-2-landing-page/
  index.html                 → single self-contained landing page (HTML/CSS/vanilla JS)
  pagespeed-mobile-score.png → ADD THIS — see "Before you submit" below
task-3-integration-design/
  integration-design.md      → HubSpot / WhatsApp / Google Ads integration architecture
README.md                    → this file
```

## Before you submit — 3 things you still need to do

1. **Get the PageSpeed screenshot.** I can't generate a real Lighthouse score myself — it has to be measured against a live URL. Fastest path:
   - Push this repo to GitHub, then turn on **GitHub Pages** (Settings → Pages → Deploy from branch → `main` → `/task-2-landing-page`). You'll get a URL like `https://github.com/PiyushSharma81/OrthoNow_Landing_Page/`.
   - Run that URL through [PageSpeed Insights](https://pagespeed.web.dev/), **Mobile** tab.
   - Screenshot the score and save it as `task-2-landing-page/pagespeed-mobile-score.png`.
   - The page is built with zero external requests (no web fonts, no CDN scripts, no hosted images — everything is inline) specifically so it has a real shot at 90+; if it comes in lower, the likely culprit is whatever hosting platform you pick adding its own redirects/headers, not the page itself.

2. **Swap the placeholder numbers for real ones.** I flagged these directly in the code/docs — the clinic phone number (`+91 80 4567 8900`), the "4.8★ / 2,300+ patients" style stats, and the testimonial are placeholders standing in for real OrthoNow data, since I don't have their actual figures. Search each file for these before treating it as launch-ready.

3. **Record the Loom (max 8 min)**, per the brief: GTM schema decisions (~2 min) → live demo of the form firing the dataLayer push in the browser console (~3 min) → integration architecture answer (~3 min).
   - For the console demo: open `index.html` (or the hosted URL) → DevTools console → type `dataLayer` and hit enter first so they see it's an empty array → submit the form → run `dataLayer` again and show the `consultation_form_submitted` push sitting in it.

## A note on the "INTERVIEWER ONLY" sections in the brief

The PDF you were given includes internal grading notes not meant for candidates — worth flagging to Namoza. Since you've already seen them, it's genuinely worth being ready to explain (not recite) these two points live, because they'll likely ask directly:

- **Task 1:** GTM has no native trigger for "user is on step 2 of a form" — someone has to write `dataLayer.push()` into the form's own code at each step transition. That's the front-end developer's job; the GTM owner's job is to spec the exact event name/parameters/firing moment. This is explained in full in `task-1-gtm-schema/gtm-event-schema.md`.
- **Task 3:** HubSpot dedupes on email by default, not phone, and this form never collects email — so an explicit phone-based Search API check is required before every create. If two patients share a phone number with different names, the fallback in `task-3-integration-design/integration-design.md` never silently overwrites the existing name; it flags a conflict for manual review instead.
