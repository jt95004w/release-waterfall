# Release Waterfall → live shared website

This turns `release-waterfall.html` into a website your whole group can open and edit
live, for free. The page is **static** (just one file on GitHub Pages); the shared edits
live in **Firebase Firestore** (free tier). Nobody but you needs an account, and anyone
with the link can view and edit.

You do this once. Budget ~45–60 minutes. Order matters: do Firebase first (Step 1), because
Step 2 pastes values you get from it.

---

## Step 1 — Create the free Firebase project (~20 min)

1. Go to **https://console.firebase.google.com** and sign in with any Google account.
2. Click **Add project** (or **Create a project**).
   - Name it anything, e.g. `release-waterfall`.
   - When it asks about **Google Analytics**, toggle it **OFF** — you don't need it.
   - Click **Create project**, wait, then **Continue**.
3. In the left sidebar open **Build → Firestore Database**. Click **Create database**.
   - Choose a location near you (e.g. `nam5 (United States)`). This can't be changed later,
     but any choice is fine.
   - Start in **Production mode** (we'll set the exact rules in step 1.5). Click **Enable**.

### Step 1.5 — Set the security rules (this is what makes "anyone with the link can edit" safe-ish)

1. Still in **Firestore Database**, click the **Rules** tab at the top.
2. Delete everything in the box and paste **exactly** this:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {

       // The one document that holds the whole schedule: anyone may read + write it.
       match /charts/waterfall {
         allow read, write: if true;
       }

       // Everything else in the database is locked.
       match /{document=**} {
         allow read, write: if false;
       }
     }
   }
   ```

3. Click **Publish**.

   > What this means: only the single `charts/waterfall` document is open, and only the
   > people who have your link will ever find it. Nothing else in the project is reachable.
   > The real protection is that the URL is private — treat it like a group password.

### Step 1.6 — Get your config (the six values)

1. Click the **gear icon** (top-left, next to "Project Overview") → **Project settings**.
2. Scroll to **Your apps**. Click the **`</>`** (Web) icon.
3. Give it a nickname (e.g. `waterfall-web`). **Do NOT** check "Firebase Hosting." Click
   **Register app**.
4. Firebase shows a `const firebaseConfig = { ... }` block. Keep this tab open — you need
   those six values (`apiKey`, `authDomain`, `projectId`, `storageBucket`,
   `messagingSenderId`, `appId`) in the next step.

   > These values are **not secret**. They're safe to publish in a static page — security
   > comes from the rules above, not from hiding them.

---

## Step 2 — Paste the config into the file (~5 min)

1. Open `release-waterfall.html` in any text editor (VS Code, Notepad, etc.).
2. Near the **bottom**, find the block that starts with:

   ```js
   var firebaseConfig = {
     apiKey: "PASTE_apiKey_HERE",
     ...
   ```

3. Replace each `"PASTE_..._HERE"` with the matching value from Firebase. The result looks
   like:

   ```js
   var firebaseConfig = {
     apiKey: "AIzaSyB....",
     authDomain: "release-waterfall.firebaseapp.com",
     projectId: "release-waterfall",
     storageBucket: "release-waterfall.appspot.com",
     messagingSenderId: "123456789012",
     appId: "1:123456789012:web:abc123..."
   };
   ```

4. Save the file. **Test it locally:** double-click the file to open it in your browser.
   The status dot near the top-right should go from "Loading…" to **"Shared · edits save for
   everyone with access."** Drag a bar, refresh the page — your change should still be there.
   If it says "Edits stay in this browser only," the config wasn't pasted correctly.

---

## Step 3 — Put it online with GitHub Pages (~15 min)

1. On **https://github.com** (your account, JacobTpng), click **New repository**.
   - Name it e.g. `release-waterfall`.
   - Public or Private both work with Pages.
   - Click **Create repository**.
2. Add the file: on the empty repo page, **uploading files** → drag in your edited
   `release-waterfall.html`. **Rename it to `index.html`** (so the bare URL loads it).
   Commit.
   - *Or, from a terminal:* `git clone` the repo, copy the file in as `index.html`,
     `git add`, `git commit`, `git push`.
3. In the repo, go to **Settings → Pages**.
   - Under **Build and deployment → Source**, pick **Deploy from a branch**.
   - Branch: **main**, folder: **/ (root)**. Click **Save**.
4. Wait 1–2 minutes. The Pages section will show your live URL, e.g.
   **`https://jacobtpng.github.io/release-waterfall/`**.

That URL is the website. Open it — the status should say **"Shared."**

---

## Step 4 — Share it with the group

- Send the group the GitHub Pages URL. Anyone who opens it can view and drag-edit live;
  changes appear for everyone within a second or two.
- The **first person to load it** seeds the schedule from the built-in data (the plan you
  have now). After that, the live Firestore copy is the source of truth.

---

## Habits & safety net

- **Backup button** (top toolbar): downloads the whole schedule as a dated `.json` file.
  Do this once a week, or before any big change. It's your undo for a leaked link or a
  bad edit.
- **Restore button**: pick a backup `.json` to replace the schedule for everyone (it asks
  you to confirm first). This is how you recover.
- **Updating the look/features of the page later:** edit `index.html` and push a new commit
  (or re-upload). The schedule data is untouched — it lives in Firebase, not in the file.

## What the status dot means

| Text | Meaning |
|---|---|
| Shared | Connected; edits save for everyone. |
| Saving… / Saved | A change is being written / was written. |
| Edits stay in this browser only | Firebase config missing or wrong — fix Step 2. |
| View only | The rules are blocking writes — check Step 1.5. |
| Shared copy not reachable yet | Network hiccup; usually fixes itself on refresh. |

## If something's off

- **"Edits stay in this browser only"** → the six config values aren't pasted right, or one
  still says `PASTE_`. Re-check Step 2.
- **"View only"** → the Firestore rules didn't publish, or the path differs. The code uses
  `charts/waterfall`; the rules must match. Re-do Step 1.5.
- **Page loads but is blank** → make sure the file is named `index.html` in the repo and
  Pages is set to the `main` branch, root folder.
