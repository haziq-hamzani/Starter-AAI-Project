# Login – Tasks

> **In what order, and how to check.** Build one task at a time, in order.
> After each task: the AI builds it → a reviewer agent tests it → **you check it
> in the browser**.
>
> If a check fails, **fix the spec first** (requirements or design), then ask
> the AI to rebuild. The requirements and design are the source of truth.
>
> These tasks are **already done** in the starter app – they show what a task
> list looks like. In Session 2, add the Leave Request tasks after Task 6.

**Core and Stretch.** **Core** tasks must be finished first: the app is not
usable without them. **Stretch** tasks are nice extras. Do them only if time is
left, and only if you ask for them. All the login tasks are Core, because login
has to work before anything else.

**Before you start:** `npm install`, then `cp .env.example .env`
(Windows: `copy .env.example .env`).

| # | Task | Core / Stretch | Checks (from requirements §4) |
|---|------|----------------|-------------------------------|
| 1 | Set up the app | Core | – |
| 2 | Database table and demo users | Core | – |
| 3 | Log in and log out | Core | AC-01, AC-02, AC-03, AC-06 |
| 4 | Role protection and home pages | Core | AC-04, AC-05, AC-07 |
| 5 | Top bar | Core | AC-08, AC-09 |
| 6 | Final check and README | Core | All |

---

## Task 1 – Set up the app *(Core)*

- **Goal:** A Next.js + TypeScript app with Tailwind CSS that starts.
- **Files:** `package.json`, `next.config.ts`, `tsconfig.json`,
  `src/app/layout.tsx`, `src/app/page.tsx`, `src/app/globals.css`
  (starts with `@import "tailwindcss";` – no Tailwind config file needed).
- **Check:** `npm run dev` starts, and http://localhost:3000 shows a page.

## Task 2 – Database table and demo users *(Core)*

- **Goal:** Prisma with SQLite, the `User` table from design Part A, and the
  demo users from design §6 with hashed passwords.
- **Files:** `prisma/schema.prisma`, `prisma/migrations/`, `prisma/seed.ts`,
  `src/lib/db.ts`, `src/lib/constants.ts`, `.env.example`, the `db:seed`
  script in `package.json`.
- **Check:** `npm run db:migrate` creates `prisma/dev.db`. `npm run db:seed`
  prints the 7 demo users, and `npm run db:studio` shows them.

## Task 3 – Log in and log out *(Core)*

- **Goal:** Login page, signed session cookie, and logout with a confirmation
  box. After login, go to the role's home page.
- **Files:** `src/lib/auth.ts` (`getCurrentUser`, `startSession`, `endSession`,
  `roleHome`), `src/app/actions.ts` (`loginAction`, `logoutAction`),
  `src/app/login/page.tsx`, `src/components/LoginForm.tsx`,
  `src/components/LogoutButton.tsx`, `src/components/ui/` (Button, Field,
  dialogs).
- **Check:** AC-01, AC-02, AC-03, AC-06.

## Task 4 – Role protection and home pages *(Core)*

- **Goal:** A `requireRole(...)` helper used at the top of every page, so each
  role sees only its own screens, and a home page for each role with only a short hint line.
- **Files:** `src/lib/auth.ts`, `src/app/employee/page.tsx`,
  `src/app/supervisor/page.tsx`, `src/app/hr/page.tsx`,
  `src/app/admin/page.tsx`.
- **Check:** AC-04, AC-05, AC-07.

## Task 5 – Top bar *(Core)*

- **Goal:** The top bar from design §4 (app name, who is logged in, **Log
  out**), and the **Not found** page.
- **Files:** `src/components/AppShell.tsx`, `src/components/LogoutButton.tsx`,
  `src/components/ui/`, `src/app/not-found.tsx`.
- **Check:** AC-08, AC-09.

## Task 6 – Final check and README *(Core)*

- **Goal:** Run `npm run db:seed`, then walk through the whole acceptance
  checklist as each role. Make sure `README.md` explains how to install, run
  and test.
- **Check:** Every AC box ticked; `npm run lint` and `npm run build` pass.

---

# Session 2 – Leave Request Tasks

> Added in Session 2, continuing the numbering after Task 6. The loop is the
> same: build one task → run its checks → you try it in the browser →
> confirm before the next task. If a check fails, **fix the spec first**.
>
> **All ten tasks are Core** — the requirements have no Stretch stories, so
> nothing here is optional. Checks with no AC id are setup steps proven by the
> tasks after them.

| # | Task | Core / Stretch | Checks (from requirements §4) |
|---|------|----------------|-------------------------------|
| 7 | Leave request table and sample requests | Core | – |
| 8 | Apply for leave | Core | AC-10, AC-11, AC-12, AC-23, AC-24 |
| 9 | Employee home: track, withdraw, send to HR | Core | AC-10, AC-16, AC-17, AC-18, AC-19, AC-25 |
| 10 | Supervisor queue: sign and reject | Core | AC-13, AC-14, AC-15 |
| 11 | HR queue: approve and reject (shared, first to act) | Core | AC-20, AC-21, AC-22 |
| 12 | Admin: see every record | Core | all rows listed, no buttons (rest of AC-26 in Task 13) |
| 13 | Request detail screens (all roles) | Core | AC-26; remarks/officer display of AC-15, AC-21, AC-22 |
| 14 | Server-side protection sweep | Core | AC-17, AC-19, AC-28 |
| 15 | Confirmations and success messages everywhere | Core | AC-25, AC-27 |
| 16 | Final check and README | Core | all (AC-01…AC-28) |

## Task 7 – Leave request table and sample requests *(Core)*

- **Goal:** The `LeaveRequest` table exactly as design Part A §1, and seed
  data that covers all six statuses, both rejection stages and the deciding
  HR officer.
- **Files:** `prisma/schema.prisma`, a new folder in `prisma/migrations/`,
  `prisma/seed.ts`, `src/lib/constants.ts` (`LEAVE_TYPES`, `LEAVE_STATUSES`
  with guard functions), `src/lib/leave.ts` (`workingDays`, used by the seed).
- **Check:** `npm run db:migrate` creates the table; `npm run db:seed` prints
  the sample requests; `npm run db:studio` shows the rows from design Part B
  §7, with the correct computed `days`.

## Task 8 – Apply for leave *(Core)*

- **Goal:** The employee's apply screen creates a live request: three leave
  types, own-division supervisor list, app-counted working days, and the
  server refusing empty fields, an end date before the start, and an overlap
  with one of the employee's live requests.
- **Files:** `src/app/employee/leave/new/page.tsx`,
  `src/components/leave/LeaveForm.tsx`, `src/app/actions.ts`
  (`createLeaveRequestAction`), `src/lib/leave.ts` (`canAct`, `hasOverlap`,
  `workingDays`), `src/lib/constants.ts`.
- **Check:** AC-10, AC-11, AC-12, AC-23, AC-24.

## Task 9 – Employee home: track, withdraw, send to HR *(Core)*

- **Goal:** `/employee` shows the current and history tables, and the two
  employee actions live where the status allows: **Withdraw** while *With
  Supervisor*, **Send to HR** while *Signed* — each with its confirmation and
  success message.
- **Files:** `src/app/employee/page.tsx`,
  `src/components/leave/LeaveTable.tsx`, `src/app/actions.ts`
  (`withdrawAction`, `sendToHrAction`).
- **Check:** AC-10, AC-16, AC-17, AC-18, AC-19, AC-25 (employee page).

## Task 10 – Supervisor queue: sign and reject *(Core)*

- **Goal:** The supervisor's home lists only their assigned *With Supervisor*
  requests. **Sign** works with a confirm; **Reject** goes to its own screen
  where a remark is required.
- **Files:** `src/app/supervisor/page.tsx`,
  `src/app/supervisor/leave/[id]/reject/page.tsx`,
  `src/components/leave/RejectForm.tsx`, `src/app/actions.ts`
  (`signAction`, `supervisorRejectAction`).
- **Check:** AC-13, AC-14, AC-15.

## Task 11 – HR queue: approve and reject *(Core)*

- **Goal:** Both HR officers see the same *At HR* list. **Approve** works with
  a confirm; **Reject** uses the shared reject screen with a required remark.
  The deciding officer is stored and shown, and the second officer's attempt
  is refused.
- **Files:** `src/app/hr/page.tsx`, `src/app/hr/leave/[id]/reject/page.tsx`,
  `src/app/actions.ts` (`approveAction`, `hrRejectAction` — reusing
  `RejectForm`).
- **Check:** AC-20, AC-21, AC-22.

## Task 12 – Admin: see every record *(Core)*

- **Goal:** The admin's home lists every request from every staff member —
  Staff, Division, Type, From–To, Days, Status, HR officer — with **no action
  buttons anywhere**.
- **Files:** `src/app/admin/page.tsx` (reusing `LeaveTable` or a sibling).
- **Check:** After `npm run db:seed`, every sample request appears once,
  staff from both divisions visible, and no button can change anything.

## Task 13 – Request detail screens *(Core)*

- **Goal:** One detail screen per role showing the full record: fields,
  status, remarks with who wrote them, and the deciding HR officer's name.
  Buttons only for that role's allowed moves; the admin's is read-only; a
  row that isn't yours to see gives **Not found**.
- **Files:** `src/components/leave/RequestDetail.tsx`,
  `src/app/employee/leave/[id]/page.tsx`,
  `src/app/supervisor/leave/[id]/page.tsx`,
  `src/app/hr/leave/[id]/page.tsx`, `src/app/admin/leave/[id]/page.tsx`.
- **Check:** AC-26 (open a record as admin: details only, no buttons);
  as `employee-2` the supervisor remark on the rejected Casual is visible
  with its author (AC-15 display); as `employee-1` the HR remark and **HR 2**
  appear (AC-22 display); the approved row shows **HR 1** (AC-21 display).

## Task 14 – Server-side protection sweep *(Core)*

- **Goal:** Every leave page and action enforces itself on the server:
  `requireRole` at the top of each page, row ownership on detail screens,
  and `canAct` (role + status) in every action — so hidden buttons are never
  the only protection.
- **Files:** audit pass over all leave pages, `src/app/actions.ts`,
  `src/lib/leave.ts`.
- **Check:** AC-17, AC-19 and AC-28 by trying the URLs directly while logged
  in as the wrong role or at the wrong status — refused, nothing changed.

## Task 15 – Confirmations and success messages everywhere *(Core)*

- **Goal:** Every state-changing action goes through a confirmation box
  (Cancel changes nothing) and ends with a success message; every table has
  friendly empty text.
- **Files:** the row buttons in `src/components/leave/`, wiring to the
  existing `ConfirmDialog` and `SuccessModal`.
- **Check:** AC-25 and AC-27 across all four home pages.

## Task 16 – Final check and README *(Core)*

- **Goal:** From a clean `npm run db:seed`, walk the whole acceptance
  checklist as each role — login checks **and** leave checks — and update
  `README.md` with the new module and demo data.
- **Check:** AC-01…AC-28 all ticked; `npm run lint` and `npm run build`
  pass; the login checks (AC-01…AC-09) still pass.
