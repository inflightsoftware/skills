---
name: inflight
version: 1.0.0
description: "Inflight: Add the feedback widget to any staging site and share versions for design review."
---

# Inflight — AI Agent Skill

Inflight lets teams collect design feedback directly on staging URLs. This skill teaches you how to install the widget, authenticate, and share versions using the `inflight` CLI.

## Prerequisites

- **Node.js 18+**
- **npm** (for global install)
- An Inflight account at https://inflight.co

## Step 1: Install the CLI

```bash
npm install -g inflight-cli
```

Verify:

```bash
inflight --version
```

## Step 2: Authenticate

```bash
inflight login
```

This opens a browser window for the user to sign in. Credentials are stored locally and securely.

If already authenticated, `inflight login` confirms the session and exits.

## Step 3: Add the Widget Script Tag

**Before sharing a version, the staging site must include the Inflight widget loader.** This is a lightweight script (~30 lines) that checks if a version exists for the current hostname and loads the feedback widget if so. It does nothing if no version exists — zero overhead.

Add this `<script>` tag to the site's HTML, just before `</body>`:

```html
<script src="https://inflight.co/widget.js" async></script>
```

### Where to add it

The exact location depends on the framework:

**Next.js (App Router):**
```tsx
// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
        <script src="https://inflight.co/widget.js" async />
      </body>
    </html>
  );
}
```

**Next.js (Pages Router):**
```tsx
// pages/_document.tsx
import { Html, Head, Main, NextScript } from "next/document";

export default function Document() {
  return (
    <Html>
      <Head />
      <body>
        <Main />
        <NextScript />
        <script src="https://inflight.co/widget.js" async />
      </body>
    </Html>
  );
}
```

**Vite / Create React App:**
```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.tsx"></script>
  <script src="https://inflight.co/widget.js" async></script>
</body>
```

**Remix:**
```tsx
// app/root.tsx — inside the <body> of your root component
<script src="https://inflight.co/widget.js" async />
```

**Nuxt 3:**
```ts
// nuxt.config.ts
export default defineNuxtConfig({
  app: {
    head: {
      script: [{ src: "https://inflight.co/widget.js", async: true }],
    },
  },
});
```

**Plain HTML:**
```html
<script src="https://inflight.co/widget.js" async></script>
```

### Important notes

- The script tag must be deployed to the staging environment **before** running `inflight share`
- The widget matches versions by **hostname** (e.g., `my-feature.vercel.app`)
- The widget is fully isolated — it won't affect the host site's styles or events
- If no version exists for the current hostname, the widget does nothing — zero overhead

### After adding the script tag

Commit and deploy the change to your staging environment:

```bash
git add .
git commit -m "Add Inflight feedback widget"
git push
```

Wait for the staging deployment to complete before proceeding to Step 4.

## Step 4: Share a Version

Once the widget script is deployed to staging, share a version:

```bash
inflight share
```

This interactive command:

1. **Validates auth** — checks your stored API key
2. **Resolves workspace** — reads `.inflight/workspace.json`, or prompts you to select one (stored for future runs)
3. **Detects git info** — branch, commit SHA, commit message, remote URL, diff
4. **Prompts for staging URL** — choose a provider:
   - **Vercel** — auto-detects deployments from the Vercel API
   - **Manual** — paste any URL (e.g., `https://my-feature.vercel.app`)
5. **Creates the version** — sends project + version to the Inflight API
6. **Opens the staging URL** in your browser with the widget active

### Output

On success, the CLI prints the Inflight version URL:

```
┌ Your Inflight version
│ https://inflight.co/v/v_abc123
└
✓ Done — opening staging URL...
```

### Config files created

- `.inflight/workspace.json` — links this repo to a workspace (auto-added to `.gitignore`)

## Complete Workflow Example

```bash
# 1. Install
npm install -g inflight-cli

# 2. Authenticate
inflight login

# 3. Add widget script to your app's HTML (see Step 3 above)
# ... edit index.html / layout.tsx / etc.

# 4. Commit & deploy to staging
git add .
git commit -m "Add Inflight feedback widget"
git push
# Wait for staging deploy to finish...

# 5. Share the version
inflight share
# Select workspace → pick Vercel or paste staging URL → done!
```

## Troubleshooting

### Widget not appearing on staging

1. **Check the script tag is deployed** — view source on the staging URL, confirm `widget.js` is present
2. **Check the hostname matches** — the CLI normalizes the staging URL to just the hostname. `https://my-app.vercel.app/some/path` becomes `my-app.vercel.app`
3. **Check browser console** — look for `[Inflight]` log messages

### "Not logged in" error

Run `inflight login` to re-authenticate. Sessions can expire.

### "No workspaces found"

Create a workspace at https://inflight.co first, then retry `inflight share`.
