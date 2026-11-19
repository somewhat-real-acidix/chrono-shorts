# Chrono: Shorts from your subscriptions, newest first

A one-page web app. Sign in with Google, and it plays Shorts from every channel you subscribe to in strict chronological order. It works on your PC and on your iPhone (added to the Home Screen it opens like an app). Everything is free.

**What's in this folder**

| File | What it is |
|---|---|
| `index.html` | The whole app |
| `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Home-screen icon and app name for phones |
| `README.md` | This guide |

**The big picture.** You'll do four things, about 30–40 minutes in total:

1. Tell Google about your app: a free "OAuth client" in Google Cloud.
2. Try it on your PC, at an address only your PC can see.
3. Put it on the internet for free with GitHub Pages, so your phone can reach it.
4. Add it to your iPhone's Home Screen.

> **Why not just double-click `index.html`?** Google sign-in and the YouTube player both refuse to work on pages opened as files (`file://…`). The app has to be served from a web address: `http://localhost:8000` on your PC for testing, or your GitHub Pages address for real use.

---

## Step 1: Set up Google sign-in (Google Cloud)

You can reuse the Google Cloud project you made for the Chrome extension's API key.

### 1a. Make sure the YouTube API is enabled
1. Go to <https://console.cloud.google.com/> and pick your project in the project picker at the top.
2. Open **APIs & Services → Library**, search for **YouTube Data API v3**, and click it.
3. If you see **Enable**, click it. If it says **Manage**, it's already on.

### 1b. Set up the consent screen (what you see when you sign in)
In the left menu, open **Google Auth Platform**. Older consoles call it **APIs & Services → OAuth consent screen**.

1. If it says the platform isn't configured yet, click **Get started**:
   - **App name:** `Chrono`
   - **User support email:** your Gmail address
   - **Audience:** choose **External**
   - **Contact email:** your Gmail address
   - Agree to the policy, then click **Create**.
2. Open **Audience**:
   - Leave **Publishing status** set to **Testing**.
   - Under **Test users**, click **Add users** and enter the Google account you use on YouTube. **Don't skip this.** If you do, Google will say "Access blocked" when you try to sign in.
3. Optional: open **Data Access → Add or remove scopes**, search for `youtube.readonly`, tick it, and click **Update** then **Save**. The app requests this scope anyway, so this step only makes the consent screen tidier.

### 1c. Create the OAuth client
1. Open **Google Auth Platform → Clients**. Older consoles call it **APIs & Services → Credentials → Create credentials → OAuth client ID**.
2. Click **Create client**:
   - **Application type:** **Web application**
   - **Name:** `Chrono web`
3. Under **Authorized JavaScript origins**, click **Add URI** and enter:
   - `http://localhost:8000`
4. Under **Authorized redirect URIs**, click **Add URI** and enter:
   - `http://localhost:8000/` (with the slash at the end)
5. Click **Create**. Copy the **Client ID**, which looks like `1234567890-abcdefg.apps.googleusercontent.com`. You'll paste it into the app.
   - Ignore the client secret. This app doesn't use it.

You'll add your GitHub Pages address to the same client in Step 3. Changes to these lists can take a few minutes, occasionally longer, to take effect.

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
Go back to **Google Auth Platform → Clients → Chrono web** and add:
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

| Action | Phone | Computer |
|---|---|---|
| Next (older) Short | Swipe up | ↓, `j`, scroll down, or ▼ |
| Previous (newer) Short | Swipe down | ↑, `k`, scroll up, or ▲ |
| Pause / play | Tap the video | Click the video, or Space |
| Sound on/off | 🔇 badge | `m` or 🔊 |
| Back to newest | | `Home` key |
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
| **"Access blocked: Chrono has not completed the Google verification process"** | Add your Google account under **Audience → Test users** (Step 1b). |
| **"Google hasn't verified this app"** | Expected. Click **Continue** or **Advanced → Go to Chrono**. |
| "The YouTube Data API v3 isn't enabled…" | Do Step 1a in the **same** project as your OAuth client. |
| "Daily YouTube API quota used up" | It resets at midnight Pacific time. You can keep watching cached Shorts. |
| A Short shows "The creator doesn't allow this Short to play outside YouTube" | That creator turned off embedding. Tap **Open on YouTube** or **Skip**. |
| Error 152/153 inside the player | You opened the file directly or from an unusual address. Use `http://localhost:8000` or your GitHub Pages address. |
| The page is blank or `localhost` won't load | The Python server isn't running. Start it again (Step 2) and keep its window open. |
| Missing Shorts from a channel | Shorts older than your "last N days" setting are hidden, as are any beyond "Shorts per channel". Raise either in ⚙ Settings. A new upload can also take up to 30 minutes to appear; press **Refresh** to check right away. |
| Want to start over | ⚙ Settings → **Sign out** (clears sign-in and cached data), or **Clear cached data**. |
