# Task 01 — GTM Event Schema (OrthoNow)

## Event Schema Table

| # | Event Name | Trigger Type | Key Parameters | Feeds Into (GA4) |
|---|---|---|---|---|
| 1 | `location_page_view` | Page View — URL matches `/clinics/*` | `clinic_name`, `clinic_city`, `specialty_listed` | Clinic-level traffic report — compares views vs. bookings per location to guide ad spend allocation across the 9 clinics |
| 2 | `call_now_click` | Click — Click ID/Class matches `call-now-btn` | `clinic_name`, `page_location` (homepage / clinic page / landing page), `phone_number_clicked` | Conversion event, imported into Google Ads — a call click is a strong intent signal for a healthcare business |
| 3 | `whatsapp_widget_click` | Click — Click Class matches `whatsapp-widget` | `page_location`, `device_type` | Engagement report — informs whether WhatsApp deserves a dedicated ad campaign objective |
| 4 | `patient_guide_download_started` | Form Submission — gated PDF form | `lead_source_page`, `form_location` | Top-of-funnel lead capture report — softer leads, tracked separately from booking funnel for nurture campaigns |
| 5 | `patient_guide_download_completed` | Custom Event — dataLayer push after successful file delivery | `guide_topic` | Confirms the PDF actually delivered, not just that the form submitted (delivery can fail silently) |
| 6 | `blog_scroll_75` | Scroll Depth — 75% trigger | `article_title`, `article_category` | Content engagement report — identifies which blog topics actually hold attention vs. just attract clicks |
| 7 | `booking_step_complete` (step 1) | Custom Event — dataLayer push | `step_number: 1`, `step_name: "location_specialty_selected"`, `clinic_location`, `specialty` | GA4 Funnel Exploration — Step 1 of booking funnel |
| 8 | `booking_step_complete` (step 2) | Custom Event — dataLayer push | `step_number: 2`, `step_name: "contact_details_entered"`, `preferred_date` | GA4 Funnel Exploration — Step 2 of booking funnel |
| 9 | `booking_confirmed` | Custom Event — dataLayer push on final confirmation | `step_number: 3`, `clinic_location`, `specialty`, `booking_value` (if applicable) | Primary conversion — imported into Google Ads as the optimization goal |

---

## Booking Form Funnel — Step-Level Tracking

GTM cannot natively detect progress through a multi-step form, since each "step" is a UI state change within a single page, not a new URL or a generic click GTM recognizes out of the box. This requires the **front-end developer** to manually fire a `dataLayer.push()` at the moment each step completes. GTM is then configured with a **Custom Event trigger** that listens for each specific `event` name in the dataLayer and fires the corresponding GA4 tag.

### dataLayer push — Step 1 (Location + Specialty selected)
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}"
}
```

### dataLayer push — Step 2 (Contact details entered)
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "preferred_date": "{{preferred date}}"
}
```

### dataLayer push — Step 3 (Booking confirmed)
```json
{
  "event": "booking_confirmed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}"
}
```

### How this surfaces in GA4 Funnel Exploration
A funnel is built in GA4 Explore with three ordered steps, each matched to one of the events above (`booking_step_complete` with `step_number = 1`, `step_number = 2`, then `booking_confirmed`). GA4 then visualizes, step by step, what percentage of users who completed one step proceeded to the next — making drop-off points immediately visible (e.g. "60% of users who select a clinic and specialty never reach the contact details step"). This pinpoints exactly where the form is losing patients, rather than only knowing the overall conversion rate.

### Who writes the dataLayer push?
The **front-end developer** writes it — GTM only listens for events that already exist in the dataLayer; it has no native way to detect an in-page state change like "step 2 of a form was completed." Briefing the front-end dev for Step 2 specifically would mean: "When the user successfully submits the name/phone/preferred-date fields and the form transitions to Step 3, fire `window.dataLayer.push({event: 'booking_step_complete', step_number: 2, step_name: 'contact_details_entered', preferred_date: <value>})` immediately after validation passes, before moving to the next step's UI."

---

## Conversion Action to Import into Google Ads

**`booking_confirmed`**

This is chosen over `call_now_click` or `patient_guide_download_completed` because Google Ads' bidding algorithms optimize toward whichever action is marked as the conversion goal. A confirmed booking is the actual business outcome — a patient who will generate revenue — while a call click or guide download are upstream, lower-intent signals. Importing a softer action as "the" conversion would train Google's automated bidding to chase volume over quality, inflating lead count while diluting the lead-to-patient conversion rate that actually matters to OrthoNow.
