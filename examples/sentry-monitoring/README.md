# Integrate Sentry Monitoring in Your Astro CLI App

## Overview

This integration gives you two things:

1. **Automatic request logging** — every request/response pair that hits your Astro middleware is logged to Sentry, tagged with a request ID, Wix `siteId`, and `instanceId`, so you can trace what happened around any given call.
2. **Manual error capture** — a `captureError()` helper you call from inside your API routes' `catch` blocks to send exceptions to Sentry and get back a request ID to return to the client.

Both use the *same* Sentry client under the hood (initialized once in `src/monitoring/sentry.ts`), so request logs and manually-captured errors sharing the same request ID will show up correlated in Sentry.

## Prerequisites

- A Sentry project and its DSN (`Settings → Client Keys (DSN)` in Sentry).
- The Sentry browser SDK installed:
  ```bash
  npm install @sentry/browser
  ```
- Astro's typed env support requires Astro **5.0+**.
- Declare `SENTRY_DSN` in `astro.config.mjs` using [`astro:env`](https://docs.astro.build/en/guides/environment-variables/#type-safe-environment-variables):
- Refer to Wix Official documentation for env support. [Here](https://dev.wix.com/docs/build-apps/develop-your-app/develop-an-app-with-the-cli/project-development/environment-variables/manage-environment-variables-in-the-cli)
  ```javascript
  // astro.config.mjs
  import { defineConfig, envField } from 'astro/config';

  export default defineConfig({
      // ...your existing config
      env: {
          schema: {
              SENTRY_DSN: envField.string({
                  context: 'server',
                  access: 'secret', // use 'public' if you also need this client-side
              }),
          },
      },
  });
  ```
- Then set the actual value in your `.env` (never hardcode it in source):
  ```
  # .env
  SENTRY_DSN=https://xxxxxxxx@xxxxx.ingest.sentry.io/xxxxxxx
  ```

## Files Needed

| File | Path |
|---|---|
| `middleware.ts` | `src/middleware.ts` |
| `sentry.ts` | `src/monitoring/sentry.ts` |
| `reportError.ts` | `src/pages/api/reportError.ts` |

`sentry.ts` is the single source of truth: it's the only place `Sentry.init()` is called, and it exports both `captureError` and `captureRequest`. `reportError.ts` is a thin re-export that exists purely so API routes elsewhere in `src/pages/api/` have a short, conventional import path (`../reportError`) — it does **not** call `Sentry.init()` again. Calling `init()` more than once in the same runtime can cause duplicate reporting, so keep it that way if you customize this further.

## Code

### `src/monitoring/sentry.ts`

```typescript
import * as Sentry from '@sentry/browser';
import { SENTRY_DSN } from 'astro:env/server';

// Single, centralized Sentry.init() call. Do NOT call Sentry.init() anywhere
// else in the app — calling it more than once can cause duplicate event
// reporting and unpredictable scope behavior.
Sentry.init({
    dsn: SENTRY_DSN, // defined in astro.config.mjs + .env — never hardcode the DSN
    environment: import.meta.env.MODE === 'production' ? 'production' : 'development',
    tracesSampleRate: 0, // 0 = no performance tracing, just error/message capture
    defaultIntegrations: false, // minimal setup, no automatic error capture
});

interface CaptureOptions {
    requestId?: string;
    siteId?: string;
    instanceId?: string;
}

const captureWithScope = (message: string, level: 'error' | 'warning' | 'info', options?: CaptureOptions) => {
    Sentry.withScope((scope) => {
        if (options?.requestId) {
            // TODO: customise this tag name to match your app (e.g. 'x-myApp-request-id')
            scope.setTag('x-myApp-request-id', options.requestId);
        }
        Sentry.captureMessage(message, level);
    });
};

/**
 * Manually capture an error or message from anywhere in the app
 * (e.g. inside an API route's catch block).
 */
export const captureError = (error: Error | string, options?: CaptureOptions) => {
    const requestId = options?.requestId ?? crypto.randomUUID().replace(/-/g, "");
    const finalOptions = { ...options, requestId };

    Sentry.withScope((scope) => {
        scope.setLevel('error');
        // TODO: customise this tag name
        scope.setTag('x-myApp-request-id', requestId);
        if (finalOptions.siteId) scope.setTag('siteId', finalOptions.siteId);
        if (finalOptions.instanceId) scope.setTag('instanceId', finalOptions.instanceId);

        if (error instanceof Error) {
            Sentry.captureException(error);
        } else {
            Sentry.captureMessage(error, 'error');
        }
    });

    return { requestId };
};

/**
 * Automatically called from middleware.ts on every request/response cycle.
 * You generally shouldn't need to call this yourself.
 */
export const captureRequest = (request: any, response: any, responseBody: any, requestBody: any, options?: CaptureOptions) => {
    const level = response.status >= 500 ? 'error' : response.status >= 400 ? 'warning' : 'info';
    const apiPath = new URL(request.url).pathname;

    Sentry.withScope((scope) => {
        scope.setLevel(level);

        // TODO: customise this tag name
        if (options?.requestId) scope.setTag('x-myApp-request-id', options.requestId);
        if (options?.siteId) scope.setTag('siteId', options.siteId);
        if (options?.instanceId) scope.setTag('instanceId', options.instanceId);

        scope.setExtra('request', {
            method: request.method,
            url: request.url,
            headers: Object.fromEntries(request.headers.entries()),
        });
        scope.setExtra('requestBody', requestBody);
        scope.setExtra('response', {
            status: response.status,
            headers: Object.fromEntries(response.headers.entries()),
        });
        scope.setExtra('responseBody', responseBody);
        Sentry.captureMessage(`New Request: ${apiPath} ${request.method} ${response.status}`, level);
    });
};

export { Sentry };
```

### `src/pages/api/reportError.ts`

```typescript
// Thin re-export so API routes have a short, conventional import path
// (`../reportError`) without duplicating Sentry.init(). All actual logic
// lives in src/monitoring/sentry.ts, which is the single source of truth.
export { captureError } from '../../monitoring/sentry';
```

### `src/middleware.ts`

```typescript
import { defineMiddleware } from "astro:middleware";
import { captureRequest } from './monitoring/sentry';

const getTokenInfo = async (authToken: string | null) => {
    if (!authToken) return null;
    try {
        const response = await fetch("https://www.wixapis.com/oauth2/token-info", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ token: authToken }),
        });
        return await response.json();
    } catch (error) {
        console.error('Error fetching token info:', error);
        return null;
    }
};

export const onRequest = defineMiddleware(async (context, next) => {
    // Generate a fallback request ID in case neither the incoming request
    // nor the outgoing response already carries one.
    const requestId = crypto.randomUUID().replace(/-/g, "");

    // TODO: replace "myApp" with your app's name so this header doesn't get
    // confused with a Wix-generated request ID.
    const currentRequestId = context.request.headers.get("x-myApp-request-id");

    const requestBodyText = await context.request.clone().json().catch(() => "");
    const response = await next();
    const info = await getTokenInfo(context.request.headers.get("Authorization"));

    // Site ID and instance ID help identify which Wix site produced the error.
    const siteId = info?.siteId || "unknown";
    const instanceId = info?.instanceId || "unknown";

    // TODO: customise this header name
    const endpointRequestId = response.headers.get("x-myApp-request-id");

    // Prefer whichever request ID already exists so errors thrown later in
    // the same request chain can be correlated back to this log entry.
    const finalRequestId = currentRequestId || endpointRequestId || requestId;

    const bodyJson = await response.clone().json().catch(() => null);
    const responseBody = JSON.stringify(bodyJson) || "";
    const requestBody = requestBodyText;

    // TODO: customise this header name
    response.headers.set("x-myApp-request-id", finalRequestId);

    // Logs every request/response pair to Sentry, regardless of status code.
    captureRequest(context.request, response, responseBody, requestBody, { requestId: finalRequestId, siteId, instanceId });

    return response;
});
```

## Usage

Every request is logged to Sentry automatically via `middleware.ts` — you don't need to do anything extra for that. Inside your own API routes, call `captureError()` in a `catch` block to send exceptions to Sentry and get a request ID back to return to the client.

### Example

```typescript
import type { APIRoute } from 'astro';
import { auth } from '@wix/essentials';
import { items } from '@wix/data';
import { captureError } from '../reportError';

export const POST: APIRoute = async ({ request }) => {
    let status = 500;
    try {
        const body = await request.json();
        const { id } = body;

        if (!id) {
            status = 400;
            throw new Error('Role ID is required for duplication.');
        }

        const findId = () => {
          // example function
        }

        return new Response(JSON.stringify({ success: true, id: findId() }), {
            status: 200,
            headers: {
                'Content-Type': 'application/json',
            },
        });
    } catch (error: any) {
        // Capture the error and send to Sentry
        const { requestId } = captureError(error);

        // Customise your error message
        return new Response(JSON.stringify({ success: false, error: 'Error with....' }), {
            status: status,
            headers: {
                'Content-Type': 'application/json',

                // Customise the request ID header name
                'x-myApp-request-id': requestId,
            },
        });
    }
};
```

> **Note:** This example route lives at `src/pages/api/*.ts` alongside `reportError.ts`, so `../reportError` resolves correctly. If your route lives deeper (e.g. `src/pages/api/roles/duplicate.ts`), either adjust the relative path (`../../reportError`) or import directly from `../../monitoring/sentry` instead.

## Customization Checklist

Before shipping, search the code for `TODO:` comments and update:

- [ ] Replace `"myApp"` in `x-myApp-request-id` (both the header name and the Sentry tag name) with your actual app name, in **all three files** — it must match everywhere for correlation to work.
- [ ] Declare `SENTRY_DSN` in `astro.config.mjs` under `env.schema`, and set the real value in `.env` (don't hardcode it).
- [ ] Confirm the `environment` value in `sentry.ts` matches your deploy setup if you have more than just `production`/`development`.
- [ ] Update the placeholder error message in the example route (`'Error with....'`).
