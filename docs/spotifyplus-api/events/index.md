---
sidebar_label: 'Event payloads'
---

# Event payloads

`SpotifyPlus.Events` broadcasts Spotify state changes to extensions. Custom events emitted with `Events.emit()` stay inside the emitting extension.

| Event | Payload |
| --- | --- |
| `settingChanged` | `{ id: string; value: SpotifySettingValue \| null; previousValue: SpotifySettingValue \| null }` |
| `upNextChanged` | `{ revision: string; contextUri: string; reason: 'initial' \| 'context' \| 'advance' \| 'replacement'; source: 'spotify' \| 'extension' }` |
| `contextChanged` | `{ uri: string; previousUri: string }` |
| `songChanged` | `{ uri: string \| null; previousUri: string \| null; reason: SongChangeReason; reasonSource: 'command' \| 'inferred' \| 'unknown' }` |
| `playPause` | `{ isPlaying: boolean; isPaused: boolean }` |
| `trackSeeked` | `{ positionMs: number; previousPositionMs: number }` |
| `shuffleChanged` | `{ enabled: boolean }` |
| `repeatChanged` | `{ mode: RepeatMode }` |
| `deviceChanged` | `{ device: ConnectDevice \| null; previousDeviceId: string \| null }` |
| `settings.changed` | `ExtensionSetting`, for your extension's settings UI |