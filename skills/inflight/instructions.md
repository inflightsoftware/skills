# Inflight

Inflight lets teams collect design feedback directly on staging URLs. You will help the user install the CLI, authenticate, add the widget script, and share versions.

## Prerequisites

- Node.js 18+
- An Inflight account — sign up at https://www.inflight.co

## Step 1: Install or update the CLI

```bash
inflight --version
```

If the command is not found, install it:

```bash
npm install -g inflight-cli
```

If already installed, check for updates:

```bash
npm outdated -g inflight-cli
```

If a newer version is available, update:

```bash
npm install -g inflight-cli
```

## Step 2: Authenticate

```bash
inflight login
```

- If already authenticated, the CLI confirms the session and exits.
- If not, it opens a browser for sign-in. Wait for the CLI to confirm authentication before proceeding.
- On success it prints: `✓ Logged in successfully`.
- Credentials are managed automatically by the CLI.

To log out later: `inflight logout`

## Step 3: Add the widget script tag

**IMPORTANT: The widget script must be deployed to the staging environment BEFORE running `inflight share`.** If it's not there, the feedback UI won't appear.

First, search the project for any existing `inflight.co/widget.js` reference. If it already exists, skip to Step 4.

Add this script tag to the site's HTML, just before `</body>`:

```html
<script src="https://www.inflight.co/widget.js" async></script>
```

### Framework-specific placement

Detect the project's framework by checking `package.json` dependencies and file structure, then add the script in the correct location:

**Next.js (App Router)** — if `app/layout.tsx` exists:
```tsx
// app/layout.tsx — add inside <body>, after {children}
<script src="https://www.inflight.co/widget.js" async />
```

**Next.js (Pages Router)** — if `pages/_document.tsx` exists:
```tsx
// pages/_document.tsx — add inside <body>, after <NextScript />
<script src="https://www.inflight.co/widget.js" async />
```

**Vite / Create React App** — if `index.html` exists at project root:
```html
<!-- index.html — add inside <body>, after the main script -->
<script src="https://www.inflight.co/widget.js" async></script>
```

**Remix** — if `app/root.tsx` exists:
```tsx
// app/root.tsx — add inside <body> of the root component
<script src="https://www.inflight.co/widget.js" async />
```

**Nuxt 3** — if `nuxt.config.ts` exists:
```ts
// nuxt.config.ts — add to the app.head.script array
export default defineNuxtConfig({
  app: {
    head: {
      script: [{ src: "https://www.inflight.co/widget.js", async: true }],
    },
  },
});
```

**SvelteKit** — if `src/app.html` exists:
```html
<!-- src/app.html — add inside <body>, after %sveltekit.body% -->
<script src="https://www.inflight.co/widget.js" async></script>
```

**Astro** — if `src/layouts/` exists:
```astro
<!-- In the base layout, add inside <body> before </body> -->
<script src="https://www.inflight.co/widget.js" async></script>
```

**Plain HTML:**
```html
<!-- Add before </body> -->
<script src="https://www.inflight.co/widget.js" async></script>
```

If you cannot determine the framework, ask the user where their root HTML layout lives.

### After adding the script

Commit the change and tell the user to deploy to staging before proceeding:

```bash
git add .
git commit -m "Add Inflight feedback widget"
git push
```

The user must push and wait for the staging deployment to complete before Step 4 will work.

## Step 4: Share a version

```bash
inflight share
```

**This is a fully interactive command. Do NOT try to pass arguments or pipe input — run it and let the user walk through the prompts in their terminal.**

The command walks the user through:

1. **Workspace selection** — prompts the user to pick a workspace (remembered for future runs).
2. **Staging URL** — the user chooses how to provide their staging URL:
   - **Vercel** — auto-detects deployments and branch previews. May prompt for Vercel login and project linking if needed.
   - **Paste a URL** — accepts any URL (e.g., `my-branch.vercel.app`).
3. **Creates the version** — registers the staging URL with Inflight.
4. **Opens the browser** — launches the staging URL with the feedback widget active.

On success the CLI prints:
```
✓ Inflight added to your staging URL — opening in browser...
```

## Switching workspaces

If the user needs to change which Inflight workspace this project is linked to:

```bash
inflight workspace
```

## CLI Command Reference

| Command              | What it does                                      |
| -------------------- | ------------------------------------------------- |
| `inflight login`     | Authenticate with your Inflight account           |
| `inflight logout`    | Disconnect your Inflight account                  |
| `inflight share`     | Share a staging URL for design feedback            |
| `inflight workspace` | Switch the active workspace for this project      |

## How the widget works

- The widget checks if an Inflight version exists for the current hostname.
- If a version exists, it loads the feedback UI (comments, polls, vibe checks).
- If no version exists, it does nothing — zero overhead.
- The widget is fully isolated — it won't affect the host site's styles or events.
- Versions are matched by hostname (e.g., `my-app.vercel.app`), not by path.

## Troubleshooting

**Widget not appearing on staging:**
1. View source on the staging URL — confirm the `widget.js` script tag is in the HTML.
2. Check the hostname matches — the staging URL is normalized to just the hostname.
3. Check the browser console for `[Inflight]` log messages.

**"Not logged in" error:**
Run `inflight login` to re-authenticate. Sessions can expire.

**"No workspaces found":**
The user needs to create a workspace at https://www.inflight.co first, then retry.

**Vercel not detecting deployments:**
Make sure the user is logged into the Vercel CLI (`vercel login` or `npx vercel login`).
