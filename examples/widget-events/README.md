# Widget Events

### Navigation
- [Class Component Example](#class-component-example)
- [Function Component Example](#function-component-example)
- [Site Code Example](#site-code-example)

## Class Component Example

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
    
    setTimeout(() => {
      this.changeEvent();
    }, 3000);

        this.button = this.querySelector('button')!;

    this.button.addEventListener('click', () => {
      this.count++;

      this.changeEvent();
      this.update();
    });
  }

  changeEvent() {
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
    this.innerHTML = \`

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
      <h1>Count: \${this.count}</h1>
      </div>
    \`;
  }

  update() {
    this.querySelector('h1')!.textContent = \`Count: \${this.count}\`;
  }
}

export default MyElement;
```

## Function Component Example 

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

## Site Code Example 
**Note** - Replace `#eventCommsWidget1` with your ACTUAL widget ID

```javascript showLineNumbers
$w.onReady(function () {
    $w('#eventCommsWidget1').on("change", (event) => {
        console.log(\`Info received from change event: \${event.detail.count}\`);
        $w('#wixOutput').text = \`\${event.detail.count}\`;
    });
});
```
