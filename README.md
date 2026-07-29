# Teacher Management System (Firebase-Connected, Role-Based)
 
A teacher management system built with plain HTML, CSS, and JavaScript, fully connected to
**Firebase Authentication** and **Firestore**. Two people share the same data with different
permissions:
 
- **المستر (teacher / "teacher" role):** full access, same as the assistant — students, groups,
  attendance, lessons, exams, notes, **and all money/subscription data** (amounts, group fees,
  revenue reports).
- **الاسستنت (assistant / "manager" role):** full access to everyday work (students, groups,
  attendance, lessons, exams, notes) but **zero access to any money figure anywhere** — no
  amounts, no group fees, no revenue numbers, on any page. They can still mark a subscription as
  "paid / late / unpaid" for the current month (so day-to-day payment collection keeps working),
  but never sees or enters an actual number.
## How data is stored (where, exactly)
 
Everything lives in one Firebase project, under a **single shared account space** (not
per-login-account). Money is split into its own document specifically so it can be hidden from
the assistant's account at the database level — not just hidden in the UI:
 
| Firestore path                              | What's in it                                                              | Who can access |
|----------------------------------------------|-----------------------------------------------------------------------------|----------------|
| `system/main/appData/state`                  | Students, groups (day/time/name — **no fee**), lessons, attendance, exams, notes, `paymentStatus` (see below), settings | Both roles (read + write) |
| `system/main/appData/money`                  | The real subscriptions (`dueAmount`, `paidAmount`, status) + each group's `fee` | **Teacher only** (read + write). The assistant's Firestore rules flat-out deny this document — even inspecting network traffic from their browser won't reveal it. |
| `system/main/appData/connectionTest`         | Scratch doc used by the "اختبار الاتصال" button                             | Both roles |
| `system/main/members/{uid}`                  | `{ email, role: 'manager' \| 'teacher' }` — who is allowed to do what        | Each user can read their own doc only |
 
### How "assistant can mark paid but can't see the amount" works
The assistant's account never touches the `money` document, so there's a lightweight parallel
list in the main `state` document called `paymentStatus` — just `{ studentId, month, status,
paidAt, paymentMethod }`, **no amount field at all**. When the assistant clicks "تسجيل الدفعة" on
a student, it only edits an entry in this list. The next time the **teacher** logs in, the app
reconciles: any status the assistant recorded gets copied onto the real subscription record in
`money` (filling in the amount from the group's fee automatically). This is a one-way street —
amounts never flow back down to the assistant's side.
 
**Note on real-world security:** this is a genuine Firestore-level restriction (enforced by the
security rules below, not just hidden in the interface), which is the best that's realistically
achievable without a paid backend. It's appropriate for a small, trusted 2-person team; it isn't
meant to withstand a hostile actor with full control of their own Firebase project.
 
## One-time setup (do this once)
 
### 1. Enable Firebase basics
- Authentication → Sign-in method → enable **Email/Password**.
- Firestore Database → Create database (Native mode, Spark/free plan is fine).
- Firestore → Rules → paste the contents of `firestore.rules` from this repo → Publish.
### 2. Create the two login accounts
Open the site and log in twice, once per person, with two **different emails**:
- Log in with the assistant's email + password (6+ chars) → account is created automatically.
- Log out, then log in with the teacher's email + password → account is created automatically.
At this point both accounts exist in Firebase but **neither has a role yet**, so the app will
say "حسابك لسه مش متسجل له صلاحية" and sign them back out. That's expected — do step 3 next.
 
### 3. Assign roles (manual, in Firebase Console — by design, not doable from the app)
- Firebase Console → Authentication → Users → copy the **User UID** for each account.
- Firestore Database → Data → create collection `system` → document `main` →
  subcollection `members`.
- Add a document with **Document ID = the assistant's UID**, field `role` = `manager` (string).
- Add another document with **Document ID = the teacher's UID**, field `role` = `teacher`.
- Roles are only editable from the Console on purpose — this keeps a browser (even a hacked one)
  from ever granting itself teacher/money access.
Now log in again with each account — both get the full app; only the teacher sees money.
 
## Monthly auto-generated lessons & subscriptions
Every group has a fixed weekday + time (`dayOfWeek` / `time`). Each time **either** account opens
the app, it checks: "does this group already have its 4 auto-generated lessons for this calendar
month?" If not, it creates them on the group's weekday automatically.
 
Separately, each time a student is missing a `paymentStatus` entry for the current month, one is
created (status `unpaid`, no amount). When the **teacher** next logs in, those get turned into
real subscription records in `money` with the correct due amount pulled from the group's fee.
 
> **Free (Spark) plan note:** Firebase's free plan has no background/scheduled functions, so
> nothing fires itself at midnight on the 1st with nobody around. Generation happens the next
> time someone opens the app on/after the 1st — in practice the same result, just triggered by
> opening the app instead of a clock.
 
## How to run locally
Browsers block Firebase (ES Modules) when you open `index.html` directly by double-click.
Run it through a local server:
 
### Windows
double-click `run.bat`
 
### macOS / Linux
```bash
bash run.sh
```
Then open `http://localhost:8000` (not the file directly).
 
## Hosting on GitHub Pages
1. Push this repo to GitHub.
2. Repo → Settings → Pages → Deploy from branch → `main` / `/root`.
3. Site goes live at `https://<username>.github.io/<repo-name>/`.
4. Firebase Console → Authentication → Settings → **Authorized domains** → add
   `<username>.github.io`, otherwise login fails with `auth/unauthorized-domain` on the live site.
> The `firebaseConfig` values in `index.html` are not secret — Firebase web apps are always
> public. Real protection comes from the Firestore rules and role checks above.
 
## Features
- Firebase Authentication (email/password) with two roles: teacher (full access incl. money) and
  assistant (full access, money hidden everywhere)
- "مسجل الدخول الآن" banner always visible, showing who's logged in and their role
- Firestore sync — no local/offline storage, same data from any device
- Connection status badge (topbar + Settings) + a manual "اختبار الاتصال الآن" button
- Auto-generated monthly lessons per group (4 per month, on the group's day/time)
- Auto-generated monthly payment-status placeholders, reconciled into real amounts on the
  teacher's next login
- Academic years: 3 Prep (Middle School) + 3 Secondary years
- Dashboard with cards and simple charts (revenue card hidden for the assistant)
- Students with profile, guardian, attendance, subscriptions, exams, notes
- QR code per student — from the profile, per row in the table, or bulk-print for a whole year
- Search, sort, filter, printable reports and QR cards
- JSON export/import and CSV export as backups
 
