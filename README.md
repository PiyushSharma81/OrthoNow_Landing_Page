# OrthoNow — Namoza Developer Assignment Submission

**Role applied for:** Developer – Position 1 (Client Web + Martech)
**Client:** OrthoNow (9 orthopaedic clinics — Bengaluru, Hyderabad, Chennai)

## Links for the reviewer

| | |
|---|---|
| **Live landing page** | https://piyushsharma81.github.io/OrthoNow_Landing_Page/ |
| **PageSpeed Insights report (Mobile)** | https://pagespeed.web.dev/analysis/https-piyushsharma81-github-io-OrthoNow_Landing_Page/p5jspmjplb?form_factor=mobile |
| **Loom walkthrough** | https://www.loom.com/share/ea849319745144489ed25ec84b40e718 |

---

## Repo structure

```
task-1-gtm-schema/
  gtm-event-schema.md        → GTM event table + booking-form dataLayer JSON
task-2-landing-page/
  index.html                 → single self-contained landing page
  pagespeed-mobile-score.png → PageSpeed Insights screenshot (Mobile)
task-3-integration-design/
  integration-design.md      → HubSpot / WhatsApp / Google Ads integration architecture
README.md                    → this file
```

---

## Task 01 — GTM Event Schema
*See `task-1-gtm-schema/gtm-event-schema.md`*

A complete event tracking plan for the OrthoNow site: 9 events covering the 3-step booking form, Call Now buttons, the WhatsApp widget, the gated Patient Guide download, all 9 clinic location pages, and blog scroll depth — each with trigger type, minimum 3 parameters, and its GA4 destination.

For the booking form specifically, the doc includes the actual dataLayer JSON for all 3 steps, an explanation of why those pushes have to be written by the front-end developer (GTM has no native way to detect a client-side form's internal step state), and how the resulting events get read as a drop-off funnel in GA4 Funnel Exploration. It closes with the single event recommended for import into Google Ads (`booking_confirmed`) and why that one over softer engagement signals like clicks or step completions.

## Task 02 — Landing Page Build
*See `task-2-landing-page/index.html`*

A replacement for OrthoNow's 2.1%-converting "Book a Consultation" page, built as a single HTML file with inline CSS and vanilla JS — no frameworks, no external requests (no web fonts, no CDN scripts, no hosted images), which is what let it score **100/100/100/100** (Performance / Accessibility / Best Practices / SEO) on PageSpeed Insights Mobile, well past the 90+ requirement.

Functionally: a 2-field form (Name + Phone) with real Indian mobile number validation, a `consultation_form_submitted` dataLayer push carrying 3 parameters that fires only on successful submit — never on page load — and a thank-you state that swaps in without a page reload. Mobile-specific details include a sticky bottom CTA bar, 16px input font sizing to avoid iOS auto-zoom, and full keyboard-focus visibility.

## Task 03 — Integration Design
*See `task-3-integration-design/integration-design.md`*

A 360-word written architecture for connecting the landing page form to HubSpot, Karix WhatsApp, and Google Ads: a serverless function called directly from the browser (over Zapier/Make or a native embed) running the HubSpot and WhatsApp calls in parallel rather than chained, with the Google Ads conversion firing client-side so it's decoupled from either backend call failing. It identifies HubSpot's email-based default deduplication as the single biggest failure point for a form that never collects email, with a phone-based Search API check and a manual-review fallback for same-number/different-name conflicts, plus a monitoring approach for the 2-minute WhatsApp SLA.

---

## A note on the data in Task 02

The patient counts, surgeon count, and ratings shown on the landing page are illustrative figures for this build, not verified OrthoNow numbers — flagging this directly rather than presenting them as confirmed client data. These should be swapped for actual, current figures before the page goes live on paid traffic.

---

## Submission checklist

- [x] Repo shared with himanshu@namoza.com (or set to public)
- [x] Loom recorded (GTM schema → live console demo → integration architecture)
- [ ] Email sent to naman@namoza.com — subject: `Developer Assignment - [Your Name]`, with repo link + Loom link
