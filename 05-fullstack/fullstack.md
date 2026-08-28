# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

https://preview--deal-navigator-aid.lovable.app/

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| orgs | id, name, licensed_seats int, plan text, timestamps. org_members — org_id, user_id, unique(org_id, user_id). | _____ |
| invites | token uuid default gen_random_uuid() unique, expires_at timestamptz default now() + interval '14 days', accepted_by uuid, accepted_at timestamptz, role app_role default 'rep'. next_step_templates — id, org_id, stage deal_stage, label text, sort_order int. | _____ |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

Table 	Rep 	Leadership 	Anon
profiles 	read/write own; read org peers' name + title 	same 	none
user_roles 	read own 	read org members' roles 	none
orgs / org_members 	read own org rows 	read own org rows 	none
deals, queue_items, activities, meetings 	full CRUD where owner_id = auth.uid() 	read where is_org_leader(org_id); no write 	none
deal_update_events 	insert + read own 	read org-wide (metrics only) 	none
connected_tools 	CRUD own 	none 	none
invites 	insert + read own sent invites 	read org-wide, may revoke 	read one row by token via a security-definer RPC only — never a table policy
adoption_targets, org_adoption_daily 	none 	read where leader 	none

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | Screens rendered against seeded arrays, so a brand-new account saw either hardcoded rows that weren't theirs or a bare page with zeroed metrics and no explanation. No org meant queries returned nothing and the UI just looked broken. | FirstRunState renders on Home and the Adoption panel when the account has no org — "Your workspace is empty" plus an "Invite a teammate" action. List surfaces use useListLoad, which distinguishes ready-but-empty from loading, showing the exact copy "Nothing on deck!" on Who's Next? and Active Deals. seed_demo_workspace() runs once per user on first sign-in, so metrics are non-zero and friendly instead of all zeros, and the header Demo mode toggle lets you flip to only your real records. |
| Bad / malicious input | Role came from a client-side toggle, /admin was reachable by anyone, invite tokens didn't exist, and any signed-in user could read all rows. | Roles live in user_roles and are read server-side via has_role; the header shows a read-only badge and /admin is gated by a real leadership check. Org-scoped RLS on every table (is_org_member, is_org_leader, is_leader_over) — you cannot read or write another user's records regardless of what the client sends. Invite tokens are validated by security-definer RPCs. invite_preview masks the email for signed-out visitors; accept_invite locks the row and rejects expired, already-accepted, revoked, declined, wrong-email, and unknown tokens. Malformed tokens are rejected by a UUID regex before the RPC is called. A partial unique index blocks duplicate pending invites, so replaying a send doesn't spam or overwrite. |
| Failure / offline | A failed fetch left a permanent spinner or an empty list indistinguishable from "no data," and a failed write silently dropped the change. | Fetch failures render the exact copy "Failure to load, please refresh." with a working Retry that refetches. Queue complete/snooze are optimistic with cache rollback: on error the row reappears and a toast offers "Retry." Manual activity logging and invite acceptance use the same retry-toast pattern, surfacing the underlying error message. ?state=loading|empty|error forces any of the three states so the behaviour stays demonstrable without breaking the backend. |

## Stress test results

_What you threw at it, and what held / broke._

Spam clicking, generated 20  updates in the activity log.
offline testing did not work.
