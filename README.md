# Schedule-Max: The Ultimate Serverless Daily Planner
---

**Schedule-Max** is a robust, high-performance daily planner and routine manager built entirely on **Cloudflare Workers** and **Upstash Redis**. It combines the speed of a Single Page Application (SPA) with the reliability of Server-Side Rendering (SSR).

It features Google OAuth login, offline support (PWA), real-time Telegram notifications, and a beautiful "Optimistic UI" that feels instant even on slow networks.

---

## 🌟 Features

### Core Functionality
*   **Smart Dashboard:**
    *   **"Now" Card:** dynamic card that tells you exactly what you should be doing right now based on the time.
    *   **Today & Tomorrow:** Seamlessly switch views to plan ahead.
    *   **Missed & Completed:** Auto-categorization of tasks based on time deadlines.
*   **Task Management:**
    *   **Routines (Templates):** Set a weekly schedule (e.g., "Gym" every Monday) that auto-populates.
    *   **Custom Tasks:** Add one-off tasks for specific days.
    *   **Swipe Actions:** Swipe tasks right to complete, or utilize the delete button.
*   **Optimistic UI:** The interface updates instantly when you check a box or add a task, syncing with the server in the background. No loading spinners for simple actions.

### Advanced Features
*   **Telegram Notifications:** Get notified via Telegram 1 minute before a task starts and if you miss a task.
*   **Copy Routines:** A "Copy From..." feature allows you to duplicate one day's routine to another (e.g., Copy Monday's schedule to Tuesday).
*   **Time Travel:** Built-in developer tools to simulate different dates/times for testing.
*   **PWA Ready:** Installable on iOS and Android with offline fallback support.
*   **Dark Mode:** automatic theme detection with a manual toggle.

---

## 🛠️ Tech Stack

*   **Runtime:** Cloudflare Workers (Node.js compat).
*   **Database:** Upstash Redis (Serverless).
*   **Frontend:** Vanilla JavaScript & CSS (injected via SSR).
*   **Auth:** Google OAuth 2.0 + JWT (HttpOnly Cookies).
*   **Cron:** Cloudflare Cron Triggers (for notifications).

---

## 🚀 Setup Guide

Follow these steps to deploy your own instance of Schedule-Max.

### 1. Prerequisites
*   A Cloudflare Account.
*   An [Upstash](https://upstash.com/) Account (Free tier is sufficient).
*   Node.js and npm installed.
*   Wrangler CLI installed (`npm install -g wrangler`).

### 2. Create the Worker Project
Initialize a new Cloudflare Worker project:

```bash
npm create cloudflare@latest schedule-max
# Select "Hello World" worker
# Select "No" for TypeScript (unless you want to convert it)
cd schedule-max
npm install @upstash/redis
```

Copy the provided `index.js` code into your project's `src/index.js`.

### 3. Database Setup (Upstash Redis)
1.  Log in to Upstash and create a new Redis database.
2.  Scroll down to the "REST API" section.
3.  Copy the **UPSTASH_REDIS_REST_URL** and **UPSTASH_REDIS_REST_TOKEN**. You will need these for the secrets step.

### 4. Authentication Setup (Google OAuth)
1.  Go to the [Google Cloud Console](https://console.cloud.google.com/).
2.  Create a new project.
3.  Navigate to **APIs & Services > Credentials**.
4.  Click **Create Credentials > OAuth client ID**.
5.  Application Type: **Web application**.
6.  **Authorized redirect URIs:** This is crucial. It must match your worker URL exactly.
    *   Format: `https://<your-worker-name>.<your-subdomain>.workers.dev/auth/google/callback`
    *   *Example:* `https://schedule-max.anuragfr.workers.dev/auth/google/callback`
7.  Copy the **Client ID** and **Client Secret**.

### 5. Notification Setup (Telegram Bot)
1.  Open Telegram and search for **@BotFather**.
2.  Send `/newbot` and follow the instructions to name your bot.
3.  Copy the **HTTP API Token** provided.
4.  (Optional) Create a dummy bot strictly for getting user IDs if you don't want to use the main bot, but the script handles this.

---

## 🔑 Configuration & Secrets

You need to securely store your API keys in Cloudflare. Run the following commands in your terminal:

```bash
# 1. Database Credentials
npx wrangler secret put UPSTASH_REDIS_REST_URL
# Paste your URL when prompted

npx wrangler secret put UPSTASH_REDIS_REST_TOKEN
# Paste your Token when prompted

# 2. Google Auth Credentials
npx wrangler secret put GOOGLE_CLIENT_ID
# Paste your Client ID

npx wrangler secret put GOOGLE_CLIENT_SECRET
# Paste your Client Secret

# 3. Security
npx wrangler secret put JWT_SECRET
# Type a long, random string here (e.g., "super-secret-random-key-123")

# 4. Telegram
npx wrangler secret put TELEGRAM_BOT_TOKEN
# Paste your Bot Token
```

---

## 📝 Code Adjustments (Placeholders)

Before deploying, you **must** modify specific lines in `src/index.js` to match your environment.

### 1. Update the Redirect URI
Search for `handleGoogleLogin` and `handleGoogleCallback` functions (near the bottom of the file). You must change the hardcoded URL to your own Cloudflare Worker URL.

**Find these lines:**
```javascript
authUrl.searchParams.set('redirect_uri', `https://schedule-max.anuragfr.workers.dev/auth/google/callback`);
```
**And:**
```javascript
redirect_uri: `https://schedule-max.anuragfr.workers.dev/auth/google/callback`,
```
**Change to:**
```javascript
// Replace 'your-project' and 'your-name' with your actual worker details
`https://your-project.your-name.workers.dev/auth/google/callback`
```

### 2. Update the App Icon
At the top of the file, find this:
```javascript
const ICON_PNG_B64 = '';
```
You can paste a Base64 string of a PNG image here. This serves as the app icon when installed on a phone. 
You can use an online tool to convert a PNG to Base64.

### 3. Timezone Configuration
The code is currently optimized for **IST (Indian Standard Time, UTC+5:30)**.
If you are in a different timezone, search for `5.5` in the code (it appears in Cron logic and date calculations) and adjust it to your offset (e.g., `-5` for EST).

---

## ⚙️ Deployment & Automation

### 1. Configure `wrangler.toml`
Open your `wrangler.toml` file and ensure you enable the Cron Trigger for notifications. Add this to the file:

```toml
name = "schedule-max"
main = "src/index.js"
compatibility_date = "2023-10-30"

# Run the cron job every minute to check for tasks
[triggers]
crons = ["* * * * *"]
```
### if you have a `wrangler.jsonc`
then 
```jsonc
{
	"$schema": "node_modules/wrangler/config-schema.json",
	"name": "schedule-max",
	"main": "src/index.js",
	"compatibility_date": "2025-11-15",
	"observability": {
		"enabled": true
	},
  "triggers": {
    "crons": [
      "* * * * *",
    ]
  },
}
```


### 2. Deploy
Run the deployment command:

```bash
npm run deploy
# or
npx wrangler deploy
```

---

## 📱 How to Use

1.  **Login:** Open your deployed URL and sign in with Google.
2.  **Onboarding:** You will see a hand-drawn arrow guide. Follow it to set up your permanent routines.
3.  **Telegram Setup:**
    *   Click the `...` menu -> **Set Telegram Alerts**.
    *   It will ask for your Chat ID. If you don't know it, click the link provided in the modal to message your bot, which will reply with your ID.
    *   Save the ID.
4.  **Install PWA:**
    *   **iOS:** Share -> Add to Home Screen.
    *   **Android:** Chrome Menu -> Install App.

## 🤝 Contributing
Feel free to fork this repository and submit pull requests. Areas for improvement:
*   User-selectable Timezones in the UI.
*   Drag-and-drop task reordering.
*   More granular notification settings.

## 📄 License
MIT License. Free to use and modify.
