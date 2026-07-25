# Roll Call — Campus Job Board

A single HTML file (`index.html`) with real login, a shared candidate pool, job postings,
and automatic candidate matching per job — powered by Firebase (free tier), deployed on
GitHub Pages.

Nothing here needs a server you host yourself. Firebase *is* the backend; GitHub Pages just
serves the static file.

---

## 1. Create a Firebase project (free)

1. Go to https://console.firebase.google.com → **Add project** → give it any name → finish
   the wizard (you can disable Google Analytics, you don't need it).

## 2. Turn on Email/Password login

1. In your project, go to **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Email/Password**.

## 3. Create the database

1. Go to **Build → Firestore Database → Create database**.
2. Choose **production mode** (not test mode) and any nearby region.
3. Once it's created, go to the **Rules** tab, delete what's there, and paste in the
   contents of `firestore.rules` (included in this folder). Click **Publish**.
   - These rules mean: only logged-in users can read anything, everyone can see the full
     candidate pool and job list, but you can only edit your own profile and delete your
     own job postings.

## 4. Get your config and paste it into `index.html`

1. Go to **Project settings** (gear icon) → scroll to **Your apps** → click the **</>**
   (web) icon → register an app (nickname doesn't matter, skip hosting).
2. Firebase shows you a `firebaseConfig` object. Copy it.
3. Open `index.html`, find this block near the top of the `<script type="module">` section:

   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

4. Replace it with the object Firebase gave you. Save the file.

## 5. Push to GitHub and enable Pages

1. Create a new GitHub repo, and add `index.html` (and optionally `README.md` /
   `firestore.rules`) to it — commit and push.
2. In the repo: **Settings → Pages → Build and deployment → Source: Deploy from a branch**
   → Branch: `main`, folder: `/ (root)` → **Save**.
3. GitHub gives you a URL like `https://yourusername.github.io/your-repo-name/`. It can
   take a minute or two to go live.

## 6. Whitelist your GitHub Pages domain in Firebase

This step is easy to miss and login will silently fail without it:

1. Back in Firebase: **Authentication → Settings → Authorized domains → Add domain**.
2. Add `yourusername.github.io` (just the domain, no path).

That's it — visit your GitHub Pages URL, sign up, fill in a profile, post a job, and you
should see it appear.

---

## How matching works

- Every candidate profile has a `domain` and `experienceYears`.
- Every job has a required `domain` and `minExperience`.
- A candidate is shown as "eligible" for a job when their domain matches (or is contained
  within) the job's domain, **and** their experience is at or above the minimum.
- Matching happens live in the browser — post a job, and any candidate who already fits (or
  later fits, once they update their profile) will show up automatically without a refresh.
- The "eligibility notes" field on a job is free text (e.g. "final-year students only") and
  is just displayed under the posting — it isn't used for automatic filtering, since that
  kind of open-ended requirement isn't reliably matchable by an algorithm.

## Notes / limitations

- Firebase's free "Spark" plan is generous for a project like this (tens of thousands of
  reads/writes a day) — you won't hit limits at student/campus scale.
- All matching is done client-side against every candidate/job in the database. Fine for
  hundreds or a few thousand records; if this ever needs to scale to tens of thousands of
  users, the queries would need to move server-side.
- Anyone with an account can both maintain a candidate profile *and* post jobs — there's no
  separate "recruiter" vs "student" role. If you want that split later (e.g. only certain
  accounts can post), that's a small addition to the Firestore rules plus a role field.
