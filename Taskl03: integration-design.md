# Task 03 — Integration Design: Landing Page → HubSpot / WhatsApp / Google Ads

## Architecture

I'd route the form through one lightweight serverless endpoint (Vercel or Cloudflare Worker) called directly via `fetch()` on submit — not a native HubSpot embed, and not Zapier/Make. A native embed can't run the phone-dedupe logic this needs; Zapier/Make add polling latency that eats into a 2-minute SLA, and their branching gets awkward for a conditional search-then-create/update step. A direct API call gives full control over sequencing and failure handling.

On submit: **(1)** the browser pushes `consultation_form_submitted` to the dataLayer immediately, so the Google Ads tag fires client-side, decoupled from anything downstream — if HubSpot or WhatsApp fails later, ad optimisation is unaffected. **(2)** The function calls the HubSpot Contacts Search API filtered on phone; a match gets `PATCH`, no match gets `POST` with Name, Phone, Clinic Preference, Source = "Google Ads - Consultation Landing Page," Lead Status = "New Enquiry." **(3)** In parallel, it calls Karix's WhatsApp API with a pre-approved template. Running HubSpot and WhatsApp concurrently (`Promise.allSettled`, not chained `await`s) means a slow HubSpot response never delays WhatsApp.

## Biggest failure point

HubSpot's default dedup runs on **email**, not phone — and this form never collects email. Without an explicit phone-based Search API check first, every repeat visitor becomes a duplicate. Worse: if two patients share a phone number with different names (common with shared household numbers in Indian healthcare lead gen), naively matching-and-overwriting silently loses the first patient's identity. Fallback: match on normalised phone (E.164), update Source/Lead Status freely, but never blindly overwrite Name — if the incoming name differs meaningfully, leave the record untouched and create a task flagging a possible shared-number conflict for manual review.

## SLA monitoring

The 2-minute SLA is most likely broken by Karix rate limits, template approval delays, or a cold-starting function. I'd compare the submit timestamp against Karix's delivery-status webhook, alert via Slack if elapsed time exceeds 90 seconds, then auto-retry with an SMS fallback and a flagged HubSpot task after two failed WhatsApp attempts, so a lead never silently falls through.

---
*Word count: ~360*
