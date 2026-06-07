# Hello World Widget Example

## Widget Example
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

const makeResponsiveFont = (fontString: string): string => {
  return fontString.replace(/(d+(?:.d+)?)px/, (_, size) => {
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
