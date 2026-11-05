---
sidebar_label: 'Theme types'
---

# Theme types

`SpotifyPlus.Theme` lets your extension change Spotify Plus's theme. You can choose colors yourself, or have the theme follow the current song's artwork.

```ts
interface ThemeConfiguration {
    enabled: boolean;
    colors: Record<string, string>;
    artworkBlend: number;
    autoTheme: boolean;
    autoThemeFade: boolean;
    autoThemeStyle: 'light' | 'neutral' | 'dark';
}

interface ThemeState {
    active: ThemeConfiguration;
    saved: ThemeConfiguration;
    restartRequired: boolean;
    supported: boolean;
    supportedVersion: string;
    installedHooks: number;
    failures: string[];
}
```

`active` describes the applied configuration; `saved` describes the stored configuration. Check `restartRequired` after making a change. `supportedVersion`, `installedHooks`, and `failures` help diagnose theme support on the installed Spotify build.

## Configuration

`artworkBlend` is a number from 0 to 1 controlling how much artwork color is retained in themed backgrounds. `autoTheme` generates colors from album artwork when Spotify opens and when the song changes. `autoThemeFade` fades between song colors and defaults to `true`.

`autoThemeStyle` accepts `light` for a pastel of the artwork color, `neutral` for the artwork color itself, or `dark` for a darker version.

`colors` accepts the following roles:

```ts
'background', 'backgroundHighlight', 'backgroundPress',
'surface', 'surfaceHighlight', 'surfacePress', 'frame',
'text', 'textSubdued', 'essential', 'essentialSubdued',
'accent', 'accentHighlight', 'accentPress', 'accentText', 'onAccent',
'decorative', 'negative', 'warning', 'positive', 'announcement', 'onArtwork'
```

Use color strings such as `#f3ebff`. Unknown roles and configuration keys throw. You can update a single role without supplying the whole palette:

```ts
SpotifyPlus.Theme.set({ colors: { accent: '#c7a1ef' } });
```
