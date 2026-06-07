# Page Extension 

**DISCLAMER** - These are not actual "cheats" but just examples of extensions that are not officially documented but can be used. You would normally add these extensions in your app dashboard. Use at your own risk.

## Code
Place this in your widget pages direction. e.g `src/extensions/site/pages/<page-name>/extension.ts`

Don't forget to apply this extension to your app in `src/extensions.ts`

```javascript showLineNumbers
import { extensions } from '@wix/astro/builders';

export default extensions.genericExtension({
    // you will need to generate a unique uuid 
    // for your compId
    compId: '<comp-id>',
    compName: '<page-name>',
    compType: 'UNIFIED_PAGE',
    compData: {
        "unifiedPage": {
            "base": {
                "name": "<page-name>",
                "id": "<page-id>",
                "description": ""
            },
            "installation": {
                "base": {
                    "autoAdd": true,
                    "essential": false,
                    "maxInstances": 1
                },
                "page": {
                    "slug": "/<page-id>",
                    "addToSiteMenu": true,
                    "linkable": false
                }
            },
            "content": {
                // in order to automatically add the widget to the page
                // you will need to make autoAdd false in your widgets
                // .extension.ts file.

                // if you don't want to add a widget to the page 
                // then leave "widgets" a empty array.
                "widgets": [
                    {
                        // widgetGuid also known as widgetId or id can be found in the widget's 
                        // .extension.ts file
                        "widgetGuid": "<widget-guid>",
                        "essential": true
                    }
                ]
            },
            "editorSettings": {
                "base": {
                    "addToSiteMenu": true,
                    "duplicatable": false
                },
                "slug": "/<page-id>"
            }
        }
    }
});
```
