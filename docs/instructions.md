# Inflight

Inflight lets teams collect design feedback directly on staging URLs. You will install the CLI, add the widget script to the project, help the user deploy, authenticate, and then tell the user to run the final sharing command in their terminal.

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

## Step 2: Add the widget script tag

**IMPORTANT: The widget script must be deployed to the staging environment BEFORE the user runs `inflight share`.** If it's not there, the feedback UI won't appear.

First, search the project for any existing `inflight.co/widget.js` reference. If it already exists, skip to Step 3.

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

## Step 3: Deploy the widget to staging

The widget script tag must be live on the staging environment before `inflight share` will work. Commit and push the change:

```bash
git add .
git commit -m "Add Inflight feedback widget"
git push
```

**Tell the user they need to wait for their staging deployment to finish before running `inflight share`.** This is critical — the widget won't appear if the script tag isn't deployed yet.

While the deployment is in progress, proceed to Step 4.

## Step 4: Authenticate

Run this command:

```bash
inflight login
```

This opens a browser for sign-in and polls until complete. Wait for it to print `✓ Logged in successfully` before moving on.

If it prints an email and `✓ You're already logged in`, the user is already authenticated — proceed to Step 5.

## Step 5: Share a version

**Do NOT run this command. Tell the user to run it in their terminal.**

This command requires an interactive terminal and will fail if you try to run it. Instruct the user:

> Run `inflight share` in your terminal. It will walk you through selecting a workspace and staging URL, then open your site with the feedback widget active.
