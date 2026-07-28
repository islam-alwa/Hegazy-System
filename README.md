# Teacher Management System (Firebase-Connected, Role-Based)

A teacher management system built with plain HTML, CSS, and JavaScript, fully connected to
**Firebase Authentication** and **Firestore**. Two people share the same data with different
permissions:

- **الاسستنت (assistant / "manager" role):** full access — adds students, groups, attendance,
  subscriptions, exams, etc.
- **المستر (teacher / "teacher" role):** read-only — sees **one number**: total money collected.
  Nothing else.

## How data is stored (where, exactly)

Everything lives in one Firebase project, under a **single shared account space** (not per-login-account):

| Firestore path                              | What's in it                                   | Who can access |
|----------------------------------------------|-------------------------------------------------|----------------|
| `system/main/appData/state`                  | The entire app data (students, groups, lessons, attendance, subscriptions, exams, notes, settings) as one JSON document | Assistant only (read + write) |
| `system/main/appData/totalMoney`             | `{ total: number, updatedAt }` — recalculated automatically every time the assistant saves | Assistant (read/write), Teacher (read only) |
| `system/main/appData/connectionTest`         | Scratch doc used by the "اختبار الاتصال" button | Assistant only |
| `system/main/members/{uid}`                  | `{ email, role: 'manager' \| 'teacher' }` — who is allowed to do what | Each user can read their own doc only |

Firebase Authentication holds the actual login accounts (email + password) — that's separate
from Firestore and is what gives each person a `uid`. The `members` collection is what maps a
`uid` to a role. **Nothing is stored in the browser (`localStorage`) anymore** — every read/write
goes straight to Firestore, so the app is fully Firebase-dependent, not a local/offline app.

## One-time setup (do this once)

### 1. Enable Firebase basics
- Authentication → Sign-in method → enable **Email/Password**.
- Firestore Database → Create database (Native mode, Spark/free plan is fine).
- Firestore → Rules → paste the contents of `firestore.rules` from this repo.

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
- Add a document with **Document ID = the assistant's UID**, field `role` = `manager` (string),
  optionally `email` = their email.
- Add another document with **Document ID = the teacher's UID**, field `role` = `teacher`.
- Roles are only editable from the Console on purpose — this keeps a browser (even a hacked one)
  from ever granting itself admin/assistant access.

Now log in again with each account — the assistant gets the full app, the teacher gets the
one-number totals screen.

## Monthly auto-generated lessons
Every group has a fixed weekday + time (`dayOfWeek` / `time`). Each time the **assistant** opens
the app, it checks: "does this group already have its 4 auto-generated lessons for this calendar
month?" If not, it creates them on the group's weekday (first 4 occurrences of that weekday in
the month) automatically and saves.

> **Free (Spark) plan note:** Firebase's free plan has no background/scheduled functions, so this
> can't fire itself exactly at midnight on the 1st with nobody around. It generates lessons the
> next time the assistant opens the app on/after the 1st — which in practice is the same result,
> just triggered by opening the app instead of a clock. If you upgrade to Blaze later, this can be
> moved into a real scheduled Cloud Function that runs with zero manual steps.

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
- Firebase Authentication (email/password) with two roles: assistant (full access) and teacher
  (totals-only)
- "مسجل الدخول الآن" banner always visible, showing who's logged in and their role
- Firestore sync — no local/offline storage, same data from any device
- Connection status badge (topbar + Settings) + a manual "اختبار الاتصال الآن" button
- Auto-generated monthly lessons per group (4 per month, on the group's day/time)
- Academic years: 3 Prep (Middle School) + 3 Secondary years
- Dashboard with cards and simple charts
- Students with profile, guardian, attendance, subscriptions, exams, notes
- QR code per student — from the profile, per row in the table, or bulk-print for a whole year
- Monthly subscriptions due on the 1st of each month
- Search, sort, filter, printable reports and QR cards
- JSON export/import and CSV export as backups

