# Concrete Pump Partners — Interactive Operator Training (Prototype v2.2)

## What works now
- Trainer and trainee role views, with one-device handoff mode.
- Full nine-page evaluation structure: road test, pump controls, jobsite operation, procedural review, and individually acknowledged safety checklist.
- Trainer dropdowns for Yes/No and Proficient/Needs Improvement, re-evaluation status and date, comments, road test result.
- Trainee acknowledgment dropdown plus required initials for each final-procedure and safety statement and separate drawn trainer / trainee signatures.
- Automatic browser-local saving, same-browser multi-tab updates, JSON export/import, print-to-PDF report, and installation-ready PWA.
- Optional cross-device synchronization through a Supabase backend (requires configuration).

## Open locally
Unzip and open `index.html` in a modern browser. This works as a local single-device prototype, although PWA installation and cross-device features need HTTPS hosting.

## Host the app
Deploy this folder to a static web host with HTTPS (for example Cloudflare Pages, Netlify, or GitHub Pages). Then open the URL on tablets/phones and choose **Add to Home Screen** or **Install app**.

## Set up live syncing for a pilot
1. Create a Supabase project. In its SQL editor, run the following *pilot-only* table and policies:

```sql
create table if not exists public.training_sessions (
  id uuid primary key,
  data jsonb not null,
  updated_at timestamptz not null default now()
);
alter table public.training_sessions enable row level security;
create policy "pilot read" on public.training_sessions for select to anon using (true);
create policy "pilot insert" on public.training_sessions for insert to anon with check (true);
create policy "pilot edit" on public.training_sessions for update to anon using (true) with check (true);
```

2. **Do not put actual personnel records in this pilot configuration.** The policies above make sessions accessible to anyone who has the project's public key and record ID, with no participant authentication. This is only an isolated demo environment.
3. In app Setup, select **Two devices / live sync**, enter Supabase project URL and the publishable/anon key (never a `service_role` secret), and choose **Connect/create session**.
4. Host the app at an HTTPS address and use **Copy trainee session link** to open it on the second device. Both devices must be configured to connect to the same backend (enter the project URL/key on the trainee's device too, or provision a protected company build).
5. The app polls for updates roughly every two seconds; this is near-real-time synchronization, not fully instant event-based Realtime.

## Important production work before use
- Add accounts, authentication, role-checked RLS and session-specific permissions; never use the pilot demo SQL for real employee data.
- Separate role identity from the synchronized session document (role in current prototype is a convenience UI toggle; not an authorization boundary).
- Add immutable event-level audit history, timestamped signature intent and attestation, legally appropriate e-sign consent, and tamper-evident finalized records.
- Implement conflict resolution for concurrent edits. The current whole-document update strategy is last-write-wins and may overwrite edits if both participants make changes simultaneously.
- Confirm the paper binder transcription, safety wording, policies, equipment-specific criteria, and any applicable regulatory standards with the company's safety/training lead before operational use.
- Support authenticated offline changes with conflict-safe queueing once the production backend is designed.

## What this is not
This is a functional user-interface prototype, not a secure or legally verified training record system. Local browser storage is not a durable employee records database. It should not be used to attest training completion or compliance until the production security and review work has been completed.

## Version 2.1 setup changes
- License class dropdown: Class A / Class B / Permit; separate license expiration date picker.
- Equipment type, pump size (20–65 meters), manufacturer dropdowns.
- Unit selector with Add unit button. Unit numbers are stored in this browser, not centrally; configure a shared fleet catalog before production.
- Existing local evaluation records are retained; printouts include new equipment details.

## Version 2.2 evaluation changes
- Native dropdowns for road test, yard controls, and jobsite evaluation rows.
- Final procedures and safety acknowledgment rows require BOTH Acknowledged and typed initials, entered in trainee mode.
- Final review includes a computed provisional score: passing Yes/Proficient items divided by all road, controls, and jobsite evaluation items (completed Proficient re-evaluations count as passing). This percentage is **not** an approved pass threshold.
- Review blocks finalization if required acknowledgments, initials, signatures, or evaluation items are missing, or if No/Needs Improvement ratings remain unresolved.
- Print / Save as PDF from the summary includes the score and trainee initials. Browser print dialog must be used to save the PDF.
- No real time backend, login security or audit trail is provided in this version.
