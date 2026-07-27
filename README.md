# Teacher Management System (Firebase-Connected)

A teacher management system built with plain HTML, CSS, and JavaScript, connected to
**Firebase Authentication** and **Firestore** so your data syncs across devices.

## How to run locally
Browsers block Firebase (ES Modules) when you open `index.html` directly by double-click.
You must run it through a local server:

### Windows
double-click `run.bat`

### macOS / Linux
```bash
bash run.sh
```

Then open `http://localhost:8000` (not the file directly).

## Hosting on GitHub Pages
1. Push this repo to GitHub.
2. Repo → Settings → Pages → Deploy from branch → pick `main` (or your default branch) and `/root`.
3. Your site will be live at `https://<username>.github.io/<repo-name>/`.
4. **Important:** go to Firebase Console → Authentication → Settings → **Authorized domains**,
   and add your GitHub Pages domain (e.g. `<username>.github.io`), otherwise login will fail
   with an `auth/unauthorized-domain` error on the live site.

## Firebase setup (one-time)
1. **Authentication** → Sign-in method → enable **Email/Password**.
2. **Firestore Database** → Create database (Native mode).
3. **Firestore Rules** (Firestore → Rules) — restrict each user to their own data:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```
> The `firebaseConfig` values in `index.html` (apiKey, projectId, etc.) are not secret — Firebase
> web apps are always public. Real protection comes from the Firestore rules above and from
> restricting Authorized domains, not from hiding the config.

## First login
Enter any email + password (6+ characters). The first time creates your account automatically;
after that, sign in with the same email/password. Each account has its own private data in Firestore.

## Features
- Firebase Authentication (email/password) — real accounts, not a hardcoded login
- Firestore sync — open from any device, same data, saved in real time
- Connection status badge (topbar + Settings page) + a manual "test connection" button
- Academic years: 3 Prep (Middle School) + 3 Secondary years
- Dashboard with cards and simple charts
- Year-separated dashboards and filters
- Students with profile, guardian, attendance, subscriptions, exams, notes
- **QR code per student** — view/print from the student profile, from the students table (per row),
  or bulk-print all students in the current year via "طباعة QR لكل الطلاب"
- Groups with 4 lessons per month by default
- Monthly subscriptions due on the 1st of each month
- Search, sort, filter
- Printable reports and QR cards
- JSON export/import and CSV export as backups

## Notes
- Data used to live only in the browser's `localStorage`; it now lives in Firestore under
  `users/{your-uid}/appData/state`, so switching devices/browsers keeps your data as long as
  you sign in with the same account.
- The old `localStorage`-only login (`admin` / `admin123`) has been fully replaced.
