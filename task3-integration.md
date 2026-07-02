 ## Integration Design

## How I'd wire this up

Let's start with what's already done. The Google Ads conversion doesn't 
need any new work — the `dataLayer.push` in Task 2 already fires 
`consultation_form_submitted` the moment someone hits submit. I'd just 
import that event into Google Ads from GA4 and done. No backend, no 
extra code.

The heavier lift is connecting the form to HubSpot and getting that 
WhatsApp message out within 2 minutes. For this I'd use **Make** 
(formerly Integromat) as the middleware. 

Why Make and not the others? A direct HubSpot API call needs a server 
sitting in the middle — that's extra infrastructure I don't want to 
spin up for a single form. Zapier works but gets expensive fast and 
doesn't handle conditional branching as cleanly. Make hits the right 
balance of flexibility and cost for this setup.

Here's the actual sequence once someone submits:

1. Form submit triggers a Make webhook with Name, Phone, Clinic Preference
2. Make searches HubSpot: does a contact with this phone number already exist?
3. Yes → update the existing record, flip Lead Status back to 'New Enquiry'
4. No → create a fresh contact, tag Source as 
   'Google Ads - Consultation Landing Page', set Lead Status to 'New Enquiry'
5. Either way, Make immediately hits the Karix API and the WhatsApp 
   confirmation goes out

## The thing most people miss — deduplication

Here's a real problem I've run into before: HubSpot's default 
deduplication key is email, not phone. This form doesn't collect email 
at all. So if the same patient submits twice, or two family members use 
the same number, HubSpot creates separate duplicate contacts unless 
you explicitly search by phone before creating anything. 

This is why the search step in Make isn't optional — skip it and the 
CRM fills up with junk data within a week. In Indian healthcare lead gen 
this comes up constantly because patients rarely give their email.

## Where this is most likely to break

Honestly, the Make webhook. If Make has downtime or the webhook times out, 
that lead just disappears — no error, no alert, nothing. 

My fallback is simple: I'd add a parallel step in the Make scenario that 
sends every submission as an email to a monitored inbox. It's not elegant 
but it means zero leads fall through even if the main automation fails.

## Keeping WhatsApp under 2 minutes

In practice three things tend to blow this SLA:

- Make running slow under load (rare but happens)
- Karix being down (less rare than you'd hope)
- Phone numbers coming in without the +91 prefix — Karix will 
  reject these silently and you'll never know

I'd handle the formatting issue in Make before the Karix call ever fires. 
For monitoring I'd use Make's native error notifications plus Karix's 
delivery webhook — if I don't get a delivery confirmation back within 
3 minutes of submission, the scenario retries automatically.