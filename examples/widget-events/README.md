# Widget Events

Widget events let your Wix app widget — built with the Wix CLI — communicate back to the Wix site. The widget dispatches a native browser `CustomEvent`, and your site's Velo code listens for it using the `.on()` method on the widget element.

This pattern is useful any time something happens inside your widget (a button click, a form submission, a state change) that the surrounding Wix page needs to react to.

### Navigation
- [Class Component Example](#class-component-example)
- [Function Component Example](#function-component-example)
- [Site Code Example](#site-code-example)

---

## How It Works

1. **Inside the widget** — when an event occurs, the widget dispatches a `CustomEvent` with a `detail` payload. The event must use `bubbles: true` and `composed: true` so it crosses the shadow DOM boundary into the Wix page.
2. **On the site (Velo)** — the Velo `$w('#yourWidgetId').on('eventName', handler)` call listens for that event on the widget element and receives the `detail` data in the handler.

---

## Class Component Example

A vanilla Web Component approach. The widget dispatches a `change` event both on an automatic timer (after 3 seconds) and on every button click. The `detail` object carries the current `count` value.

```javascript showLineNumbers
class MyElement extends HTMLElement {
  button!: HTMLButtonElement;
  count = 0;
  static get observedAttributes() {
    return ['count'];
  }
  attributeChangedCallback(name: string, oldValue: string, newValue: string) {
    if (name === 'count') {
      this.count = Number(newValue);
      this.update();
    }
  }
  connectedCallback() {
    this.render();
    
    // Fires a change event automatically after 3 seconds
    setTimeout(() => {
      this.changeEvent();
    }, 3000);

    this.button = this.querySelector('button')!;
    this.button.addEventListener('click', () => {
      this.count++;
      this.changeEvent(); // Fires a change event on every click
      this.update();
    });
  }
  changeEvent() {
    // Dispatches a CustomEvent with the current count.
    // bubbles + composed: true are required to cross the shadow DOM boundary
    // so Velo can catch the event on the host page.
    this.dispatchEvent(
      new CustomEvent('change', {
        detail: {
          count: this.count
        },
        bubbles: true,
        composed: true
      })
    );
  }
  render() {
    this.innerHTML = `
    <style>
          .action-button {
            background: #2563eb;
            color: #fff;
            border: none;
            border-radius: 999px;
            padding: 10px 18px;
            font-weight: 600;
            cursor: pointer;
            transition: background 180ms ease, transform 180ms ease, box-shadow 180ms ease;
            box-shadow: 0 8px 20px rgba(37, 99, 235, 0.18);
          }
          .action-button:hover {
            background: #7c3aed;
            transform: translateY(-1px);
            box-shadow: 0 12px 24px rgba(124, 58, 237, 0.22);
          }
          .button-group {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            margin-top: 16px;
          }
        </style>
        
        <div>
      <button class="action-button">Click to Increment</button>
      </div>
      <div>
      <h1>Count: ${this.count}</h1>
      </div>
    `;
  }
  update() {
    this.querySelector('h1')!.textContent = `Count: ${this.count}`;
  }
}
export default MyElement;
```

---

## Function Component Example

A React-based approach using `react-to-webcomponent` to wrap a functional component as a Web Component. The `useEffect` hook fires the `change` event whenever `count` changes, dispatching from the root `div` ref (which becomes the shadow host). The `count` prop can optionally be passed in from the Wix Editor to set the starting value.

```javascript showLineNumbers
import React, { type FC, useEffect, useRef, useState } from 'react';
import ReactDOM from 'react-dom';
import reactToWebComponent from 'react-to-webcomponent';

interface Props {
  count?: number;
}

const CustomElement: FC<Props> = ({ count: initialCount = 0 }) => {
  const [count, setCount] = useState(initialCount);
  const hostRef = useRef<HTMLDivElement>(null);

  // Dispatch a change event whenever count updates.
  // The event originates from the root div (shadow host) so it
  // can be caught by Velo on the host page.
  useEffect(() => {
    hostRef.current?.dispatchEvent(
      new CustomEvent('change', {
        detail: { count },
        bubbles: true,
        composed: true,
      })
    );
  }, [count]);

  function handleClick() {
    setCount((prev) => prev + 1);
  }

  return (
    <div ref={hostRef} className="root">
      <div>
        <button className="action-button" onClick={handleClick}>
          Click to Increment
        </button>
      </div>
      <div>
        <h1>Count: {count}</h1>
      </div>
    </div>
  );
};

// Wrap the React component as a Web Component, exposing `count` as a
// settable numeric attribute/property from the Wix Editor or Velo.
const customElement = reactToWebComponent(
  CustomElement,
  React,
  ReactDOM as any,
  {
    props: {
      count: 'number',
    },
  }
);
export default customElement;
```

---

## Site Code Example

On the Wix site, use Velo's `$w().on()` to subscribe to the widget's custom event. The `event.detail` object contains whatever data the widget included in its `CustomEvent` — in this case, `count`.

This code goes in your **site's page code** or **masterpage.js**, not inside the widget itself.

**Note** — Replace `#eventCommsWidget1` with your actual widget element ID from the Wix Editor.

```javascript showLineNumbers
$w.onReady(function () {
    // Listen for the 'change' event emitted by the widget.
    // event.detail matches the `detail` object dispatched in the widget.
    $w('#eventCommsWidget1').on("change", (event) => {
        console.log(`Info received from change event: ${event.detail.count}`);
        $w('#wixOutput').text = `${event.detail.count}`;
    });
});
```
