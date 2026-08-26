# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

The licensed CRM is not used. Validated signals from the discovery round:

    18% weekly active adoption across licensed seats.
    11 min average task time to "update a deal" against a 2-min target.
    2.1 / 5 internal CSAT.
    Rep voice: "It's faster to keep my deals in a spreadsheet than to fight the CRM's eight required fields." / "Logging activity feels like data entry for management, not something that helps me sell." / "If it could just tell me who to call next, I'd open it every morning."

Validated hypothesis: reps abandon the CRM because it extracts data rather than returning value. If the tool (a) opens on a prioritized list of who to contact next, (b) requires no more than three fields to update a deal, and (c) captures activity passively from email/calendar/dialer, then weekly active adoption clears the 60% healthy target and update task time falls below 2 minutes — and it does so on a downward trend, not a one-off spike.

## Users & jobs

- **Primary user:** Accopunt Executive/Sales Rep
- **Job to be done:** When I start my day, I want to be told exactly who to contact and why, so I can spend my morning selling instead of deciding — and record what happened without it feeling like reporting.

## Scope

- **In:** Rep home with a scored Next Best Action queue, inline Call / Email / Log / Snooze.
"Start your day" briefing screen as the day's anchor, linking to Who's Next? and Active Deals.
Who's Next? dense worklist with view selector, filters, sorting, pagination.
Active Deals week-view timeline with filter sidebar and deal bars.
Deal update as a slide-over drawer with exactly 3 required fields (stage, next step, close date), all else optional/auto-filled, with an elapsed timer and Save & next.
Activity timeline of auto-captured items with source badges, plus a sub-30-second manual composer.
Role-gated admin adoption panel: adoption %, 6-month trend with 50%/60% bands, avg update time trend, seat table.
Loading (skeleton), empty ("Nothing on deck!"), and error ("Failure to load, please refresh.") states on the Who's Next? and Active Deals lists.
- **Out (explicitly):** Any backend, database, or persistence — state is in-memory and resets on reload.
Real authentication or server-side role enforcement.
Real integrations with Gmail, Google Calendar, or a dialer.
Real scoring model; scores are hand-authored constants.
Pipeline forecasting, quotas, territory management, mobile app, offline mode.
Email/calendar write-back (sending mail, booking meetings).

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | Rep landing screen shows a ranked Next Best Action queue, never a blank pipeline | Must | On load, / renders ≥8 ranked items, each with contact, company, reason chip, score, and inline actions. No empty pipeline grid is shown as the default view. |
| 2 | Deal update requires at most 3 fields | Should | Drawer marks only stage, next step, close date as required; save succeeds with those three alone. All other fields sit under a collapsed "Optional details" group. |

## Data & events

_What gets stored, what gets tracked._

Deal 	id, company, contact, title, amount, stage, nextStep, closeDate, lastTouchDays, owner, competitor, autoNotes 	Mocked — 8 seeded records
QueueItem 	id, dealId, score, action, reason, signal, when 	Mocked — scores are authored constants, no model
Activity 	id, type, source, auto, contact, company, summary, time, duration 	Mocked — no integration writes these
Meeting 	id, time, title, company, attendees, source 	Mocked — no calendar sync
ConnectedTool 	name, detail, status 	Mocked — statuses are static strings
AdoptionHistory 	month, adoption %, taskMinutes 	Mocked — 6 illustrative points (18%→61%, 11min→1.9min)
SeatUsage 	rep, role, lastLogin, sessions, dealsUpdated, avgUpdate 	Mocked

## Open questions

Which system of record backs this — do we replace the CRM or sit on top of it as a rep-facing client?
What actually drives the priority score, and who owns it? Rules engine first, or a model? Do reps see and adjust the weighting?
Which integrations do we build first (Gmail vs. Outlook, which dialer), and can we get read access to email/calendar under existing data-privacy policy?
If a deal can be saved with three fields, which downstream reports break, and who signs off on the fields we drop from required?
Is auto-captured activity written back to the system of record, or held locally? Does the rep get a review-before-write step?
Adoption denominator: all licensed seats, or only reps with an active quota?
Who owns the kill-switch decision, and what is the review cadence against the 50% floor?
Does the manual composer need call recording/transcription, or is an outcome chip sufficient?
Mobile: is a morning-queue mobile view required for the SDR persona in phase 1?
What happens to a snoozed item — how long, and does it resurface with a boosted score?
