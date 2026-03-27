# Inflight Agent Instructions

Inflight lets teams collect design feedback directly on staging URLs. You will install the CLI, add the widget script, authenticate, and share the staging URL — all from this agent session.

## Prerequisites

- Node.js 18+
- An Inflight account — sign up at https://www.inflight.co

---

## Setup Flow (first time in a project)

### Step 1: Install the CLI

Check if already installed:

```bash
inflight --version
```

If not found, install it:

```bash
npm install -g inflight-cli
```

### Step 2: Authenticate

```bash
inflight login
```

This opens a browser for sign-in. Wait for the success message before continuing.

If it says "You're already logged in", proceed to Step 3.

### Step 3: Get the workspace and widget ID

```bash
inflight workspaces --json
```

Returns:

```json
{"active": "ws_abc", "workspaces": [{"id": "ws_abc", "name": "My Workspace", "widgetId": "wgt_123"}]}
```

If `active` is set, use that workspace. If `active` is null and there are multiple workspaces, ask the user which one to use. Save the `id` and `widgetId` for the next steps.

### Step 4: Add the widget script tag

**IMPORTANT:** The widget script must be deployed to staging BEFORE `inflight share` will work. If it's not there, the feedback UI won't appear.

First, search the project for any existing `inflight.co/widget.js` reference. If found, skip to Step 5.

Detect the framework from `package.json` and file structure, then add the script in the correct location. Replace `WIDGET_ID` with the actual `widgetId` from Step 3.

**Next.js (App Router)** — `app/layout.tsx` or `src/app/layout.tsx`:

```tsx
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async />
```

Add inside `<body>`, after `{children}`.

**Next.js (Pages Router)** — `pages/_document.tsx`:

```tsx
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async />
```

Add inside `<body>`, after `<NextScript />`.

**Vite / Create React App** — `index.html` at project root:

```html
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

Add inside `<body>`, before `</body>`.

**Remix** — `app/root.tsx`:

```tsx
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async />
```

Add inside `<body>`, after `<Scripts />`.

**SvelteKit** — `src/app.html`:

```html
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

Add after `%sveltekit.body%`.

**Nuxt 3** — `nuxt.config.ts`:

```ts
export default defineNuxtConfig({
	app: {
		head: {
			script: [{ src: "https://www.inflight.co/widget.js", "data-workspace": "WIDGET_ID", async: true }],
		},
	},
});
```

**Astro** — base layout in `src/layouts/`:

```html
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

Add before `</body>`.

**Plain HTML:**

```html
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

If you cannot determine the framework, ask the user where their root HTML layout lives.

### Step 5: Commit and push

Only commit the file you modified with the widget script — do not stage unrelated changes.

```bash
git add <file-you-modified>
git commit -m "Add Inflight feedback widget script tag"
git push
```

Tell the user their changes need to be deployed before the widget will show. They can proceed to Step 6 immediately — they just need to pick a deployment that includes this push.

### Step 6: Set up Vercel (if using Vercel)

Ask the user: "Do you deploy with Vercel, or do you want to paste a staging URL?"

**If Vercel:** First, get the team and project IDs:

```bash
inflight vercel teams
```

Ask which team. Then:

```bash
inflight vercel projects --team=TEAM_ID
```

Ask which project. Then save the Vercel configuration:

```bash
inflight vercel --team=TEAM_ID --project=PROJECT_ID --json
```

This saves the Vercel project globally so future `inflight share` and `inflight vercel deployments` commands use it automatically. To change the project later, run this command again with different IDs.

### Step 7: Share a staging URL

**If Vercel:** Try the branch preview URL first (recommended — it auto-updates with each push):

```bash
inflight vercel branch-url --branch=BRANCH_NAME
```

Returns `{"url": "my-branch.my-app.vercel.app", "branch": "main"}` or `{"url": null}` if no deployment exists for that branch.

If no branch URL is available, fall back to picking from recent deployments:

```bash
inflight vercel deployments
```

**Important:** Only suggest deployments created AFTER the commit from Step 5, since older deployments won't have the widget script.

**If manual URL:** Ask the user for their staging URL.

Then share:

```bash
inflight share --url=STAGING_URL --workspace=WORKSPACE_ID --json
```

This creates the version with the feedback widget active.

---

## Share Flow (repeated use)

When the user wants to share a new version:

### Step 1: Check for CLI updates

```bash
inflight --version
npm install -g inflight-cli@latest
```

### Step 2: Check authentication and get workspace

```bash
inflight workspaces --json
```

If this fails with an auth error, run `inflight login` first.

If `active` is set, use that workspace ID directly. If `active` is null and there are multiple workspaces, ask the user which one, then set it:

```bash
inflight workspaces --set=WORKSPACE_ID
```

### Step 3: Get staging URL

Ask: "Vercel or paste a URL?"

**If Vercel:** Try the branch preview URL first (recommended):

```bash
inflight vercel branch-url --branch=BRANCH_NAME
```

If no branch URL, fall back to deployments:

```bash
inflight vercel deployments
```

These commands use the saved Vercel project configuration. If the user wants to change the Vercel project:

```bash
inflight vercel --team=NEW_TEAM_ID --project=NEW_PROJECT_ID --json
```

**If manual URL:** Ask the user for their staging URL.

**Important:** If the widget script tag was recently added, only suggest deployments created after that change was pushed. Older deployments won't have the widget.

### Step 4: Share

```bash
inflight share --url=STAGING_URL --workspace=WORKSPACE_ID --json
```

---

## CLI Commands Reference

| Command                                                | Purpose                                | Output                           |
| ------------------------------------------------------ | -------------------------------------- | -------------------------------- |
| `inflight login`                                       | Authenticate via browser               | Opens browser, polls for session |
| `inflight workspaces --json`                           | List workspaces + active ID            | JSON object                      |
| `inflight workspaces --set=ID`                         | Set the active workspace               | JSON confirmation                |
| `inflight share --url=URL --workspace=ID --json`       | Share a staging URL                    | JSON result                      |
| `inflight vercel --team=ID --project=ID --json`        | Save Vercel project configuration      | JSON confirmation                |
| `inflight vercel teams`                                | List Vercel teams                      | JSON array (always)              |
| `inflight vercel projects --team=ID`                   | List projects for a team               | JSON array (always)              |
| `inflight vercel branch-url --branch=NAME`             | Get stable branch preview URL          | JSON object (always)             |
| `inflight vercel deployments`                          | List recent deployments (saved config) | JSON array (always)              |
| `inflight vercel deployments --team=ID --project=ID`   | List deployments (explicit IDs)        | JSON array (always)              |
| `inflight vercel deployments --branch=NAME`            | List deployments filtered by branch    | JSON array (always)              |
| `inflight vercel deployments --limit=N`                | Limit number of deployments (default 10) | JSON array (always)            |
