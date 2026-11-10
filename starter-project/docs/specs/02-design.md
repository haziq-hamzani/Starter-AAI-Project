# Login – Design

> **How it is built.** This document turns the requirements into a plan a
> builder (person or AI) can follow without asking the business again.
> [01-requirements.md](01-requirements.md) is the source of truth for
> *behaviour*; if the two disagree, fix the requirements first, then this file.
>
> - **Part A – Data design** (sections 1–2): **you own and review this.**
> - **Part B – App design** (sections 3–8): the AI drafts this; you check it.

---

# Part A – Data design (you review)

## 1. Tables

The app stores data in one table so far. Every row gets a number (`id`)
automatically. In Session 2 you add the leave request table here.

### `User` – one row per person who can log in

| Field | Type | Required? | What it holds |
|-------|------|-----------|---------------|
| `id` | number | auto | Unique number for the user |
| `email` | text | yes, unique | Used to log in, stored in lowercase, e.g. `employee-1@budiman.com` |
| `name` | text | yes | Full name shown on screen, with the division in brackets, e.g. "Employee 1 (Finance & Administration)" |
| `passwordHash` | text | yes | Scrambled password – never the real one |
| `role` | text | yes | `EMPLOYEE`, `SUPERVISOR`, `HR` or `ADMIN` |
| `identificationNo` | text | yes | Staff ID: `00-1` then 5 digits, e.g. `00-148291` |
| `designation` | text | yes | Job title, one of the list in rule 4, e.g. `Administrative Officer` |
| `division` | text | yes | Department, one of the list in rule 4, e.g. `Finance & Administration` |
| `createdAt` | date + time | auto | When the user was created |

> **Why is role plain text?** SQLite (our database) does not support a fixed
> list of choices ("enums"). So the allowed values are listed in
> `src/lib/constants.ts`, and the app checks them.

## 2. Business rules

1. **Emails.** Every email is a company address ending in `@budiman.com`. It is
   stored in lowercase and is unique, so login ignores capitals.
2. **Passwords.** Stored only as a hash (bcrypt). A wrong email and a wrong
   password give the **same** message, so nobody can find out which emails exist.
3. **Roles.** Each user has exactly one role. The role decides which pages they
   can open (§4).
4. **Divisions and designations** are chosen from fixed lists, kept in
   `src/lib/constants.ts`:
   - **Division:** Software Development, Quality Assurance, IT Infrastructure &
     Operations, Cybersecurity, Data & Analytics, Product & Design, Customer
     Support, Human Resources, Finance & Administration, Management.
   - **Designation:** Software Engineer, QA Engineer, System Administrator,
     IT Support Technician, Data Analyst, UI/UX Designer, Team Lead, HR Officer,
     Administrative Officer, Director. The designation is only a job title;
     the role decides what someone can do.
5. **Sessions.** A login lasts 7 days on the same browser, then the user must
   log in again.

---

# Part B – App design (AI drafts, you check)

## 3. Technology

Chosen so everyone can run the app on their own laptop with no extra accounts.

| Part | Choice | In plain words |
|------|--------|----------------|
| Web framework | **Next.js** (App Router) + **TypeScript** | Builds both the screens and the server logic in one project. |
| Database | **SQLite** via **Prisma 6** | The database is a single file; Prisma lets the code read and write it. |
| Login | Signed **session cookie** | After login the browser keeps a signed note saying who you are (7 days). JavaScript on the page can't read it (`httpOnly`). |
| Passwords | **bcryptjs** | Stores a scrambled version of each password. |
| Styling | **Tailwind CSS v4** | Styles are written as class names, e.g. `text-sm`. No config file needed. |
| Buttons and forms | **Server actions** | Pressing a button calls a function on the server, which checks the user before doing anything. There are no separate `/api` routes. |

## 4. Screens (routes)

After login, each user is sent to their role's home page. Opening a page for
another role sends you back to your own home page.

| Route | Who | What it shows |
|-------|-----|---------------|
| `/` | anyone | Sends you to your home page, or to `/login`. |
| `/login` | anyone | Email and password form. A logged-in user is sent to their home page. |
| `/employee` | Employee | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |
| `/supervisor` | Supervisor | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |
| `/hr` | HR | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |
| `/admin` | Admin | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |

A page that doesn't exist shows **Not found**, with a button back to your
home page.

**Layout:** no sidebar. A top bar across every page shows the app name
(Budiman HRMS) on the left. On the right it shows who is logged in (name and
role) and a **Log out** button. On a phone it shows the user's initials
instead of the name and role. The page below the top bar is empty apart from a short hint line.

**Feedback:** **Log out** opens a confirmation box first. Errors are shown in
red above the form.

## 5. Where the code lives

| Folder / file | What's in it |
|---------------|--------------|
| `prisma/schema.prisma` | The tables from Part A. |
| `prisma/migrations/` | Steps that create the tables. |
| `prisma/seed.ts` | The demo users (§6). |
| `src/lib/constants.ts` | Roles, divisions, designations, time zone, date helpers. |
| `src/lib/auth.ts` | Login cookie, `getCurrentUser()`, `requireRole()`, `roleHome()`. |
| `src/lib/db.ts` | The single database connection. |
| `src/app/actions.ts` | Server actions: `loginAction`, `logoutAction`. |
| `src/app/login/` | The login page. |
| `src/app/<role>/` | One folder per role's screens (pages with only a hint for now). |
| `src/components/` | Screen pieces: app frame (top bar), login form and log out button. |
| `src/components/ui/` | Small building blocks used everywhere: Button, Card, PageHeader, EmptyState, Avatar, Icons, form field styles, dialogs, and the overlay page scrollbar. Use these when you build the module. |
| `src/app/globals.css` | Brand colours (the `brand-…` and `navy` colours). Change them here. |

## 6. Demo data

`npm run db:seed` deletes all users and creates:

| Email (`@budiman.com`) | Name | Role | ID no. | Designation | Division |
|----------|------|------|--------|-------------|----------|
| `employee-1@` | Employee 1 (Finance & Administration) | Employee | 00-148291 | Administrative Officer | Finance & Administration |
| `employee-2@` | Employee 2 (Software Development) | Employee | 00-173604 | Software Engineer | Software Development |
| `supervisor-1@` | Supervisor 1 (Finance & Administration) | Supervisor | 00-159827 | Team Lead | Finance & Administration |
| `supervisor-2@` | Supervisor 2 (Software Development) | Supervisor | 00-126418 | Team Lead | Software Development |
| `hr-1@` | HR 1 (Human Resources) | HR | 00-191356 | HR Officer | Human Resources |
| `hr-2@` | HR 2 (Human Resources) | HR | 00-184072 | HR Officer | Human Resources |
| `admin@` | Admin (Management) | Admin | 00-105739 | Director | Management |

All use the password `password123` (workshop only). There are two users for
each of Employee, Supervisor and HR, in different divisions, so the module you
build can be tested with more than one person per role.

## 7. Security (in plain words)

- Every page checks who is logged in and their role (`requireRole`).
- Every server action checks again on the server – hiding a button is not enough.
- The login cookie is signed with `AUTH_SECRET` from `.env`, so it can't be faked.
- Passwords are stored only as a hash.

## 8. Commands

| Command | What it does |
|---------|--------------|
| `npm run dev` | Start the app at http://localhost:3000 |
| `npm run db:migrate` | Create or update the database tables |
| `npm run db:seed` | Reset the demo users |
| `npm run db:studio` | Open a browser view of the database |
| `npm run lint` | Check the code for common mistakes |
| `npm run build` | Check the whole app compiles |

---

# Leave Request – Design

> **How it is built.** This document turns the Leave Request requirements
> ([01-requirements.md](01-requirements.md) §1–6) into a plan a builder
> (person or AI) can follow without asking the business again. If the two
> disagree, fix the requirements first, then this file.
>
> - **Part A – Data design** (sections 1–4): **you own and reviewed this** —
>   approved as drafted.
> - **Part B – App design** (sections 5–9): the AI drafts this; **you check it.**

# Part A – Data design (you review)

## 1. Tables

One new table. Every row is **one leave request by one employee**. Like the
`User` table, every row gets a number (`id`) automatically, and the fixed
lists are kept in `src/lib/constants.ts` because SQLite has no enums.

### `LeaveRequest` – one row per leave request

| Field | Type | Required? | What it holds |
|-------|------|-----------|---------------|
| `id` | number | auto | Unique number for the request |
| `userId` | number | yes | Which employee made the request (the `id` of their row in `User`) |
| `leaveType` | text | yes | `ANNUAL`, `SICK` or `CASUAL` — one of the three agreed types |
| `startDate` | date | yes | First day of the leave |
| `endDate` | date | yes | Last day of the leave |
| `days` | number | auto | Working days (Mon–Fri) between the two dates, counted by the app when the form is submitted — the employee never types it |
| `supervisorId` | number | yes | Which supervisor must decide: chosen at submission, must be a Supervisor in the employee's own division. Never changes after that |
| `status` | text | yes | Where the request is in the workflow — one of the six values in §2 |
| `supervisorRemark` | text | no | The supervisor's reason, filled in only when the supervisor rejects |
| `hrRemark` | text | no | HR's reason, filled in only when HR rejects |
| `hrOfficerId` | number | no | Which HR officer made the final decision — the `id` of their row in `User`. Empty until HR acts; their name is shown on the request |
| `createdAt` | date + time | auto | When the request was submitted |

> **Why two remark fields instead of one?** A rejection ends the request
> (rule 7), so at most one of them is ever filled — and the app always knows
> which stage said no without an extra "who wrote this" field. There is no
> field for balances or approvals history: nothing is stored "in case".

## 2. Statuses

The status says where the request is. It is stored as text; the six allowed
values go in `src/lib/constants.ts`. Users only ever see the plain words.

| Stored as | Shown as | Meaning | What can happen next |
|-----------|----------|---------|----------------------|
| `WITH_SUPERVISOR` | **With Supervisor** | Submitted, waiting for the chosen supervisor | Supervisor signs → **Signed**; supervisor rejects (remark required) → **Rejected**; employee withdraws → **Withdrawn** |
| `SIGNED` | **Signed – waiting to send to HR** | The supervisor has signed; only the employee can move it on | Employee sends it to HR → **At HR** |
| `AT_HR` | **At HR** | In the shared HR queue; either HR officer may take it | HR approves → **Approved**; HR rejects (remark required) → **Rejected** |
| `APPROVED` | **Approved** | Finished: leave granted | Nothing (final) |
| `REJECTED` | **Rejected** | Finished: refused, with a remark from the stage that said no | Nothing (final) |
| `WITHDRAWN` | **Withdrawn** | Finished: the employee withdrew it before the sign | Nothing (final) |

**Allowed moves — nothing else is legal:**

```
(new) ──submit──▶ With Supervisor ──sign──▶ Signed ──send to HR──▶ At HR ──approve──▶ Approved
                     │                         │                      │
                     │ reject (remark)         │ (employee only)      │ reject (remark)
                     ▼                         ▼                      ▼
                 Rejected                                            Rejected
                     │
                     └── employee withdraws (before sign only) ──▶ Withdrawn
```

In words: submit → With Supervisor; sign → Signed; reject → Rejected; send
to HR → At HR; approve → Approved; withdraw → Withdrawn. **Approved, Rejected
and Withdrawn are final** — a mistake means a brand-new request.

## 3. Business rules

1. **Leave types.** Only `ANNUAL`, `SICK` and `CASUAL`. The list lives in
   `src/lib/constants.ts`, and the app checks any value against it.
2. **Dates.** Both are required. The end date must not be before the start
   date. Past dates are allowed (retroactive sick leave). There is no maximum
   length.
3. **Days.** Counted by the app, never entered: the number of Mondays–Fridays
   from start to end inclusive. A request that covers only a weekend shows
   **0 days** and is still allowed.
4. **The supervisor.** Chosen at submission from the employees' own division
   (a user whose role is SUPERVISOR and whose division matches; the
   designation plays no part). It never changes, and **only that supervisor**
   sees the request in their needs-your-action table and may sign or reject
   it.
5. **No double-booked dates.** A request cannot overlap the dates of one of
   the same employee's **live** requests. "Live" means anything not yet
   Rejected or Withdrawn. A submission that overlaps is refused with a clear
   message and nothing is saved.
6. **Who may do what.** Employees only create requests — for themselves —
   withdraw (rule 8) and send to HR. Supervisors only sign or reject at the
   **With Supervisor** stage. HR officers only approve or reject at the **At
   HR** stage. Admins change nothing, ever. Nobody can create a request on
   someone else's behalf, and no stage can be skipped: the server checks the
   actor's role *and* the request's current status before any change.
7. **Rejection ends the request and needs a remark.** The supervisor's or
   HR's remark is required to reject, and is shown to the employee and the
   admin. Signing and approving carry no remark.
8. **Withdrawal.** The employee may withdraw only while the request is
   **With Supervisor**. After that the only ways forward are the supervisor
   rejecting or HR deciding.
9. **First HR officer to act wins.** The officer who approves or rejects
   first is stored in `hrOfficerId` and shown as the deciding officer. The
   second officer's attempt is refused because the request is no longer
   **At HR** (rule 6 covers this: the status check is the protection).
10. **Status values are checked in code.** Like roles today, the six allowed
    status strings and the three leave types live in `src/lib/constants.ts`
    and every write goes through a check.

## 4. Who sees what

| Role | Sees | Can change |
|------|------|------------|
| **Employee** | Only their own requests (current table + history). Each one shows its status, days, dates, and any remark with who wrote it, plus the deciding HR officer's name once HR has acted. | Submits new requests for themselves; withdraws while **With Supervisor**; sends to HR when **Signed**. Nothing else. |
| **Supervisor** | Only requests where they are the chosen `supervisorId`: **With Supervisor** ones in the needs-your-action table, the rest in their history. | Signs or rejects the requests in their needs-your-action table. Nothing else. |
| **HR** | The whole system: every request **At HR** in the shared needs-your-action queue (same list for both officers), and every finished request in history. | Approves or rejects a request in the queue (first to act wins). Never edits fields, never signs on a supervisor's behalf. |
| **Admin** | Every request from every staff member, view only, with a **Staff** column. | Nothing. |

Two visibility rules to note:

- A supervisor never sees requests belonging to another supervisor (AC-13),
  and an employee never sees anyone else's requests at all.
- The employee sees remarks **after** they are made — the form itself has no
  reason field (requirements §3, U6).

> **Demo data:** `npm run db:seed` will also create sample requests that
> cover all six statuses, so each screen can be demoed straight after
> seeding. The exact sample rows are listed in Part B, §7.

# Part B – App design (AI drafts, you check)

## 5. Screens — one screen per action

The app keeps the same frame: top bar with the app name, who is logged in,
and **Log out**. Everything reuses the `src/components/ui/` building blocks
(PageHeader, Card, Button, Field, ConfirmDialog, SuccessModal, EmptyState,
DetailSection). Errors show in red above the form, like login.

Each role action gets its own screen. Confirmations and success messages are
part of every action, not separate screens.

| Route | Who | Action it serves | What it shows |
|-------|-----|------------------|---------------|
| `/employee` | Employee | Track; withdraw; send to HR | An **Apply for leave** button (to the form below). Current table: my live requests — columns Type, From–To, Days, Status, Supervisor; **Withdraw** shows while *With Supervisor*, **Send to HR** while *Signed*. History table below: my Approved / Rejected / Withdrawn. Rejected rows show the remark. Friendly empty text when a table has nothing. |
| `/employee/leave/new` | Employee | Apply for leave | Form: leave type (three choices), start date, end date, supervisor (only Supervisors in my division; a note if there are none). A "Working days: N" count appears as the dates are picked. Errors in red (end before start; overlapping dates). **Submit** → confirm box → success message → back to `/employee`. **Cancel** → back without saving. |
| `/supervisor` | Supervisor | See the queue; sign; open a rejection | Needs-your-action table: my assigned requests that are *With Supervisor* — columns Who, Type, From–To, Days. Row buttons: **Sign** (confirm → success) and **Reject** (goes to the screen below). History table below. |
| `/supervisor/leave/[id]/reject` | Supervisor | Reject with a remark | A summary of the request, then a remark box (**required**). **Confirm rejection** → success → back to `/supervisor`. Cancel → back, nothing changed. |
| `/hr` | HR | Shared queue; approve; open a rejection | Needs-your-action table: every request *At HR* — the same list for both officers — columns Who, Division, Type, From–To, Days. Row buttons **Approve** (confirm → success) and **Reject** (screen below). History table below: every finished request. |
| `/hr/leave/[id]/reject` | HR | Reject with a remark | Same shape as the supervisor's reject screen. |
| `/admin` | Admin | See every record | One table of every request ever made — columns Staff, Division, Type, From–To, Days, Status, HR officer. **No action buttons anywhere.** A row links to the read-only detail. |
| `/employee/leave/[id]` | Employee | Read one of my requests in full | All fields, status, remark with who wrote it, deciding HR officer's name once decided. Buttons only for my own allowed actions (Withdraw / Send to HR). |
| `/supervisor/leave/[id]` | Supervisor | Read an assigned request in full | Same layout; **Sign** and **Reject** while it is *With Supervisor*. |
| `/hr/leave/[id]` | HR | Read any request in full | Same layout; **Approve** and **Reject** while it is *At HR*. |
| `/admin/leave/[id]` | Admin | Read any request in full | Read-only. It never shows a button. |

Rules that apply to all of them:

- Every page starts with `requireRole(...)`, like the login pages: the wrong
  role is sent to their own home page.
- Detail screens belong to the viewer: an employee opening another employee's
  id, or a supervisor another supervisor's, gets **Not found** — same as any
  unknown page.
- Buttons appear only when Part A's moves allow them (who + status). Hiding
  is for tidiness; the server re-checks (Part B §8).

## 6. Server actions and where the code lives

Like login, everything runs through server actions — no `/api` routes. Seven
new actions go in `src/app/actions.ts`, and **every one** re-checks the
signer's role, the request's row ownership and its current status before
writing (using `canAct` below), then refreshes the pages it changed.

| Action | Who may call it | Allowed from status |
|--------|-----------------|---------------------|
| `createLeaveRequestAction` | Employee (for themselves) | (new) |
| `withdrawAction` | The employee who owns it | *With Supervisor* |
| `sendToHrAction` | The employee who owns it | *Signed* |
| `signAction` | The chosen supervisor | *With Supervisor* |
| `supervisorRejectAction` | The chosen supervisor | *With Supervisor* |
| `approveAction` | Either HR officer | *At HR* |
| `hrRejectAction` | Either HR officer | *At HR* |

| Folder / file | What's in it |
|---------------|--------------|
| `prisma/schema.prisma` | The `LeaveRequest` model from Part A §1. |
| `prisma/migrations/…` | A new migration creating the table. |
| `prisma/seed.ts` | Demo users as before, plus the sample requests in §7. |
| `src/lib/constants.ts` | Adds `LEAVE_TYPES` and `LEAVE_STATUSES` with guard functions, beside the existing role/division lists. |
| `src/lib/leave.ts` **(new)** | Small plain helpers shared by pages and actions: `workingDays(start, end)`, `hasOverlap(...)`, `canAct(status, who, action)`. |
| `src/app/actions.ts` | The seven actions above, next to `loginAction` / `logoutAction`. |
| `src/app/employee/` | Home page (tables + row buttons) plus `leave/new/` and `leave/[id]/`. |
| `src/app/supervisor/`, `src/app/hr/` | Home pages plus `leave/[id]/` and `leave/[id]/reject/`. |
| `src/app/admin/` | Home table plus `leave/[id]/`. |
| `src/components/leave/` **(new)** | `LeaveTable` (current/history), `LeaveForm`, `RejectForm`, `RequestDetail`, and the row buttons wired to ConfirmDialog + SuccessModal. |
| `src/components/ui/` | Reused everywhere — nothing new added here unless a block is truly missing. |

## 7. Demo data

`npm run db:seed` keeps the seven demo users (unchanged) and adds these
sample requests, so every status, both rejection stages and the deciding
officer can be demoed straight away. Dates are relative to the day you seed.

| Staff | Type | Dates (relative) | Supervisor | Status | Extras |
|-------|------|------------------|------------|--------|--------|
| employee-1 | Casual | next Mon–Tue | supervisor-1 | With Supervisor | – |
| employee-1 | Annual | next week Mon–Fri | supervisor-1 | Signed | – |
| employee-1 | Sick | next Fri | supervisor-1 | At HR | – |
| employee-2 | Sick | next Mon | supervisor-2 | At HR | second queue item, for the first-to-act demo |
| employee-2 | Annual | last Mon–Wed | supervisor-2 | Approved | deciding officer **HR 1** |
| employee-2 | Casual | last Fri | supervisor-2 | Rejected | remark by supervisor-2: "Team handover not arranged." |
| employee-1 | Sick | last Thu | supervisor-1 | Rejected | remark by **HR 2**: "Needs a doctor's note." |
| employee-2 | Casual | last Mon | supervisor-2 | Withdrawn | – |

(The finished rows may share dates with live ones — the overlap rule applies
only when an employee submits, never to seed.)

## 8. Security (in plain words)

- Every leave page calls `requireRole`, and detail screens check the row is
  the viewer's to see (own request / assigned request) — otherwise Not found.
- Every action re-checks on the server: role, row ownership, and **current
  status** via `canAct`. This is also the first-to-act protection: the second
  HR officer fails the status check.
- Dates, working-day count and overlap are enforced server-side, not only in
  the browser — the form's nice errors are for people, not for safety.
- Confirmations and success messages are usability, not security: an action
  still re-validates even if someone skips them.

## 9. Commands

No new commands. `npm run db:migrate` now also creates the leave table, and
`npm run db:seed` resets users **and** sample requests. Everything else is
the table from the login design §8.
