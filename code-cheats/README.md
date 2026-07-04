# Code Cheatsheet

### Get editor type through window 

```
type ExtendedWindow = Window &
  typeof globalThis & { viewerModel?: { site?: { editorName?: string } } };

export const isStudioSite = (): boolean =>
  (window as ExtendedWindow).viewerModel?.site?.editorName === 'Studio';
```
