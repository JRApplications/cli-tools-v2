# Hello World Widget Example

A minimal end-to-end example of a Wix CLI app widget. It demonstrates how a widget renders content using props, and how the Editor panel lets the user control those props through the Wix Design System UI.

There are two parts:
- **Widget** — the React component compiled as a Web Component that renders on the Wix site
- **Panel** — the Editor sidebar UI that reads and writes the widget's props

---

## Widget Example

The widget renders a simple "Hello World" heading. It accepts three props from the Editor panel — `font`, `textDecoration`, and `textColor` — and applies them as inline styles.

The `makeResponsiveFont` helper converts a fixed pixel font size (as supplied by Wix's font picker) into a `clamp()`-based responsive value, so the heading scales fluidly across screen widths rather than staying fixed.

The component is wrapped with `reactToWebComponent` so it can be mounted as a Web Component inside the Wix platform. The `props` config tells the wrapper which attributes to expose and their types.

```javascript showLineNumbers
import React, { type FC } from 'react';
import ReactDOM from 'react-dom';
import reactToWebComponent from 'react-to-webcomponent';
import styles from './hello-world.module.css';

interface Props {
  font?: string;
  textDecoration?: string;
  textColor?: string;
}

// Converts a fixed px font value from the Wix font picker into a responsive
// clamp() value so the text scales with the viewport rather than staying fixed.
const makeResponsiveFont = (fontString: string): string => {
  return fontString.replace(/(\d+(?:\.\d+)?)px/, (_, size) => {
    const base = parseFloat(size);
    const min = Math.max(12, Math.round(base * 0.5));
    const vw = (base / 1440 * 100).toFixed(2);
    return `clamp(${min}px, ${vw}vw, ${base}px)`;
  });
};

const CustomElement: FC<Props> = ({
  font = '', 
  textDecoration = '',
  textColor = '',
}) => {
  const responsiveFont = font ? makeResponsiveFont(font) : font;
  console.log('Rendering with props:', { font, textDecoration, textColor });
  return (
    <div className={styles.root}>
      <h2 style={{
        font: responsiveFont, 
        textDecoration, 
        color: textColor, 
        textAlign: 'center'
      }}>Hello World</h2>
    </div>
  );
};

// Expose the component as a Web Component with typed props so the
// Wix Editor and Velo can get/set them by name.
const customElement = reactToWebComponent(
  CustomElement,
  React,
  ReactDOM as any,
  {
    props: {
        font: 'string',
        textDecoration: 'string',
        textColor: 'string',
    },
  }
);

export default customElement;
```

---

## Panel Example

The panel is the Editor sidebar UI for the widget. It uses the [Wix Design System](https://www.wix.com/design-system) for layout and controls, and the `@wix/editor` package to read and write the widget's props.

On mount, `useEffect` fetches the current prop values from the widget so the panel reflects its existing state. When the user picks a font or color, the handler both updates local React state and calls `widget.setProp()` to push the new value to the widget in real time.

- **Font picker** — uses `inputs.selectFont()`, which opens the Wix font picker. It returns both the font string and a `textDecoration` value, so both props are updated together.
- **Color picker** — uses `inputs.selectColor()`, which opens the Wix color picker and returns a color string.
- **Reset button** — clears all props back to empty strings, removing any custom styling from the widget.

```javascript showLineNumbers
import { useState, useEffect } from 'react';
import { type FC } from 'react';
import {
  WixDesignSystemProvider,
  SidePanel,
  FormField,
  Button,
  Box,
  FillPreview,
} from '@wix/design-system';
import { widget, inputs } from '@wix/editor';
import '@wix/design-system/styles.global.css';

const Panel: FC = () => {
  const [font, setFont] = useState<string>('');
  const [textDecoration, setTextDecoration] = useState<string>('');
  const [color, setColor] = useState<string>('');

  // On mount, read current prop values from the widget so the panel
  // reflects whatever is already set (e.g. after a page reload).
  useEffect(() => {
    widget.getProp('font')
      .then(font => setFont(font || ''))
      .catch(error => console.error('Failed to fetch font:', error));
    widget.getProp('text-decoration')
      .then(textDecoration => setTextDecoration(textDecoration || ''))
      .catch(error => console.error('Failed to fetch text-decoration:', error));
    widget.getProp('text-color')
      .then(color => setColor(color || ''))
      .catch(error => console.error('Failed to fetch text-color:', error));
  }, [setFont, setTextDecoration, setColor]);

  // The font picker returns both font and textDecoration together,
  // so both props are updated in a single handler.
  const handleFontChange = (value: any) => {
    const font = value.font;
    const decoration = value.textDecoration;
    setFont(font);
    widget.setProp('font', font);
    setTextDecoration(decoration);
    widget.setProp('text-decoration', decoration);
  };

  const handleColorChange = (value: string) => {
    setColor(value);
    widget.setProp('text-color', value)
      .catch(error => console.error('Failed to set color:', error));
  };

  return (
    <WixDesignSystemProvider>
      <SidePanel width="300" height="100vh">
        <SidePanel.Content noPadding stretchVertically>

          {/* Opens the Wix font picker; updates font and textDecoration together */}
          <SidePanel.Field divider>
            <FormField label="Text Font" labelPlacement='left' labelWidth='1fr'>
              <Button size='small' onClick={() =>
                inputs.selectFont({ font, textDecoration }, {
                  onChange: (value: any) => {
                    handleFontChange(value);
                  },
                })
              }>
                Select Font
              </Button>
            </FormField>
          </SidePanel.Field>

          {/* Opens the Wix color picker; FillPreview shows the currently selected color */}
          <SidePanel.Field divider>
            <FormField label="Text Color" labelPlacement='left' labelWidth='1fr'>
              <Box width="30px" height="30px">
                <FillPreview fill={color} onClick={() =>
                  inputs.selectColor(color, {
                    onChange: (value: any) => {
                      console.log('value: ', value)
                      handleColorChange(value ?? 'red');
                    },
                  })
                } />
              </Box>
            </FormField>
          </SidePanel.Field>

        </SidePanel.Content>

        {/* Clears all props, restoring the widget to its unstyled default */}
        <Button onClick={() => {
          setFont('');
          setTextDecoration('');
          setColor('');
          widget.setProp('font', '');
          widget.setProp('text-decoration', '');
          widget.setProp('color', '');
        }}>Reset</Button>
      </SidePanel>
    </WixDesignSystemProvider>
  );
};

export default Panel;
```

Key additions:
- **Intro section** explaining the two-part structure (widget + panel) and how they relate
- **Widget prose** covering what it renders, what the props do, and why `makeResponsiveFont` exists
- **Panel prose** covering the mount behaviour, the two pickers, and the reset
- **Inline comments** at the key decision points in both files
