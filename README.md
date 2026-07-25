# Roll Call — Campus Job Board

A single HTML file (`index.html`) with real login, a shared candidate pool, job postings
with automatic candidate matching, resume links, notice periods, one-tap reach-out
(Gmail / Call / WhatsApp), auto-archiving, and a role-based admin panel — powered by
Firebase (free tier), deployed on GitHub Pages.

Your Firebase config is already pasted into `index.html`, pointing at the `m-working-a526f`
project.

---

## What's new in this version

- **Admin access panel** — `fiberxpool@gmail.com` and `ceo@machinemind.in` are hardcoded
  as permanent "Owners" (see `OWNER_EMAILS` near the top of the `<script>` in `index.html`,
  and the matching list in `firestore.rules`). Owners see an **Admin** tab where they can
  grant any signed-up person one or more of three permissions:
  - **Jobs** — archive/delete anyone's job posting, not just their own
  - **Candidates** — remove a candidate profile
  - **Admins** — grant/revoke access for other people (i.e. make more admins)
  Owners can promote others to admin from that same screen — no code changes needed for
  that part.
- **Resume link** — candidates paste a share link (Google Drive/Dropbox) rather than
  uploading a file. See "Why a link and not a file upload" below.
- **Notice period** — a simple dropdown on the candidate profile, shown next to their name
  wherever they appear as an eligible candidate.
- **Reach-out buttons** — every email shows an **✉ Email** button that opens a Gmail
  compose window (not the OS mail app); every phone number shows **📞 Call** and
  **💬 WhatsApp** buttons. These appear on job postings (to reach the poster) and on each
  eligible candidate (to reach them).
- **Archiving** — a job automatically counts as "closed" once its last-date-to-apply
  passes (candidate matching stops being shown for it), and disappears from the main list.
  Anyone with **Jobs** access can also archive a posting early. A "Show archived / closed
  postings" checkbox reveals them again. Archiving is reversible only by deleting — there's
  no "un-archive" button by design, to keep old data settled once it's closed.

## Why a resume *link* and not a file upload

Firebase Cloud Storage now requires the paid **Blaze** plan (usage still free within quota,
but a card must be on file) — Google made this mandatory for all projects, old and new, as
of February 2026. To keep this project entirely on the free Spark plan with zero billing
setup, resumes are a pasted link instead of an uploaded file. If you'd rather have real
file uploads later, that's a small addition once you're on Blaze — just ask.

---

## 1. Re-publish the security rules

Because this version adds the `admins` and `users` collections and new permission logic,
you need to update the rules even if you already set some up before:

1. Firebase console → your project → **Build → Firestore Database → Rules**.
2. Replace everything with the contents of `firestore.rules` (in this folder).
3. Click **Publish**.

> If publishing errors specifically on the `.lower()` call: your Firestore rules version
> doesn't support it. Just delete `.lower()` from that one line — since both owner emails
> are already lowercase, it'll work fine as long as you sign up using exactly
> `fiberxpool@gmail.com` and `ceo@machinemind.in` (lowercase).

## 2. Make sure the owners can actually get in

`fiberxpool@gmail.com` and `ceo@machinemind.in` need to **sign up** on the live site once
(same as any other user) — the "Owner" badge and Admin tab appear automatically the moment
they log in with that exact email, no manual setup required in Firestore.

## 3. Everything else — same as before

If you haven't deployed yet: Firebase Authentication (Email/Password) needs to be turned
on, and your GitHub Pages domain needs to be added under **Authentication → Settings →
Authorized domains**, or login will silently fail. Full first-time steps are below.

<details>
<summary>First-time setup (click to expand)</summary>

### Turn on Email/Password login
Firebase console → **Build → Authentication → Get started** → enable **Email/Password**.

### Create the database
**Build → Firestore Database → Create database** → production mode, any nearby region →
then paste `firestore.rules` into the Rules tab and publish (see step 1 above).

### Push to GitHub Pages
1. New repo → **Add file → Upload files** → drag in `index.html`, `README.md`,
   `firestore.rules` → commit.
2. **Settings → Pages** → Source: Deploy from a branch → Branch `main`, folder `/ (root)`
   → Save. Your URL appears on that same screen a minute or two later.
3. Firebase console → **Authentication → Settings → Authorized domains → Add domain** →
   add `yourusername.github.io`.

</details>

## How matching works

- Every candidate profile has a `domain` and `experienceYears`.
- Every job has a required `domain` and `minExperience`.
- A candidate is "eligible" when their domain matches (or is contained within) the job's
  domain, and their experience is at or above the minimum. This runs live in the browser —
  no refresh needed.
- The free-text "eligibility notes" field on a job (e.g. "final-year only") is displayed
  but not auto-filtered, since that kind of requirement isn't reliably matchable.

## Notes / limitations

- Firebase's free Spark plan comfortably covers this at campus scale.
- Matching and the admin people-list are computed client-side against everything in the
  database — fine for hundreds to a few thousand records.
- The admin panel's people list is built from everyone who has ever logged in (`users`
  collection) plus anyone with a saved profile (`candidates`) — it can't see people who
  have a Firebase Auth account but have never opened the app since deploying this version,
  since client apps can't list Firebase Authentication users directly.
- WhatsApp links assume a 10-digit number without a country code is Indian (+91). If a
  number already includes a country code, leave it as typed and it'll still work.
