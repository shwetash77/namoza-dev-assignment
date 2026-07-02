# Namoza Developer Assignment
**Candidate:** Sweta Sharma  
**Role:** Developer - Position 1 (Client Web + Martech)

---

## What's in this repo

Three deliverables for the OrthoNow client onboarding scenario.

### Task 1 — GTM Event Schema
`task1-gtm-schema.md`

Full event tracking schema covering every key interaction on OrthoNow's 
site — booking form funnel (with actual dataLayer JSON for each step), 
call clicks, WhatsApp widget, PDF download, clinic page views, and blog 
scroll depth. Includes the reasoning for why each event matters 
specifically for a clinic with both online and call-centre booking paths.

### Task 2 — Landing Page
`task2-landingpage/index.html`

Rebuilt the Book a Consultation page from scratch in vanilla HTML/CSS/JS. 
No frameworks, no dependencies. Form fires a dataLayer push on submit 
(visible in browser console), shows a thank-you state without page reload, 
and scores 100/100 on PageSpeed Mobile.

Live: https://shwetash77.github.io/namoza-dev-assignment/task2-landingpage/

### Task 3 — Integration Design
`task3-integration.md`

Written architecture for connecting the form to HubSpot + WhatsApp (Karix) 
+ Google Ads using Make as the automation layer. Covers the deduplication 
problem with HubSpot's phone-vs-email default, fallback design, and 
WhatsApp SLA monitoring.