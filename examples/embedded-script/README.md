# Embedded Script Example

This guide walks through creating and configuring an embedded script extension for a Wix app, including an optional dashboard page for configuring it at runtime.

## Table of Contents

- [Permissions](#permissions)
- [Create Your Embedded Script Extension](#create-your-embedded-script-extension)
- [Live Reload in Dev Mode](#live-reload-in-dev-mode)
- [Dashboard Configuration](#dashboard-configuration)
- [Examples](#examples)
  - [Dashboard Page Code](#dashboard-page-code)
  - [Embedded Script Code](#embedded-script-code)

## Permissions

Before you begin, add the following permission scope to your app so it can manage embedded scripts:

```
SCOPE.DC-APPS.MANAGE-EMBEDDED-SCRIPTS
```

## Create Your Embedded Script Extension

Use the `generate` command to scaffold your embedded script extension. This creates a base directory at:

```
src/extensions/site/embedded-scripts/<your-embedded-script-name>/
```

Inside that directory, you'll find two files:

| File | Purpose |
|---|---|
| `<your-embedded-script-name>.extension.ts` | Extension configuration/entry point |
| `<your-embedded-script-name>.html` | The markup injected into the site |

## Live Reload in Dev Mode

To see live changes while developing — without releasing a new version each time — add an additional TypeScript file alongside the HTML file:

```
src/extensions/site/embedded-scripts/<your-embedded-script-name>/<your-embedded-script-name>.ts
```

Reference this file from a `<script>` tag inside your `.html` file (see the [example](#embedded-script-code) below).

## Dashboard Configuration

If you want site owners to configure your embedded script from a dashboard page, generate a dashboard page extension using the `generate` command as well.

For the full set of available parameters and methods, refer to the Wix API reference for embedded scripts:
https://dev.wix.com/docs/api-reference/app-management/embedded-scripts/introduction

## Examples

The example below implements a configurable button: a dashboard page lets the site owner set the button's text, color, and enabled state, and those settings are passed into the embedded script and rendered on the live site.

### Dashboard Page Code

```tsx
import { useEffect, useState, type FC } from 'react';
import {
  Page,
  Card,
  Box,
  FormField,
  Input,
  Button,
  Loader,
  ToggleSwitch,
  ColorPicker,
  Popover,
  Text,
} from '@wix/design-system';
import '@wix/design-system/styles.global.css';
import { dashboard } from '@wix/dashboard';
import { embeddedScripts } from '@wix/app-management';
import withProviders from '../../withProviders';
import type { ButtonScriptOptions } from './types';

const DEFAULT_OPTIONS: ButtonScriptOptions = {
  buttonText: 'Click me',
  buttonColor: '#3899EC',
  enabled: true,
};

const ButtonSettingsPage: FC = () => {
  const [options, setOptions] = useState<ButtonScriptOptions>(DEFAULT_OPTIONS);
  const [isLoading, setIsLoading] = useState(true);
  const [isSaving, setIsSaving] = useState(false);
  const [colorPickerOpen, setColorPickerOpen] = useState(false);
  const [pendingColor, setPendingColor] = useState(DEFAULT_OPTIONS.buttonColor);

  useEffect(() => {
    const loadSettings = async () => {
      try {
        const embeddedScript = await embeddedScripts.getEmbeddedScript();
        const data = (embeddedScript.parameters ?? {}) as Partial<Record<keyof ButtonScriptOptions, string>>;
        setOptions((prev) => ({
          ...prev,
          buttonText: data.buttonText ?? prev.buttonText,
          buttonColor: data.buttonColor ?? prev.buttonColor,
          enabled: data.enabled === 'false' ? false : data.enabled === 'true' ? true : prev.enabled,
        }));
        setPendingColor(data.buttonColor ?? DEFAULT_OPTIONS.buttonColor);
      } catch (error) {
        console.error('Failed to load button settings:', error);
      } finally {
        setIsLoading(false);
      }
    };
    loadSettings();
  }, []);

  const handleSave = async () => {
    setIsSaving(true);
    try {
      await embeddedScripts.embedScript({
        parameters: {
          buttonText: options.buttonText,
          buttonColor: options.buttonColor,
          enabled: String(options.enabled),
        },
      });
      dashboard.showToast({ message: 'Button settings saved!', type: 'success' });
    } catch (error) {
      console.error('Failed to save button settings:', error);
      dashboard.showToast({ message: 'Failed to save settings.', type: 'error' });
    } finally {
      setIsSaving(false);
    }
  };

  if (isLoading) {
    return (
      <Page height="100vh">
        <Page.Content>
          <Box align="center" verticalAlign="middle" height="50vh">
            <Loader text="Loading settings..." />
          </Box>
        </Page.Content>
      </Page>
    );
  }

  return (
    <Page height="100vh">
      <Page.Header
        title="Button Settings"
        subtitle="Configure the button that appears on your site."
        actionsBar={
          <Button onClick={handleSave} disabled={isSaving || !options.buttonText.trim()}>
            {isSaving ? 'Saving...' : 'Save'}
          </Button>
        }
      />
      <Page.Content>
        <Card>
          <Card.Header title="Button Configuration" />
          <Card.Content>
            <Box direction="vertical" gap="SP4">
              <FormField
                label="Button Text"
                required
                statusMessage={!options.buttonText.trim() ? 'Button text is required' : undefined}
                status={!options.buttonText.trim() ? 'error' : undefined}
              >
                <Input
                  value={options.buttonText}
                  onChange={(e) => setOptions((prev) => ({ ...prev, buttonText: e.target.value }))}
                  placeholder="Enter button label"
                />
              </FormField>

              <FormField label="Button Color">
                <Popover
                  shown={colorPickerOpen}
                  onClickOutside={() => {
                    setColorPickerOpen(false);
                    setPendingColor(options.buttonColor);
                  }}
                  appendTo="window"
                  placement="bottom"
                >
                  <Popover.Element>
                    <button
                      style={{
                        display: 'flex',
                        alignItems: 'center',
                        gap: '8px',
                        cursor: 'pointer',
                        border: 'none',
                        background: 'none',
                        padding: 0,
                        fontFamily: 'inherit',
                      }}
                      onClick={() => {
                        setPendingColor(options.buttonColor);
                        setColorPickerOpen((prev) => !prev);
                      }}
                    >
                      <Box
                        width="24px"
                        height="24px"
                        borderRadius="4px"
                        style={{
                          backgroundColor: options.buttonColor,
                          border: '1px solid #e0e0e0',
                          flexShrink: 0,
                        }}
                      />
                      <Text>{options.buttonColor}</Text>
                    </button>
                  </Popover.Element>
                  <Popover.Content>
                    <ColorPicker
                      value={pendingColor}
                      onChange={(color) => {
                        const hex = typeof color === 'string' ? color : color.hex();
                        setPendingColor(hex);
                      }}
                      onConfirm={(color) => {
                        const hex = typeof color === 'string' ? color : color.hex();
                        setOptions((prev) => ({ ...prev, buttonColor: hex }));
                        setColorPickerOpen(false);
                      }}
                      onCancel={() => {
                        setColorPickerOpen(false);
                        setPendingColor(options.buttonColor);
                      }}
                    />
                  </Popover.Content>
                </Popover>
              </FormField>

              <FormField label="Enable Button">
                <ToggleSwitch
                  checked={options.enabled}
                  onChange={(e) => setOptions((prev) => ({ ...prev, enabled: e.target.checked }))}
                />
              </FormField>
            </Box>
          </Card.Content>
        </Card>
      </Page.Content>
    </Page>
  );
};

export default withProviders(ButtonSettingsPage);
```

### Embedded Script Code

**`<your-embedded-script-name>.html`**

```html
<div
  id="flexi-btn-config"
  data-button-text="{{buttonText}}"
  data-button-color="{{buttonColor}}"
  data-enabled="{{enabled}}"
></div>

<script type="module" src="./button-config.ts"></script>
```

**`<your-embedded-script-name>.ts`**

```ts
const config = document.getElementById('flexi-btn-config');
if (!config) throw new Error('FlexiButton: config element not found');

const { buttonText, buttonColor, enabled } = config.dataset;

if (enabled !== 'true') throw new Error('FlexiButton: disabled');

function initButton() {
  const label = buttonText || 'Click me';
  const color = buttonColor || '#3899EC';

  const btn = document.createElement('button');
  btn.id = 'flexi-site-button';
  btn.textContent = label;
  Object.assign(btn.style, {
    position: 'fixed',
    top: '50%',
    left: '50%',
    transform: 'translate(-50%, -50%)',
    zIndex: '9999',
    padding: '12px 24px',
    backgroundColor: color,
    color: '#ffffff',
    border: 'none',
    borderRadius: '4px',
    fontSize: '16px',
    cursor: 'pointer',
    boxShadow: '0 2px 8px rgba(0,0,0,0.2)',
  });

  document.body.appendChild(btn);
}

if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', initButton);
} else {
  initButton();
}
```
