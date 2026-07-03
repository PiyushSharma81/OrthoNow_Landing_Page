# Task 01 — GTM Event Schema · OrthoNow

**Scope:** GA4 + GTM event tracking for the main OrthoNow site (booking form, Call Now, WhatsApp, gated PDF, 9 clinic pages, blog). The landing page campaign form (`consultation_form_submitted`) is a separate, simpler event — see Task 02 — because it lives on a standalone campaign page, not the main site's 3-step booking flow.

## 1. Full Event Schema

| Event Name | Trigger Type (GTM) | Key Parameters | GA4 Destination |
|---|---|---|---|
| `view_clinic_page` | Page View – DOM Ready, condition: Page Path matches `/clinics/*` (9 pages) | `clinic_name`, `city`, `specialty_offered` | Engagement → Pages and screens (filtered by `clinic_name`); feeds a **"Visited [City] Clinic"** audience for geo-targeted remarketing |
| `call_now_click` | Click – All Elements, condition: Click URL starts with `tel:` OR Click Classes contains `cta-call` | `page_type` (home / clinic_page / landing_page), `clinic_name`, `cta_location` (header / sticky-bar / footer) | Events report; **"High-Intent – Call Clickers"** audience; candidate Google Ads conversion (see §3) |
| `whatsapp_click` | Click – All Elements, condition: Click URL contains `wa.me` | `page_type`, `page_location`, `referrer_page` | Events report; **"High-Intent – WhatsApp Clickers"** audience |
| `patient_guide_lead_submit` | Custom Event (dataLayer push) — see note below; do **not** use the native GTM Form Submission trigger if the gate form is AJAX-submitted, since that trigger only reliably fires on a real HTTP form POST | `form_location`, `lead_type` (`patient_guide`), `page_location` | Conversions report (marked as conversion); **"Content Leads – Nurture"** audience for email remarketing |
| `patient_guide_download` | GA4 Enhanced Measurement (automatic `file_download`) fired once the gate unlocks the asset, OR a Custom Event pushed immediately after `patient_guide_lead_submit` succeeds if the PDF opens in a new tab (Enhanced Measurement can miss that case) | `file_name`, `file_extension`, `page_location` | Engagement → File downloads |
| `blog_scroll_depth` | GTM built-in **Scroll Depth** trigger, vertical thresholds 25/50/75/90% | `percent_scrolled`, `article_title`, `article_category` | Engagement report; **"Engaged Blog Readers (75%+)"** audience for content remarketing |
| `booking_step_complete` (step 1) | Custom Event (dataLayer push) | `step_number`, `step_name`, `clinic_location`, `specialty` | GA4 Funnel Exploration — Step 1 |
| `booking_step_complete` (step 2) | Custom Event (dataLayer push) | `step_number`, `step_name`, `clinic_location`, `preferred_date`, `lead_source` | GA4 Funnel Exploration — Step 2; also feeds a **"Started, Didn't Finish Booking"** remarketing audience (fired step 1 or 2, never fired `booking_confirmed`) |
| `booking_confirmed` (step 3) | Custom Event (dataLayer push) | `step_number`, `step_name`, `clinic_location`, `specialty`, `booking_reference_id` | Conversions report — **primary conversion**, imported into Google Ads (see §3) |

**Privacy note:** none of the booking-form parameters above include Name or Phone. GA4's terms prohibit sending PII into event parameters, so those two fields stay in the form's own submission payload to the backend/CRM, never in the dataLayer.

---

## 2. Booking Form: Step-by-Step Drop-off Tracking

### Why this needs custom instrumentation, not a native GTM trigger

GTM has no built-in trigger type that understands "the user is now on step 2 of a form." Its native triggers (Click, Form Submission, Page View, Scroll Depth) all key off things that exist in the DOM or the browser's navigation — a click, a real page load, a scroll position. A multi-step booking form that advances client-side (no page reload between steps) doesn't produce any of those signals on its own.

**That means each step transition has to be instrumented by the front-end developer**, not configured purely inside GTM. Concretely: the developer adds a `window.dataLayer.push({...})` call directly into the code that runs when a user successfully completes a step — typically inside the "Next" button's click handler, gated behind client-side validation so it only fires when the step actually passed (not on every click attempt, which would inflate the funnel with failed submissions).

My side of this as the GTM owner is to hand the front-end dev a precise spec: exact event name, exact parameter keys/types, and exactly when in the code the push should fire. GTM's job is just to listen for a Custom Event trigger matching `event: booking_step_complete` (or `booking_confirmed`) and forward it to GA4 as an Event tag — it never inspects the form itself.

### GTM trigger per step

All three steps use the same trigger type: **Custom Event**, listening for `booking_step_complete` (steps 1–2) or `booking_confirmed` (step 3). Three separate GA4 Event tags fire off the same trigger (or one tag with dynamic Event Name from `{{DL - step_name}}`), so GA4 receives distinct, filterable events per step.

### DataLayer pushes (actual JSON, not pseudocode)

**Step 1 — location + specialty selected:**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}"
}
```

**Step 2 — contact details entered:**
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "{{clinic name}}",
  "preferred_date": "{{preferred date selected}}",
  "lead_source": "{{utm_source or referrer, captured on page load}}"
}
```
*(Name and phone are collected in this step but deliberately excluded from the push — no PII in the dataLayer.)*

**Step 3 — booking confirmed:**
```json
{
  "event": "booking_confirmed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}",
  "booking_reference_id": "{{booking id returned by backend}}"
}
```

### Surfacing drop-off in GA4 Funnel Exploration

1. New **Funnel Exploration**, type = **Closed funnel** (the steps are strictly linear — a user can't reach step 2 without step 1, so a closed funnel gives an honest drop-off rate rather than counting people who skipped in).
2. Define steps:
   - Step 1: `event = booking_step_complete` AND `step_number = 1`
   - Step 2: `event = booking_step_complete` AND `step_number = 2`
   - Step 3: `event = booking_confirmed`
3. Turn on **"Show elapsed time"** to see how long users take between steps — a slow step 1→2 transition points at UX friction, not just disinterest.
4. Breakdown dimension: `clinic_location` first (to see if drop-off concentrates at specific clinics — could mean no slots, bad copy, or a clinic-specific issue) and `device category` second (mobile drop-off at step 2 usually means the contact-details form is fighting the keyboard/viewport).
5. The trend view shows step-over-step % drop, which is the number the performance marketing team should watch weekly once campaigns go live.

---

## 3. Which Event to Import Into Google Ads

**Import `booking_confirmed`.**

Not `call_now_click`, not `whatsapp_click`, not `booking_step_complete`. Here's the reasoning:

- `booking_confirmed` is the only event that represents a **completed appointment** — the actual business outcome OrthoNow is paying for. Everything else is an engagement or intent signal that may or may not turn into a real patient.
- `call_now_click` and `whatsapp_click` are click-based, not outcome-based — a click can be a wrong number, a "just checking hours" call, or someone who calls and never books. Feeding these into Google Ads' automated bidding tells the algorithm to optimise for clicks that *look* like intent, which historically drags in cheaper-but-lower-quality traffic.
- `booking_step_complete` is tempting because it fires earlier (more volume, faster learning for Smart Bidding), but importing a partial-completion event as "the" conversion trains the algorithm to chase people who start the form, not people who finish it — exactly the problem OrthoNow already has with its 2.1% landing page.
- Optimising toward `booking_confirmed` means every rupee of ad spend is being bid against the one signal that's actually downstream of a filled clinic slot, which is what the co-founders will care about when they ask "did the ad spend turn into patients."
