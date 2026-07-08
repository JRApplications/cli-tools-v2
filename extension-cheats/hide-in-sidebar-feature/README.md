# Implementing the "Hide in Sidebar" Feature for Dashboard Pages

This guide walks through patching the `@wix/astro` package locally to support hiding specific dashboard pages from the sidebar.

> **Disclaimer:** This is not an officially documented feature. Use at your own risk, and expect it to break on package updates so may need to refer to these steps again in the future.

---

## Step 1 — Patch the build integration

Open:

```
node_modules/@wix/astro/build/integration/index.mjs
```

Go to around **line 1256**. You should see something like:

```js
case "BackofficePage": return {
  compId: extension.options.id,
  compName: extension.options.title,
  compType: "BACK_OFFICE_PAGE",
  compData: { backOfficePage: {
    hostingPlatform: "BUSINESS_MANAGER",
    iframeUrl,
    routePath: extension.options.routePath,
    title: extension.options.title
  } },
  createdBy: extension.options.createdBy?.author,
  createdByVersion: extension.options.createdBy?.version
};
```

Add a `hideInSideBar` field inside `compData.backOfficePage`:

```js
case "BackofficePage": return {
  compId: extension.options.id,
  compName: extension.options.title,
  compType: "BACK_OFFICE_PAGE",
  compData: { backOfficePage: {
    hostingPlatform: "BUSINESS_MANAGER",
    iframeUrl,
    routePath: extension.options.routePath,
    title: extension.options.title,
    hideInSideBar: extension.options.hideInSidebar
  } },
  createdBy: extension.options.createdBy?.author,
  createdByVersion: extension.options.createdBy?.version
};
```

**Optional — make it opt-in per page:** if you only want to set this for a subset of dashboard pages (and default the rest to visible), use:

```js
hideInSideBar: extension.options?.hideInSidebar || false
```

This way you don't need to add `hideInSidebar` to every `.extension.ts` file — only the ones you want hidden (see the note at the end of Step 3).

---

## Step 2 — Add the type definition

Open:

```
node_modules/@wix/astro/build/integration/builders-DJKcjc2W.d.mts
```

Find the dashboard page options interface — it's typically named `Options$1`. Add `hideInSidebar` as a boolean:

```ts
interface Options$1 {
  id: string;
  component: string;
  createdBy?: CreatedByConfig;
  routePath: string;
  title: string;
  hideInSidebar: boolean;
}
```

---

## Step 3 — Set the flag on your dashboard page

In your dashboard page's `.extension.ts` file, add the `hideInSidebar` param:

```ts
import { extensions } from '@wix/astro/builders';

export default extensions.dashboardPage({
  id: '2352766d-6da4-4bfa-8a49-f0bc46322236',
  title: 'Dashboard Page Two',
  routePath: 'dashboard-page-two',
  component: './extensions/dashboard/pages/dashboard-page-two/dashboard-page-two.tsx',
  hideInSidebar: false,
  createdBy: {
    author: "ONE_CLI",
  },
});
```

> **Note:** If you did **not** add the `|| false` fallback in Step 1, you must add `hideInSidebar` to *every* dashboard page's `.extension.ts` file — otherwise pages without the field may behave unpredictably.

---

## Step 4 — Build and verify

```bash
npm run build
```

Refresh the page — dashboard pages with `hideInSidebar: true` should no longer appear in the sidebar.

If you're not running in dev mode, you'll also need to release a new version:

```bash
npm run release
```
