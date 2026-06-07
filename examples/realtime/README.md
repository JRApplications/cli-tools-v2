# Realtime Extension Example

This example demonstrates using the [Wix Realtime SDK](https://dev.wix.com/docs/sdk/core-modules/realtime/realtime/introduction) in a Wix CLI app.

There are four parts that work together:

- **Widget** — subscribes to a realtime channel and renders incoming messages
- **Extension** — registers the realtime permissions service plugin with your app
- **Permissions** — backend handler that controls who is allowed to subscribe
- **Endpoint** — an API route that publishes a message to the channel

### Flow Overview

1. The widget mounts and subscribes to a named channel via `subscriber.subscribe()`
2. A user types a message and clicks Send — the widget `POST`s to your API endpoint
3. The endpoint calls `publisher.publish()` (elevated, as it requires extra permissions) to broadcast the message to the channel
4. All subscribed widget instances receive the message and update their UI instantly

---

## Widget Code

The widget subscribes to a realtime channel on mount using `subscriber.subscribe()`. When a message arrives on the channel, it updates the displayed heading. The input and button allow the user to send a new message, which is `POST`ed to the backend API endpoint using `httpClient.fetchWithAuth()` — this ensures the request is authenticated with the current site visitor's session.

The channel name (`"someChannel"`) must match exactly between the widget subscriber and the endpoint publisher.

```javascript showLineNumbers
// place this in your widget file
// src/extensions/site/widgets/<widget-name>/<widget-name>.tsx

import React, { type FC } from 'react';
import ReactDOM from 'react-dom';
import reactToWebComponent from 'react-to-webcomponent';
import { subscriber } from '@wix/realtime';
import { httpClient } from '@wix/essentials';

interface Props {}

const CustomElement: FC<Props> = ({}) => {
  const [message, setMessage] = React.useState('Hello World');
  const [messageValue, setMessageValue] = React.useState('');

  // POST the typed message to the backend endpoint, which will
  // publish it to the realtime channel for all subscribers.
  const handleSendMessage = async () => {
    const baseApiUrl = new URL(import.meta.url).origin;
    const res = await httpClient.fetchWithAuth(`${baseApiUrl}/api/send-message`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: messageValue }),
    });
    const data = await res.json();
    if (!data.success) {
      console.error('Failed to send message:', data.error);
    }
  };

  React.useEffect(() => {
    const channel = { name: "someChannel" };

    // Subscribe to the channel. The callback fires whenever a message
    // is published to this channel from the backend.
    // The channel name must match exactly what the endpoint publishes to.
    subscriber.subscribe(
      channel,
      (message, channel) => {
        let payload = message.payload;
        console.log('Received message:', payload);
        setMessage(String(payload.message));
      },
      {
        onSubscribed: (id, isSynced) => console.log('Subscribed', id),
        onSubscriptionError: (error, id) => console.error('Error', error),
      }
    );
  }, []);

  return (
    <div
      style={{
        maxWidth: '400px',
        margin: '40px auto',
        padding: '20px',
        border: '1px solid #ddd',
        borderRadius: '8px',
        display: 'flex',
        flexDirection: 'column',
        gap: '12px',
      }}
    >
      <h2 style={{ textAlign: 'center', margin: 0 }}>
        {message}
      </h2>

      <input
        style={{
          padding: '10px 12px',
          fontSize: '16px',
          border: '1px solid #ccc',
          borderRadius: '6px',
          outline: 'none',
        }}
        type="text"
        value={messageValue}
        onChange={(e) => setMessageValue(e.target.value)}
        onKeyDown={(e) => e.key === 'Enter' && handleSendMessage()}
        placeholder="Type a message"
      />

      <button
        style={{
          padding: '10px',
          fontSize: '16px',
          backgroundColor: '#2563eb',
          color: 'white',
          border: 'none',
          borderRadius: '6px',
          cursor: 'pointer',
        }}
        onClick={handleSendMessage}
      >
        Send Message
      </button>
    </div>
  );
};

const customElement = reactToWebComponent(CustomElement, React, ReactDOM as any, {
  props: {},
});

export default customElement;
```

---

## Extension Code

This file registers the realtime permissions service plugin with your app. It tells the Wix platform which backend file implements the permissions handler and under what name.

Replace the placeholder values:
- `<unique-uuid>` — generate any valid UUID for this extension
- `<service-plugin-name>` — a short identifier for this plugin, e.g. `rt-permissions`

After creating this file, make sure you also register the extension in `src/extensions.ts`.

```javascript showLineNumbers
// src/extensions/backend/service-plugins/<service-plugin-name>/<service-plugin-name>.extension.ts
// Don't forget to register this extension in src/extensions.ts

import { extensions } from '@wix/astro/builders';

export default extensions.realtimePermissionsProvider({
  id: <unique-uuid>,
  name: <service-plugin-name>,       // e.g. 'rt-permissions'
  source: './extensions/backend/service-plugins/<service-plugin-name>/<service-plugin-name>.ts',
  providerName: '<service-plugin-name>', // e.g. 'rt-permissions'
});
```

---

## Permissions Code

This is the backend handler that the Wix Realtime service calls to decide whether a given subscriber is allowed to read from a channel. It runs server-side whenever a client tries to subscribe.

The example allows logged-in members and admins but blocks visitors. You can extend this logic — for example checking membership tier, specific user IDs, or custom app data — by inspecting `payload.request` and `payload.metadata`.

```javascript showLineNumbers
// src/extensions/backend/service-plugins/<service-plugin-name>/<service-plugin-name>.ts

import { realtimePermissionsProvider } from "@wix/realtime/service-plugins";

realtimePermissionsProvider.provideHandlers({
  checkSubscriberPermissions: async (payload) => {
    const { request, metadata } = payload;

    // Allow members and admins to subscribe; deny visitors.
    // Extend this logic to restrict access further if needed,
    // e.g. by checking specific user roles or conditions.
    return { read: request.subscriber.type !== "VISITOR" };
  },
});
```

---

## Endpoint Code

This API route receives the `POST` from the widget and publishes the message to the realtime channel. `publisher.publish()` requires elevated permissions, so it is wrapped in `auth.elevate()`.

**Important notes:**
- The channel name must match exactly what the widget subscribes to
- `auth.elevate()` is required — publishing to realtime channels is a privileged operation
- If this code runs in an **app environment** (not just a site), you must also add the `3DSCOPE.API_INFRA.REALTIME_PUBLISH` permission to your app's permission scopes, otherwise the elevated call will be rejected

```javascript showLineNumbers
// src/pages/api/<your-endpoint-name>.ts

import { auth } from '@wix/essentials';
import { publisher } from '@wix/realtime';
import type { APIRoute } from 'astro';

export const POST: APIRoute = async ({ request }) => {
  try {
    const { message } = await request.json();
    const channel = { name: "someChannel" }; // Must match the channel name in the widget

    // publisher.publish requires elevation — it is a privileged realtime operation.
    // In an app environment, also add "3DSCOPE.API_INFRA.REALTIME_PUBLISH" to your app permissions.
    await auth.elevate(publisher.publish)(channel, { message });

    return new Response(JSON.stringify({ success: true }), { status: 200 });
  } catch (error: any) {
    return new Response(
      JSON.stringify({ success: false, error: error.message }),
      { status: 500 }
    );
  }
};
```
