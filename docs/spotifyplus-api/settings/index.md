---
sidebar_label: 'Spotify settings'
sidebar_position: 0
---

# Spotify settings

If you want to change Spotify's own settings, you can do that through `SpotifyPlus.Settings`. For example, you could turn on private session, change crossfade, or change the shuffle algorithm. This works separately from [registering your extension's settings](../../guides/settings.md).

```ts
SpotifyPlus.Settings.set('privateSession', true);
SpotifyPlus.Settings.set('offlineMode', false);
SpotifyPlus.Settings.set('canvas', false);
SpotifyPlus.Settings.set('shuffleMode', 'random');
```

## Listen for changes

```ts
SpotifyPlus.Events.on('settingChanged', ({ id, value, previousValue }) => {
    console.log(id, previousValue, '->', value);
});
```

Changes from Spotify's UI and extensions use the same event. This event will not be fired during initialization or on start, only when a setting is actually being changed. If you change a setting inside this handler, check its value first so you do not create a loop.