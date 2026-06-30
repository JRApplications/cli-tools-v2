# Sentry Monitoring Extension 

**DISCLAMER** - These are not actual "cheats" but just examples of extensions that are not officially documented but can be used. You would normally add these extensions in your app dashboard. Use at your own risk.

## Code
Place this in your widget pages direction. e.g `src/extensions/monitoring/sentryMonitoring.extension.ts`

Don't forget to apply this extension to your app in `src/extensions.ts`

```javascript showLineNumbers
import { extensions } from '@wix/astro/builders';

export default extensions.genericExtension({
    // you will need to generate a unique uuid 
    // for your compId
    compId: '<comp-id>',
    compName: 'Monitoring',
    compType: 'MONITORING',
    compData: {
        type: "SENTRY",
        sentryOptions: {
            dsn: '<your-sentry-dns>'
        }
    }
});
```
