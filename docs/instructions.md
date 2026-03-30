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
inflight workspace --json
```

Returns:

```json
{ "active": "ws_abc", "workspaces": [{ "id": "ws_abc", "name": "My Workspace", "widgetId": "wgt_123" }] }
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

### Step 6: Get staging URL and share

Ask the user: "Where is your staging site deployed? (e.g., Vercel, Netlify, Railway, or paste a URL directly)"

**If Vercel:** Get deployments and branch alias in one call:

```bash
inflight vercel deployments
```

This auto-detects the Vercel project from the git repo and current branch. Returns JSON:

```json
{
	"branchAlias": { "url": "my-branch.vercel.app", "state": "READY", "branch": "main" },
	"deployments": [
		{
			"url": "my-project-abc123.vercel.app",
			"state": "READY",
			"branch": "main",
			"commitSha": "abc1234",
			"commitMessage": "feat: add auth",
			"createdAt": 1234567890
		}
	]
}
```

**Present the options to the user:**

- If `branchAlias` exists, recommend it — it's a stable URL that auto-updates with each push, no need to re-share
- Also show recent deployments as alternatives
- **Do not assume** — always let the user choose

**If auto-detection fails** (error with `vercel_not_configured`), list all projects:

```bash
inflight vercel projects
```

Ask the user which project, then retry with explicit IDs:

```bash
inflight vercel deployments --team=TEAM_ID --project=PROJECT_ID
```

**If manual URL:** Ask the user for their staging URL.

**Important:** If the widget script tag was just added in Step 5, only suggest deployments created AFTER that commit was pushed. Older deployments won't have the widget.

Then share:

```bash
inflight share --url=STAGING_URL --json
```

This creates the version with the feedback widget active.

---

## Share Flow (repeated use)

When the user wants to share a new version:

### Step 1: Get staging URL

Ask: "Where is your staging site deployed? (e.g., Vercel, Netlify, Railway, or paste a URL directly)"

**If Vercel:**

```bash
inflight vercel deployments
```

This auto-detects the Vercel project and returns the branch alias + recent deployments.

**Present the options to the user:**

- If `branchAlias` exists, recommend it (auto-updates with each push)
- Also show recent deployments as alternatives
- **Do not assume** — always let the user choose

**If auto-detection fails**, fall back to `inflight vercel projects`, ask user to pick, then `inflight vercel deployments --team=TEAM_ID --project=PROJECT_ID`.

**If manual URL:** Ask the user for their staging URL.

### Step 2: Share

```bash
inflight share --url=STAGING_URL --json
```

---

## Agent Output Rules

**After completing the setup flow:**

- Confirm setup is complete and let the user know they can say "share to Inflight" in future sessions to share new staging URLs.
- Do NOT summarize what was done (CLI version, auth status, workspace, widget location, etc.). The user saw each step happen.

**After completing the share flow:**

- Log the staging URL. The CLI auto-opens it in the browser.
- Do NOT add any other commentary or summary.

**General:**

- Keep output minimal. The CLI handles user-facing messaging — don't duplicate it.
- Never recap completed steps at the end of a flow.

---

## CLI Commands Reference

| Command                                              | Purpose                                  | Output        |
| ---------------------------------------------------- | ---------------------------------------- | ------------- |
| `inflight login`                                     | Authenticate via browser                 | Opens browser |
| `inflight workspace --json`                          | List workspaces + active ID              | JSON object   |
| `inflight workspace --set=ID`                        | Set the active workspace                 | JSON confirm  |
| `inflight share --url=URL --json`                    | Share a staging URL                      | JSON result   |
| `inflight vercel deployments`                        | Deployments + branch alias (auto-detect) | JSON object   |
| `inflight vercel deployments --team=ID --project=ID` | Deployments (explicit project)           | JSON object   |
| `inflight vercel projects`                           | List all Vercel projects across teams    | JSON array    |
