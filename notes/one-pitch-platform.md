---
title: "One pitch Platform"
cover: https://images.unsplash.com/photo-1486854561807-38fc02359493?auto=format&fit=crop&w=2400&q=82
favicon: 📝
---
# One Pitch – Change Logs

## Overview
- One platform. Every VC. Every startup. An Enlighten Capital initiative.  
- Next.js 14 (App Router) + Supabase (auth, Postgres, RLS) + Tailwind. Deploys to Vercel.  

## Key discussion points
- What v1 does  
- Role Can do  
- Startup Sign up, submit/edit one application, see status and pitch‑day slot  
- VC (partner) After admin approval: see only startups matched to them, record interest, score and notes  
- Investor / LP After admin approval: see Enlighten‑invested / syndicating deals, indicate ticket size  
- Admin (Enlighten) Approve VCs/investors, shortlist, match startups to VCs, create monthly pitch days, assign slots, view all VC feedback and investor interest, mark Enlighten investment  
- Status flow: submitted → shortlisted → matched → pitched → invested → syndicating (or rejected).  
- Access is enforced in the database with Row Level Security, not just in the UI.  

### Roles and admin access
- Role Access  
- Super admin Everything an admin can do, plus add admins, reset their temporary password, and disable or enable them (Admins page). Created once with the SQL in setup step 5.  
- Admin View, edit and approve every record: all users (edit details, approve or revoke), all startups (edit every field, status, matching, pitch days, data room, documents), all VC feedback and investor interest (edit or delete). Cannot create or change other admins.  
- VC, investor, startup As described below.  
- How the super admin adds an admin: Admins page, enter name and email, Create admin. The account is created with a one‑time temporary password shown on screen; share it securely. The new admin must choose their own password at first sign‑in. If the email already has an account, it is promoted instead and keeps its password.  
- Safeguards enforced in the database: only a super admin can change roles or disable accounts; nobody can change their own role or disable themselves; admins cannot edit other admin accounts; a disabled account loses all access immediately.  

### Startup onboarding
- The application form collects:  
- Company and contact: startup name, founder name, email id, contact number, website  
- Funding details: stage, sector, ticket size, previous funding, valuation, revenue for the last financial year and current annual run‑rate (all in ₹ crore)  
- Uploads: updated pitch deck, due diligence report, valuation report, audited financial statements (last 2 years, multiple files), MIS data (6 months)  
- All five documents are required to submit, and every number field is required (enter 0 for none).  
- Add documents step: once Enlighten starts reviewing, the company and funding details lock, but the founder can still add or replace documents from the same page; admin is emailed each time. The database enforces this: a founder can never change status, Enlighten’s amount or the data‑room flag, and after review starts can change documents only.  

### Documents and the data room
- Files go straight from the browser to a private Supabase Storage bucket (decks, 15 MB per file; PDF, PPT/PPTX, Word, Excel, CSV by field). Downloads use 1‑hour signed links.  
- Founder: reads their own files. Admin: reads everything.  
- Approved VC matched to the startup: reads the pitch deck always, and the other four documents only when admin has opened the data room for that startup.  
- Approved investor: same rule, for startups that are invested or syndicating.  
- Founders cannot open their own data room, and the database rejects any file path outside the founder’s own folder.  

### Email notifications
- Event → Goes to  
- New startup application → Admin team, and a receipt to the founder  
- New VC / investor signup → Admin team  
- Account approved → The VC / investor  
- Status changed (shortlisted, matched, pitched, invested, syndicating, not taken forward) → The founder  
- Pitch‑day slot assigned → The founder, with date, time and join link  
- Newly matched to a startup → The VC  
- Email errors never block the action that triggered them.  

### Pitch days, calendar invites and reminders
- Slots: admin picks a pitch day (date, meeting link, minutes per slot) and gives each startup a start time. Times are Indian Standard Time.  
- Calendar invites: assigning a slot emails the founder and every matched VC a .ics invite (opens in Google, Outlook and Apple Calendar). The same event updates if the time changes. Founders and VCs also get “Add to calendar” and Google Calendar links in their portal.  
- Reminders: a daily job (/api/cron/reminders, scheduled in vercel.json for 9:00 AM IST) emails founders and VCs the day before and again on the morning of the pitch day. Founders get their slot with the invite; each VC gets one digest of their matched startups. A log table stops duplicates. Admin can also press “Send reminders now” for any pitch day.  
- Setup: add CRON_SECRET (any long random string) in Vercel. Vercel sends it automatically with each scheduled call; without it the endpoint refuses every request. Scheduled jobs run on Vercel’s production deployment.  

### Google Meet for every pitch slot
- Instead of pasting a meeting link, connect the shared Enlighten Google account once and 1Pitch creates a Google Meet for each startup slot.  

- How it works  
- Assigning a slot creates a calendar event in that account with a new Meet room. The link goes into the invite, reminders and portals.  
- The event’s guest list is the founder and every matched, approved VC. Changing the time or the matches updates it. Google sends no emails of its own; 1Pitch sends the invites.  
- Meet rooms belong to the connected account, so keep that account active.  
- If Google is not connected or a call fails, the slot is still saved and invited, using the pitch day’s pasted link if there is one. The admin console shows a notice.  
- Only a super admin can connect or disconnect the account. The refresh token is stored in a table that only the server key can read.  

### Scheduling a call with a startup (admin)
- Every startup card in the admin console has a Schedule a call button, separate from pitch days. Pick a date, time (IST) and length, tick any matched VCs to include, add other guests by email if needed, and add an agenda. The founder is always invited.  
- A Google Meet is created for the call (or paste your own link if Google is not connected), and everyone gets an email with a calendar invite.  
- The founder sees the call on their application page, and invited VCs see it at the top of their deal flow, each with Join, Add to calendar and Google Calendar links.  
- Cancel call removes the Google event and emails a cancellation to everyone invited.  
- To change a time, cancel and schedule again.  

### Team calendar and overlap checks (admin)
- Admin, Calendar shows a week view (Monday to Sunday, IST) of everything booked: pitch slots (amber), calls (blue) and, if Google is connected, other events on the shared Enlighten Google calendar (grey, private events show as “Busy”). Anything that overlaps another booking has a red border, and an overlap list sits above the grid. Click a slot or call to jump to that startup in the console. Previous and next week buttons move around.  
- When you schedule a call, edit a call, or assign a pitch slot, 1Pitch checks the same calendar first. If the time overlaps something, nothing is saved and you are told what it clashes with. Tick Allow overlap on the form to save it anyway. Slots without a readable time (older free‑text entries) cannot be placed on the calendar.  

### Deck replaced
- When a founder saves a different pitch deck, every approved VC matched to that startup gets an email, and the deal shows a “New deck” badge for two weeks.  Because founders are not allowed to see who is matched to them.  

Admin, Syndication. For each invested or syndicating deal:

Terms: round size, success fee (% of capital raised), management fee (% a year) and carry (% of profit). Approved VCs and investors who can see the deal can read the terms.
Commitments: who invests alongside Enlighten (platform users or outside parties), with amount and stage: interested, committed, funded. Investors' interest from the portal can be added with one click.
Fee share: rules like "Partner X gets 20% of carry". Partner shares can never exceed 100% for any fee type; the rest stays with Enlighten.
Ledger: every success fee, management fee and carry entry, with basis, rate, amount, due date and status (accrued, invoiced, received, waived). Amount is calculated from basis × rate if left blank. Each row shows the Enlighten and partner split, and a summary shows what each party is owed and has received.
CSV export of the ledger for accounts.
VCs and investors see their own commitments and the fees charged to them in their portal. Fee-share rules are admin-only.

Public directory
Startups (/startups): a founder requests a listing from their application page and an admin approves it. The page shows only name, sector, stage, description, website and an "Enlighten backed" badge.
VC profiles (/vcs): a VC fills in "Public profile" (about, location, focus sectors, stages, cheque size) and ticks "Show in the public directory". Email, feedback and deal flow are never shown.
The database exposes these pages through four read-only functions that return only those columns, so the public key can never read financials, contacts or documents.
Not in v1 (suggested next)
Editing or deleting a pitch day from the admin console (today you add a day and assign slots)
Syndication workflow with automatic fee calculation on a schedule (the ledger is manual entry today)
Logos and photos on public profiles  
-