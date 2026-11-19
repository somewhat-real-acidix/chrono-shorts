# Chrono: Shorts from your subscriptions, newest first

A one-page web app. Sign in with Google, and it plays Shorts from every channel you subscribe to in strict chronological order. It works on your PC and on your iPhone (added to the Home Screen it opens like an app). Everything is free.

**What's in this folder**

| File | What it is |
|---|---|
| `index.html` | The whole app |
| `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Home-screen icon and app name for phones |
| `README.md` | This guide |

**The big picture.** You'll do four things, about 30–40 minutes in total:

1. Register your app with Google: enable the YouTube API, set up the consent screen, and create a free "OAuth client" in Google Cloud.
2. Try it on your PC, at an address only your PC can see.
3. Put it on the internet for free with GitHub Pages, so your phone can reach it.
4. Add it to your iPhone's Home Screen.

> **Why not just double-click `index.html`?** Google sign-in and the YouTube player both refuse to work on pages opened as files (`file://…`). The app has to be served from a web address: `http://localhost:8000` on your PC for testing, or your GitHub Pages address for real use.

---

## Step 1: Set up Google sign-in (Google Cloud)

**What you're doing and why.** Before Google lets any app show a "Sign in with Google" screen, the app has to be registered in Google Cloud. For Chrono, registering means three things:

| Piece | What it is | Why Chrono needs it |
|---|---|---|
| **YouTube Data API v3** (enabled) | Permission for your project to call YouTube's API | This is how Chrono reads your subscriptions and each channel's Shorts |
| **Consent screen** (Google Auth Platform) | The name, email and allowed users shown on Google's sign-in screen | Google won't sign anyone in without it. In **Testing** mode, only people you list can sign in, which means just you |
| **OAuth client** (Client ID) | Your app's ID card, plus the list of web addresses it's allowed to run from | Google only sends a sign-in back to an address on this list |

All of it is free, and none of it needs a credit card or billing set up. Budget about 10–15 minutes.

> **Use the same Google account throughout.** Sign in to the Google Cloud console with the account you watch YouTube on, or at least add that account as a test user in 1c. If Chrome is signed in to several Google accounts, check the avatar at the top right of the console before you start.

---

### 1a. Open (or create) your project

A *project* is the folder that holds everything below. You made one for the Chrome extension's API key, and you can reuse it.

1. Go to <https://console.cloud.google.com/>.
2. Find the **project picker** at the top left, next to the "Google Cloud" logo. It shows the current project's name, or "Select a project".
   - **To reuse your extension's project:** click the picker and choose it.
   - **To start fresh:** click the picker → **New project** → name it `Chrono` → **Create**. Wait for the notification bell to say it's done, then select it in the picker.
3. **Check:** the picker now shows your project's name. Everything in the following steps happens inside this project. If you switch projects by accident, the client and the enabled API end up in different places and sign-in fails.

> The first time you use Google Cloud, it may ask you to agree to the Terms of Service or pick a country. Agree; it costs nothing. If it offers a "free trial" with billing, you can close that offer; Chrono doesn't need billing.

---

### 1b. Enable the YouTube Data API v3

1. Open <https://console.cloud.google.com/apis/library/youtube.googleapis.com>. Or use the ☰ menu → **APIs & Services → Library** and search for **YouTube Data API v3**. Choose the one with exactly that name, not "YouTube Analytics API" or "YouTube Reporting API".
2. Look at the blue button:
   - **Enable:** click it. After a few seconds you land on the API's overview page.
   - **Manage:** it's already enabled, probably from the extension. Nothing to do.
3. **Check:** ☰ → **APIs & Services → Enabled APIs & services** lists **YouTube Data API v3**.

> **The API key from the extension isn't needed here.** The web app signs you in instead of using a key. Keeping the key doesn't hurt; delete it under **APIs & Services → Credentials** if you no longer use the extension.

---

### 1c. Set up the consent screen (Google Auth Platform)

This is the screen you'll see when you sign in: "Chrono wants to access your Google Account". Google moved it in 2024; it now lives under **Google Auth Platform**, split into **Branding**, **Audience**, **Clients** and **Data Access** pages. If a guide elsewhere mentions the "OAuth consent screen", it means this.

1. Open <https://console.cloud.google.com/auth/overview>, or ☰ → **Google Auth Platform**.
2. If you see **"Google Auth Platform not configured yet"**, click **Get started**. A short wizard follows:
   1. **App information**
      - **App name:** `Chrono`. It's shown to you on the sign-in screen.
      - **User support email:** pick your Gmail from the dropdown.
      - Click **Next**.
   2. **Audience:** choose **External**.
      - "External" just means personal Google accounts (gmail.com) are allowed. **Internal** is only offered to Google Workspace organizations, and only works for accounts inside that organization.
      - Click **Next**.
   3. **Contact information:** enter your email (Google sends notices about the project here) → **Next**.
   4. **Finish:** tick **I agree to the Google API Services: User Data Policy** → **Continue** → **Create**.
3. **Add yourself as a test user.** This is the step people most often miss.
   1. Open **Audience** in the left menu (<https://console.cloud.google.com/auth/audience>).
   2. **Publishing status** should say **Testing**. Leave it that way. Don't click "Publish app": that would require Google to review the app.
   3. Scroll to **Test users** → **+ Add users** → type the Google account you use on YouTube → **Save**.
   4. **Check:** your email appears in the test users list.
   - *Why:* in Testing mode, only listed accounts can sign in. Anyone else, including you if you forget this, gets "Access blocked: Chrono has not completed the Google verification process."
4. **Add the YouTube scope.** This step is optional but tidy.
   1. Open **Data Access** (<https://console.cloud.google.com/auth/scopes>) → **Add or remove scopes**.
   2. In the filter box, type `youtube.readonly` and tick **…/auth/youtube.readonly** ("View your YouTube account").
   3. Click **Update**, then **Save** at the bottom of the page.
   - Google labels this scope "sensitive" and mentions verification. Verification only applies if you publish the app to the public. In Testing mode with yourself as the only user, you don't need it.
   - *What the scope allows:* read-only access to your YouTube account, such as subscriptions and playlists. Chrono can't post, comment, like or change anything.

---

### 1d. Create the OAuth client (your Client ID)

1. Open **Clients** (<https://console.cloud.google.com/auth/clients>) → **+ Create client**.
2. **Application type:** **Web application**. Not "Desktop app" or "iOS". Even on your iPhone, Chrono runs as a web page.
3. **Name:** `Chrono web`. This is only for you, inside the console.
4. **Authorized JavaScript origins** → **+ Add URI** → enter exactly:
   ```
   http://localhost:8000
   ```
   An *origin* is just the site's address with no path: no slash or anything else after the port number. Google rejects it otherwise ("Invalid Origin: URIs must not contain a path or end with '/'").
5. **Authorized redirect URIs** → **+ Add URI** → enter exactly:
   ```
   http://localhost:8000/
   ```
   This time **with** the trailing slash. It's the page Google sends you back to after sign-in, and it has to match the app's address character for character.
6. Click **Create**.
7. A box appears with the **Client ID**, a long value ending in `.apps.googleusercontent.com`. Click the copy icon and paste it somewhere handy, such as Notepad. You can always find it again later in **Clients → Chrono web**.
   - The box may also show a **Client secret**. Chrono doesn't use it; you don't need to save or download it. Web pages can't keep secrets, so this kind of sign-in is designed to work without one.
8. **Check:** **Clients** now lists **Chrono web**, of type "Web application".

> **Coming back later (Step 3):** once your app is on GitHub Pages, you'll reopen **Clients → Chrono web** and add a second origin and redirect URI for the GitHub address. A client can hold several addresses, so `localhost` and GitHub Pages both keep working.

> **Changes can be slow to apply.** After you create or edit a client, Google can take from a few minutes up to a couple of hours to use the new addresses. If sign-in shows `redirect_uri_mismatch` right after an edit, and the addresses look correct, wait 10–15 minutes and try again.

---

### Step 1 checklist

Before moving on, confirm each item:

- [ ] The project picker shows the same project you used for every step.
- [ ] **Enabled APIs & services** lists **YouTube Data API v3**.
- [ ] **Google Auth Platform → Audience:** status **Testing**, User type **External**, and your YouTube account under **Test users**.
- [ ] **Google Auth Platform → Clients:** a **Web application** client with origin `http://localhost:8000` and redirect URI `http://localhost:8000/`.
- [ ] You've copied the **Client ID** (ends in `.apps.googleusercontent.com`).

### What sign-in will look like

When you first sign in to Chrono in Step 2, expect these screens. All of them are normal:

1. **Choose an account:** pick your YouTube account, the one you added as a test user.
2. **"Google hasn't verified this app":** click **Continue**. On some versions you click **Advanced** first, then **Go to Chrono (unsafe)**. "Unsafe" only means Google hasn't reviewed it, which is expected for a personal app.
3. **"Chrono wants access to your Google Account"** with **View your YouTube account:** if there's a checkbox, tick it, then click **Continue**.

After that you're sent back to Chrono. Next time, sign-in is usually one tap, because Google remembers you already allowed it.

### If something goes wrong in Step 1

| Problem | Likely cause and fix |
|---|---|
| You can't find "Google Auth Platform" | Use the search bar at the top of the console and type "Google Auth Platform", or go straight to <https://console.cloud.google.com/auth/overview>. |
| "Internal" is greyed out | Normal for personal Gmail accounts. Choose **External**. |
| "Invalid Origin" when adding `http://localhost:8000/` | Remove the trailing slash from the **origin**. Only the **redirect URI** has a slash. |
| The client was created in the wrong project | Switch to the correct project in the picker and create the client again there. Delete the stray one if you like. |
| "Access blocked … has not completed the Google verification process" when you sign in | Your account isn't under **Audience → Test users**, or you signed in with a different Google account. Add it, or choose the right account. |
| The console asks for billing | You've landed on a paid product page. Chrono needs nothing that's billed, so go back to the links above. |

---

## Step 2: Test on your PC

You need a tiny local web server. The simplest one comes with Python.

### Windows
1. **Check for Python:** open **Command Prompt** (Start → type `cmd`) and run `python --version`.
   - If you get a version number, skip to step 3.
   - If it opens the Microsoft Store or says "not found", install Python from the Store ("Python 3.12" or newer) or from <https://www.python.org/downloads/>. With the python.org installer, tick **"Add python.exe to PATH"**.
2. Unzip the app folder somewhere easy, for example `C:\Users\<you>\chrono-web`.
3. In Command Prompt, start the server:
   ```
   cd C:\Users\<you>\chrono-web
   python -m http.server 8000
   ```
   You should see `Serving HTTP on :: port 8000`. Leave this window open: closing it stops the server.

### Mac
Open **Terminal** and run `cd ~/Downloads/chrono-web` (or wherever you unzipped it), then `python3 -m http.server 8000`.

### Try it
1. In Chrome, go to **`http://localhost:8000`**. Type `localhost` exactly; `127.0.0.1` counts as a different address and Google will reject it.
2. Paste your **Client ID** and click **Save**.
3. Click **Sign in with Google**. You'll see a few screens:
   - Choose your account.
   - **"Google hasn't verified this app"**: this is expected, because it's your own app. Click **Continue**. (Some screens show **Advanced → Go to Chrono (unsafe)** instead.)
   - Allow **"View your YouTube account."** If there's a checkbox, tick it.
4. You're sent back to the app, which reads your subscriptions and loads the feed. The first load checks every channel and takes a few seconds; later loads use the cache.

To stop the server, press **Ctrl+C** in the Command Prompt window.

---

## Step 3: Put it online with GitHub Pages (free)

Your phone can't reach `localhost`, so the app needs a real web address. GitHub Pages hosts simple sites like this for free at `https://<your-username>.github.io/<repo-name>/`.

### 3a. Create the site
1. Make a free account at <https://github.com>, if you don't have one. Your **username** becomes part of the address.
2. Click **+** (top right) → **New repository**:
   - **Repository name:** `chrono-shorts`
   - Choose **Public**. Free GitHub Pages needs a public repository. That's fine: nothing in these files is secret (more below).
   - Click **Create repository**.
3. On the new repository's page, click **uploading an existing file** (or **Add file → Upload files**).
4. Drag in the **files inside** the app folder: `index.html`, `manifest.webmanifest`, the three `.png` files, and `README.md`. Drag the files themselves, not the folder.
5. Click **Commit changes**.
6. Open **Settings → Pages** (in the left sidebar of the repository settings):
   - **Source:** **Deploy from a branch**
   - **Branch:** `main`, folder **/ (root)**, then **Save**
7. Wait 1–2 minutes and refresh the Pages settings page. It will show **"Your site is live at https://<username>.github.io/chrono-shorts/"**.

### 3b. Tell Google about the new address
Go back to **Google Auth Platform → Clients → Chrono web** (<https://console.cloud.google.com/auth/clients>) and add:
- **Authorized JavaScript origins:** `https://<username>.github.io` (no path and no slash at the end; Google will reject it otherwise)
- **Authorized redirect URIs:** `https://<username>.github.io/chrono-shorts/` (with the slash at the end)

Click **Save**, then open `https://<username>.github.io/chrono-shorts/` and set it up just as you did on localhost. Each address keeps its own settings and sign-in, so you enter the Client ID again.

> **Tip:** In the app, open **⚙ Settings**. It shows "This page's address", which is exactly the redirect URI Google expects for that copy of the app.

> **Is a public repo safe?** Yes. The files contain no passwords or keys. The Client ID is only typed into the app, and even if it were in the files, it isn't secret: Google only lets it sign in from the addresses you listed. Your sign-in token, subscriptions and settings live only in your own browser's storage, never in the repo.

### Updating the app later
In the repository, click `index.html` → the pencil icon (or **Add file → Upload files** to replace it), then **Commit**. The site updates within a minute or two. If your phone still shows the old version, close and reopen the app.

---

## Step 4: Add it to your iPhone

1. On the iPhone, open **Safari** (not Chrome) and go to `https://<username>.github.io/chrono-shorts/`.
2. Tap the **Share** button → **Add to Home Screen** → **Add**.
3. Open **Chrono** from the Home Screen. It opens full-screen without Safari's address bar.
4. Enter the Client ID and sign in once more. The Home Screen app has its own storage, separate from Safari.

**Sound on iPhone:** Safari won't start a video with sound until you tap something. If a Short starts muted, tap **🔇 Tap for sound** once. After that, Shorts should keep playing with sound as you swipe; if one starts muted, tap the badge again.

---

## Using it

**The grid (home screen).** Chrono opens on a grid of Shorts from your subscriptions, newest first and grouped by day (Today, Yesterday, then dates).
- Tap any Short to start playing from it.
- **← Grid** (top bar), `Esc` or `Backspace` takes you back. The Short you were on gets a blue outline, so you can find your place.
- **All / Unwatched** switches between everything and only what you haven't seen. Chrono remembers your choice.
- If you open a Short from the **Unwatched** view, the player steps through only unwatched Shorts.
- **Mark all watched** greys out everything in the current feed, for when you want a clean slate.

**Watched Shorts** are greyed out with a ✓.
- A Short counts as watched once you've watched half of it, or 5 seconds, whichever comes first.
- The watched list is stored in each browser separately: your PC and your phone each keep their own.
- Chrono can't see what you watched in the YouTube app itself, because YouTube's API doesn't share watch history.
- To start over, use ⚙ Settings → **Clear watched history**.

**In the player:**

| Action | Phone | Computer |
|---|---|---|
| Next (older) Short | Swipe up | ↓, `j`, scroll down, or ▼ |
| Previous (newer) Short | Swipe down | ↑, `k`, scroll up, or ▲ |
| Pause / play | Tap the video | Click the video, or Space |
| Sound on/off | 🔇 badge | `m` or 🔊 |
| Back to the grid | ← Grid | `Esc`, `Backspace`, or ← Grid |
| Open on YouTube | Tap the channel name | ↗ button |

- **Refresh** (top bar) checks every channel again. Without it, the app reuses data it fetched in the last 30 minutes, and your subscription list is re-read twice a day.
- **⚙ Settings:** how many days back to show (default 14), how many recent Shorts to check per channel, and whether a finished Short replays or moves on to the next one.
- **Sign-in lasts about an hour.** That's normal for this kind of app. After that, you can keep watching the cached feed; pressing **Refresh** quickly signs you back in, usually with one tap and no consent screen.

**Quota:** each refresh costs roughly 1 unit per channel, plus 1 per 50 subscriptions. The free daily limit is 10,000 units, so even several hundred subscriptions refreshed many times a day is fine.

---

## Troubleshooting

| What you see | Fix |
|---|---|
| **Error 400: redirect_uri_mismatch** | The page's address doesn't exactly match a redirect URI on your client. Copy the address from ⚙ Settings into **Authorized redirect URIs**, including `http`/`https` and the trailing slash. Wait a few minutes and try again. |
| **Error 400: origin_mismatch** or an invalid origin | Add the origin (`http://localhost:8000` or `https://<username>.github.io`, with no path) under **Authorized JavaScript origins**. |
| **"Access blocked: Chrono has not completed the Google verification process"** | Add your Google account under **Audience → Test users** (Step 1c). |
| **"Google hasn't verified this app"** | Expected. Click **Continue** or **Advanced → Go to Chrono**. |
| "The YouTube Data API v3 isn't enabled…" | Do Step 1b in the **same** project as your OAuth client. |
| "Daily YouTube API quota used up" | It resets at midnight Pacific time. You can keep watching cached Shorts. |
| A Short shows "The creator doesn't allow this Short to play outside YouTube" | That creator turned off embedding. Tap **Open on YouTube** or **Skip**. |
| Error 152/153 inside the player | You opened the file directly or from an unusual address. Use `http://localhost:8000` or your GitHub Pages address. |
| The page is blank or `localhost` won't load | The Python server isn't running. Start it again (Step 2) and keep its window open. |
| Missing Shorts from a channel | Shorts older than your "last N days" setting are hidden, as are any beyond "Shorts per channel". Raise either in ⚙ Settings. A new upload can also take up to 30 minutes to appear; press **Refresh** to check right away. |
| Want to start over | ⚙ Settings → **Sign out** (clears sign-in and cached data), or **Clear cached data**. |
