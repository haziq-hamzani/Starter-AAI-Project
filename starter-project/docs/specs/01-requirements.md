# Login – Requirements

> **What and why.** This document says what the login must do and why. It is
> the source of truth for behaviour: when something needs to change, update this
> document **first**. New to specs? Read [README.md](README.md) first.

## 1. The problem

Budiman Company is building an HR Management System. Before anyone can use
it, the system has to know **who is using it** and **what they are allowed to
do**.

This first part of the system provides:

1. A **login** with each person's company email and password.
2. A **home page for each role**, so everyone lands in the right place.
3. **Protection**, so nobody can open another role's pages.

## 2. Who uses it (roles)

| Role | Home page | What they will do (built in Session 2) |
|------|-----------|-----------------------------------------|
| **Employee** | `/employee` | Apply for leave and track their requests. |
| **Supervisor** | `/supervisor` | Decide on requests sent to them. |
| **HR** | `/hr` | Decide on requests sent to HR. |
| **Admin** | `/admin` | See every leave record. |

Each user has exactly one role.

## 3. User stories

Format: *As a [role], I can [do something] so that [benefit].*

- **U1. Log in** *(Core)*. As any user, I can log in with my company email and
  password so that I see the screens for my role.
- **U2. Land on my home page** *(Core)*. As any user, after logging in I go
  straight to my role's home page, so I don't have to look for it.
- **U3. Stay logged in** *(Core)*. As any user, I stay logged in for up to 7
  days on the same browser, so I don't have to log in every time.
- **U4. Log out** *(Core)*. As any user, I can log out when I'm done, so the
  next person on this computer can't use my account.
- **U5. Only my own screens** *(Core)*. As any user, I can only open the pages
  for my own role, so nobody can act as someone else.

## 4. Acceptance checklist

The login is done when every box is ticked. Use the demo accounts (password
`password123`): `employee-1` and `employee-2` (Employees), `supervisor-1` and
`supervisor-2` (Supervisors), `hr-1` and `hr-2` (HR), `admin` (Admin). Each
logs in with that name followed by `@budiman.com`, e.g.
`employee-1@budiman.com`; the checks below use the short name.
Run `npm run db:seed` first to start from clean data.

| ✓ | ID | Check | How to test |
|---|----|-------|-------------|
| [ ] | AC-01 | A user can log in and lands on their role's home page. | Log in as `employee-1` → you are on `/employee`, a page with only a short hint line. Repeat for `supervisor-1` (`/supervisor`), `hr-1` (`/hr`), `admin` (`/admin`). |
| [ ] | AC-02 | A wrong password or an unknown email shows an error. | Log in as `employee-1` with password `wrong` → "Invalid email or password." Try `nobody@budiman.com` → the same message. |
| [ ] | AC-03 | Empty fields are not sent. | Click **Log in** with the boxes empty → the browser asks you to fill them in. |
| [ ] | AC-04 | A logged-out user is sent to the login page. | Log out, then type `/employee` in the address bar → you are on `/login`. |
| [ ] | AC-05 | A logged-in user who opens the login page goes to their home page. | Logged in as `hr-1`, type `/login` → you are on `/hr`. |
| [ ] | AC-06 | Logging out asks first, then returns to the login page. | Click **Log out** in the top bar → a confirmation box → confirm → you are on `/login`. |
| [ ] | AC-07 | A user cannot open another role's pages. | As `employee-1`, type `/hr` or `/admin` → you are sent back to `/employee`. As `admin`, type `/employee` → you are sent back to `/admin`. |
| [ ] | AC-08 | Every page shows who is logged in. | The top bar shows the user's name and role. On a phone, it shows the user's initials instead. |
| [ ] | AC-09 | An unknown page shows "Not found". | Type `/something-else` → "Not found", with a button back to your home page. |
## 5. Not included (for now)

These are **out of scope** for the login:

- Signing up, or resetting a forgotten password.
- Adding, editing or deleting user accounts (only the demo data creates users).
- The Leave Request module itself – that is what you build in Session 2.

---

# Leave Request – Requirements

> **What and why.** This document says what the Leave Request module must do
> and why. It is the source of truth for behaviour: when something needs to
> change, update this document **first**. It continues the numbering from the
> login spec above: user stories start at U6, acceptance checks at AC-10.

## 1. The problem

Budiman Company still processes leave on paper. The employee fills in a form
and hands it to their supervisor; the supervisor signs it; HR receives a copy
and enters it into a spreadsheet by hand. That causes three problems:

1. Forms get lost or delayed between the employee, the supervisor and HR.
2. Employees can't see where their request is: waiting for the supervisor,
   with HR, or decided.
3. HR has no single list of requests waiting for a decision.

**Goal:** one request, one system, visible from submission to decision. Every
person sees exactly the requests they need to act on, and nothing gets lost.

## 2. Who uses it (roles)

| Role | What they do in this module |
|------|------------------------------|
| **Employee** | Submits a leave request to a supervisor in their own division, tracks its status and remarks, can withdraw it before it is signed, and sends a signed request to the HR queue. |
| **Supervisor** | Sees the requests sent to them, and signs or rejects each one (a rejection needs a remark). |
| **HR** | Sees every request waiting for HR in one shared list, and approves or rejects it (a rejection needs a remark). The officer who decides is recorded. |
| **Admin** | Sees every leave record in the system. View only: changes nothing. |

A request moves through these steps, and the status changes after each one:
**Employee submits → Supervisor signs (or rejects) → Employee sends to HR →
HR approves (or rejects)**.

## 3. User stories

Format: *As a [role], I can [do something] so that [benefit].* All stories
here are **Core** — everything agreed in the interview is needed for the
module to work; there are no Stretch stories in this session.

- **U6. Apply for leave** *(Core)*. As an Employee, I can submit a leave
  request — one of Annual, Sick or Casual, with a start and end date, and a
  supervisor from my own division — so that my request enters the approval
  flow properly.
- **U7. Track my requests** *(Core)*. As an Employee, I can see my requests
  with their status, and any remark, on my home page so that I always know
  where my request is.
- **U8. Withdraw before it is signed** *(Core)*. As an Employee, I can
  withdraw one of my requests while it is still with the supervisor, so that
  I can put right a request I got wrong.
- **U9. Send a signed request to HR** *(Core)*. As an Employee, I can send a
  supervisor-signed request to the shared HR queue, so that HR can make the
  final decision.
- **U10. Sign or reject** *(Core)*. As a Supervisor, I can see the requests
  sent to me and sign one, or reject it with a remark, so that only checked
  requests go on to HR.
- **U11. HR shared queue** *(Core)*. As HR, I can see every request waiting
  for HR in one shared list, so that no request is missed regardless of which
  officer picks it up.
- **U12. Approve or reject** *(Core)*. As HR, I can approve a request, or
  reject it with a remark, and my name is recorded as the deciding officer,
  so that decisions are accountable.
- **U13. See every leave record** *(Core)*. As an Admin, I can see all
  requests from all staff in one place, so that there is one reference list,
  without being able to change anything.
- **U14. Confirm and get feedback** *(Core)*. As any user, every action
  (submit, withdraw, send, sign, reject, approve) asks me to confirm first
  and shows a success message after, so that I never act by accident and
  always know it worked.

## 4. Acceptance checklist

The module is done when every box is ticked. Run `npm run db:seed` first: it
creates the demo users and sample leave requests covering every status. Demo
accounts (password `password123`): `employee-1` (Finance & Administration,
supervisor `supervisor-1`) and `employee-2` (Software Development, supervisor
`supervisor-2`), `hr-1`, `hr-2` (Human Resources), and `admin`. The checks
use the short names.

| ✓ | ID | Check | How to test |
|---|----|-------|-------------|
| [ ] | AC-10 | An employee can submit a request and it lands with the chosen supervisor. | As `employee-1`, open `/employee` → **Apply for leave** → type `Casual`, start = next Monday, end = next Tuesday, supervisor `supervisor-1` → **Submit** → confirm → success message. The request shows in your current table with status **With Supervisor**. Log in as `supervisor-1` → it is in the needs-your-action table. |
| [ ] | AC-11 | Empty fields are not sent. | On the apply form, click **Submit** with everything empty → the browser asks you to fill in the fields. No request is created. |
| [ ] | AC-12 | An end date before the start date is rejected. | On the apply form, set end date earlier than start date → submit → an error appears above the form in red and no request is created. |
| [ ] | AC-13 | A supervisor sees only requests sent to them. | `employee-1` submits a request to `supervisor-1`. As `supervisor-1` it appears in the needs-your-action table; log in as `supervisor-2` → it does not appear anywhere. |
| [ ] | AC-14 | A supervisor signing moves the status. | As `supervisor-1`, **Sign** the request from `employee-1` → confirm → success message. The request moves to the supervisor's history with status **Signed – waiting to send to HR**. As `employee-1` it shows the same status. |
| [ ] | AC-15 | A supervisor rejection needs a remark and shows it. | As `supervisor-2`, open the request from `employee-2` → **Reject** → try with the remark box empty → it will not submit. Type a remark → confirm → success. Status **Rejected**; as `employee-2` the request shows in history with the remark. |
| [ ] | AC-16 | The employee sends a signed request to HR. | As `employee-1`, the signed request shows a **Send to HR** button → confirm → success message. Status becomes **At HR** and the request leaves your current table. |
| [ ] | AC-17 | A request cannot skip the supervisor. | `employee-2` submits a new request (status **With Supervisor**) → no **Send to HR** button is shown. Trying to open the send action's URL directly → the action is refused and the status is unchanged. |
| [ ] | AC-18 | An employee can withdraw while it is with the supervisor. | `employee-2` opens the request still **With Supervisor** → **Withdraw** → confirm → success message. It moves to history with status **Withdrawn**, and `supervisor-2` can no longer act on it. |
| [ ] | AC-19 | A request cannot be withdrawn after it is signed. | As `employee-1`, the **Signed – waiting to send to HR** request shows no **Withdraw** button. Trying to withdraw it another way → refused, status unchanged. |
| [ ] | AC-20 | Both HR officers see the shared queue. | `employee-1`'s request is **At HR**. Log in as `hr-1` in one window and `hr-2` in another → both see it in their needs-your-action table. |
| [ ] | AC-21 | The first HR officer to decide wins and is recorded. | As `hr-1`, **Approve** the request → confirm → success message. The request detail records **HR 1** as the deciding officer. In `hr-2`'s window the request leaves the needs-your-action table and can no longer be actioned. |
| [ ] | AC-22 | An HR rejection needs a remark and is recorded. | As `hr-2`, open a request **At HR** → **Reject** → empty remark will not submit → type a remark → confirm → success. Status **Rejected** with the remark and `HR 2` as deciding officer, visible to the employee and to `admin`. |
| [ ] | AC-23 | Days are counted as working days (Mon–Fri). | As `employee-1`, submit a request Friday → the following Monday → the Days column shows **1**. Submit another Monday → Friday → Days shows **5**. |
| [ ] | AC-24 | A new request cannot overlap one still live. | While `employee-1` has a live request (not rejected/withdrawn), submit a second request covering an overlapping date → red error message, no record created. |
| [ ] | AC-25 | Each home page has a current and a history table. | Open `/employee`, `/supervisor`, `/hr` and `/admin` → each shows a needs-your-action/current table on top and a history table below. Approved, rejected and withdrawn requests appear only in history. |
| [ ] | AC-26 | The admin sees every record and changes nothing. | Log in as `admin` → `/admin` lists every request from all staff with a **Staff** column; opening one shows details only. There is no sign, reject, approve, send or withdraw button anywhere. |
| [ ] | AC-27 | Every action asks first, then confirms. | For submit, withdraw, send to HR, sign, reject and approve: a confirmation box appears first — **Cancel** changes nothing — and a success message shows after confirming. |
| [ ] | AC-28 | The server refuses actions from the wrong role. | As `employee-1`, open the supervisor sign or HR approve screen's URL directly → you are sent back to `/employee` and the request is unchanged. (Server actions re-check the role; hidden buttons are not the protection.) |

## 5. Not included (for now)

These are **out of scope** for the Leave Request module:

- Leave balances or entitlements (nobody is told how many days they have left).
- Editing a submitted request. The only employee change is **withdraw before
  it is signed**; a mistake means withdrawing and submitting a fresh request.
- Leave types beyond Annual, Sick and Casual.
- A public-holiday calendar (only Saturdays and Sundays are excluded from the
  day count).
- Notifications, email or reminders to any role.
- Medical certificates or any file attachments.
- HR reports or exports, and any Admin changes at all (Admin is view-only).
- Reassigning a request to another supervisor (e.g. when one is away).

## 6. Assumptions

These fill gaps the brief and the interview left open. Read each one; if it is
wrong, say so **before** the design is written.

1. **Who counts as "a supervisor in their own division":** a user with role
   SUPERVISOR in the same division as the employee. The designation (job
   title) plays no part in who may be chosen or who may decide.
2. **The request is tied to the one supervisor chosen at submission.** Only
   that supervisor sees it in their needs-your-action table and may sign or
   reject it — not every supervisor in the division.
3. **Status names** shown to users: *With Supervisor*, *Signed – waiting to
   send to HR*, *At HR*, *Approved*, *Rejected*, *Withdrawn*.
4. **Rejection ends the request.** There is no appeal or re-send path; the
   employee may submit a brand-new request instead.
5. **Remarks** exist only for rejections (supervisor and HR). Signing and
   approving carry no remark. A remark is shown to the employee and the admin
   with the name of who wrote it.
6. **First to act wins** in the HR queue: the officer who presses approve or
   reject first is recorded; the other officer sees the request move to
   history on their next look.
7. **Past dates are allowed** (retroactive sick leave), and there is no
   maximum length of a request.
8. **A "live" request** is any request not yet Rejected or Withdrawn. The
   overlap check in AC-24 compares only one employee's own requests.
9. **A request covering only a weekend counts 0 working days** and is still
   allowed to submit.
10. **Demo data:** `npm run db:seed` creates sample requests that cover every
    status, so each screen can be demoed straight after seeding.
11. **Table columns** as agreed in the interview: Type, From–To, Days, Status
    (plus **Who** = the employee on the supervisor/HR tables, **Staff** on the
    admin table).
