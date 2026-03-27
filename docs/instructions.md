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
{
	/* Inflight: only activates for a specific staging URL when shared via `inflight share` */
}
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async />;
```

Add inside `<body>`, after `{children}`.

**Next.js (Pages Router)** — `pages/_document.tsx`:

```tsx
{
	/* Inflight: only activates for a specific staging URL when shared via `inflight share` */
}
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async />;
```

Add inside `<body>`, after `<NextScript />`.

**Vite / Create React App** — `index.html` at project root:

```html
<!-- Inflight: only activates for a specific staging URL when shared via `inflight share` -->
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

Add inside `<body>`, before `</body>`.

**Remix** — `app/root.tsx`:

```tsx
{
	/* Inflight: only activates for a specific staging URL when shared via `inflight share` */
}
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async />;
```

Add inside `<body>`, after `<Scripts />`.

**SvelteKit** — `src/app.html`:

```html
<!-- Inflight: only activates for a specific staging URL when shared via `inflight share` -->
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
<!-- Inflight: only activates for a specific staging URL when shared via `inflight share` -->
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

Add before `</body>`.

**Plain HTML:**

```html
<!-- Inflight: only activates for a specific staging URL when shared via `inflight share` -->
<script src="https://www.inflight.co/widget.js" data-workspace="WIDGET_ID" async></script>
```

If you cannot determine the framework, ask the user where their root HTML layout lives.

### Step 5: Commit and push

```bash
git add -A
git commit -m "Add Inflight feedback widget script tag"
git push
```

### Step 6: Share a staging URL

Ask the user: "Do you deploy with Vercel, or do you want to paste a staging URL?"

**If Vercel:**

```bash
inflight vercel teams --json
```

Ask which team. Then:

```bash
inflight vercel projects --json --team=TEAM_ID
```

Ask which project. Then:

```bash
inflight vercel deployments --json --team=TEAM_ID --project=PROJECT_ID
```

Present the deployments and ask which one. **Important:** Only suggest deployments created AFTER the commit from Step 5, since older deployments won't have the widget script.

**If manual URL:** Ask the user for their staging URL.

Then share:

```bash
inflight share --url=STAGING_URL --workspace=WORKSPACE_ID --json
```

This creates the version and opens the staging URL with the feedback widget active.

---

## Share Flow (repeated use)

When the user wants to share a new version:

### Step 1: Check authentication

```bash
inflight workspaces --json
```

If this fails, run `inflight login` first.

### Step 2: Get staging URL

Ask: "Vercel or paste a URL?"

Use the same Vercel data commands as above, or ask for a manual URL.

### Step 3: Share

```bash
inflight share --url=STAGING_URL --workspace=WORKSPACE_ID --json
```

Run `inflight workspaces --json` to get the workspace ID. If `active` is set, use it directly without asking the user.

---

## CLI Commands Reference

| Command                                                     | Purpose                         | Output                           |
| ----------------------------------------------------------- | ------------------------------- | -------------------------------- |
| `inflight login`                                            | Authenticate via browser        | Opens browser, polls for session |
| `inflight workspaces --json`                                | List workspaces + active ID     | JSON object                      |
| `inflight workspaces --set=ID`                              | Set the active workspace        | JSON confirmation                |
| `inflight share --url=URL --workspace=ID --json`            | Share a staging URL             | JSON result                      |
| `inflight vercel teams --json`                              | List Vercel teams               | JSON array                       |
| `inflight vercel projects --json --team=ID`                 | List projects for a team        | JSON array                       |
| `inflight vercel deployments --json --team=ID --project=ID` | List recent deployments         | JSON array                       |
