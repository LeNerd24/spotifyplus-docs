---
sidebar_label: 'Spotify settings'
---

# Spotify settings

These types describe Spotify's own settings. For an extension's settings page, see [Extension settings](./extension-settings.md).

```ts
type SpotifySettingValue = boolean | number | string;

interface SpotifySetting {
    id: string;
    value: SpotifySettingValue;
    type: 'boolean' | 'number' | 'string';
    writable: boolean;
}
```

`Settings.list()` returns these descriptors. A setting can be unavailable until Spotify initializes its backend. `settingChanged` may contain `null` when a local preference is removed.
