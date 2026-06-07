# Realtime Extension Example

## Widget Code
```javascript showLineNumbers
// place this in your widget file - src/extensions/site/widgets/<widget-name>/<widget-name>.tsx

import React, { type FC } from 'react';
import ReactDOM from 'react-dom';
import reactToWebComponent from 'react-to-webcomponent';
import { subscriber } from '@wix/realtime';
import { httpClient } from '@wix/essentials';

interface Props {

}

const CustomElement: FC<Props> = ({ }) => {
  const [message, setMessage] = React.useState('Hello World');
  const [messageValue, setMessageValue] = React.useState('');

  const handleSendMessage = async () => {
    const baseApiUrl = new URL(import.meta.url).origin;
    const res = await httpClient.fetchWithAuth(\`\${baseApiUrl}/api/send-message\`\,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: messageValue })
      }
    )
    const data = await res.json();
    if (!data.success) {
      console.error('Failed to send message:', data.error);
    }
  };

  React.useEffect(() => {
    const channel = { name: "someChannel" };

    subscriber.subscribe(channel, (message, channel) => {
      let payload = message.payload;
      console.log('Received message:', payload);
      setMessage(String(payload.message));
    }, {
      onSubscribed: (id, isSynced) => console.log('Subscribed', id),
      onSubscriptionError: (error, id) => console.error('Error', error),
    });

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

const customElement = reactToWebComponent(
  CustomElement,
  React,
  ReactDOM as any,
  {
    props: {},
  }
);

export default customElement;
```

## Extension Code

```javascript showLineNumbers
// place this in your backend direction. e.g src/extensions/backend/service-plugins/<service-plugin-name>/<service-plugin-name>.extension.ts
// don't forget to update this extension to your app in src/extensions.ts

import { extensions } from '@wix/astro/builders';

export default extensions.realtimePermissionsProvider({
  id: <unique-uuid>,
  name: <service-plugin-name>, // e.g 'rt-permissions'
  source: './extensions/backend/service-plugins/<service-plugin-name>/<service-plugin-name>.ts',
  providerName: '<service-plugin-name>', // e.g 'rt-permissions'
});
```

## Permissions Code

```javascript showLineNumbers
// place this in your realtime service plugin direction. e.g src/extensions/backend/service-plugins/<service-plugin-name>/<service-plugin-name>.extension.ts

import { realtimePermissionsProvider } from "@wix/realtime/service-plugins";

realtimePermissionsProvider.provideHandlers({
  checkSubscriberPermissions: async (payload) => {
    const { request, metadata } = payload;

    // Allow members and admins, deny visitors.
    return { read: request.subscriber.type !== "VISITOR" };
  },
});
```

## Endpoint Code 

```javascript showLineNumbers
// place this in your endpoint direction. e.g src/pages/api/<your-endpoint-name>.ts
// CAUTION - Realtime Publish function may need elevation
// DON'T FORGET TO ADD YOUR REALTIME PERMISSION IF IN A APP ENVIRONMENT - "3DSCOPE.API_INFRA.REALTIME_PUBLISH"

import { auth } from '@wix/essentials';
import { publisher } from '@wix/realtime';
import type { APIRoute } from 'astro';

export const POST: APIRoute = async ({ request }) => {
    try {
        const { message } = await request.json();
        const channel = { name: "someChannel" };
        await auth.elevate(publisher.publish)(channel, { message });
        return new Response(JSON.stringify({ success: true }), { status: 200 });
    } catch (error: any) {
        return new Response(JSON.stringify({ success: false, error: error.message }), { status: 500 });
    }
};
```
