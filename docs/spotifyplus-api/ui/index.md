---
sidebar_label: 'UI targets'
sidebar_position: 0
---

# UI targets

`SpotifyPlus.UI` mounts React components inside Spotify's existing screens. Use [the custom page tutorial](../../guides/custom-ui-page.md) for a complete extension, or the method pages for signatures and behavior. Import components from `spotifyplus/react`; this renderer creates Android views, so browser elements such as `div` are not supported.

## Supported boundaries

The current adapters target Spotify **9.1.82.2160**. Always call `inspect()` on the installed build before registering an operation. A name in the TypeScript union does not guarantee an adapter exists.

| Targets | Supported operations |
| --- | --- |
| `home.page`, `search.page`, `library.page`, `playlist.page`, `album.page`, `artist.page` | replace, overlay |
| `artist.discography.page`, `settings.page`, `profile.page`, `nowPlaying.page`, `lyrics.page`, `miniPlayer.root` | replace, overlay |
| `queue.page` | overlay |
| `navigation.drawer`, `contextMenu.root`, `contextMenu.header`, `contextMenu.action`, `contextMenu.actions` | replace, before, after, overlay |
| `playlist.header`, `album.header`, `artist.header`, `playlist.actions`, `album.actions`, `artist.actions` | replace, before, after, overlay on Compose instances |
| `library.header`, `library.filters`, `search.field`, `search.filters`, `search.results` | replace, before, after, overlay on Compose instances |
| `lyrics.header`, `lyrics.content`, `lyrics.line`, `navigation.bar`, `navigation.item` | replace, before, after, overlay |
| `home.header`, `home.section`, `home.item`, `search.item`, `library.list`, `library.item` | overlay |
| `playlist.section`, `playlist.list`, `playlist.item`, `album.section`, `album.list`, `album.item`, `artist.section`, `artist.list`, `artist.item` | overlay |
| `nowPlaying.header`, `nowPlaying.artwork`, `nowPlaying.trackInfo`, `nowPlaying.controls`, `nowPlaying.seekBar`, `nowPlaying.card` | overlay |
| `miniPlayer.artwork`, `miniPlayer.trackInfo`, `miniPlayer.controls`, `queue.header`, `queue.content`, `queue.item` | overlay |

All 61 built-in target names now have adapters in this Spotify version. Availability still depends on the installed hooks and screen variant. Native Android regions stay in Spotify's parent layout and support overlays. Some targets also have Compose adapters with replacement and insertion support; a native instance of the same target can still support only overlays. Check the operation on your installed build and test the screen variant you use.

Page targets preserve Spotify's outer navigation and fragment/activity boundaries. They do not replace the entire app. The mini-player adapter targets its non-embedded content.

## Resource selectors

You can also overlay an Android view by resource ID inside a supported screen:

```tsx
const target = { screen: 'nowPlaying.page', resourceId: 'com.spotify.music:id/cover_art_container', minVisible: 0.85 } as const;
const info = await SpotifyPlus.UI.inspect(target);
if (info.available && info.operations.includes('overlay')) {
    SpotifyPlus.UI.overlay(target, MyOverlay);
}
```

`MyOverlay` is your component type. Unless a selector resolves to a built-in region, each matching view gets its own instance and supports only `overlay`. `minVisible` is a fraction from 0 to 1 required to open an instance; once opened, the instance stays until the view is completely off screen. This is useful for the visible page of a carousel. Arbitrary Compose-tag discovery and extension-owned adapters are not implemented.

## Registration and lifecycle

```tsx
import { SpotifyPlus, type UIComponentProps } from 'spotifyplus';
import { View, Text } from 'spotifyplus/react';

function Header({ context, Original }: UIComponentProps) {
  return (
    <View style={{ padding: 16 }}>
      <Original />
      <Text>{context.uri ?? context.title ?? 'Spotify'}</Text>
    </View>
  );
}

async function install() {
  const info = await SpotifyPlus.UI.inspect('contextMenu.header');
  if (!info.available || !info.operations.includes('replace')) return;
  return SpotifyPlus.UI.replace('contextMenu.header', Header);
}

void install().catch(console.error);
```

`replace`, `insertBefore`, `insertAfter`, and `overlay` return a synchronous `UIRegistration` handle. Call `dispose()` for early removal; extension unload removes its registrations automatically. Registrations also apply to future instances of the target. An available adapter can have zero live instances until its screen opens.

Each live target has its own `context.instanceId`, React state, and native surface. Context updates rerender the mounted component while preserving its state. URI/title fields can be absent or null; handle that explicitly. Closed instances and disposed surfaces stop receiving their old events and commits.

The first replacement in extension priority order wins. Before/after contributions and overlays coexist. **Spotify Plus Settings → UI extension order** controls persistent ordering and applies changes immediately. Unlisted extensions follow deterministically; registrations within an extension follow registration order. `inspect()` reports replacement conflicts and their winning/suppressed extension IDs.

## Original and native parts

`Original` renders the target's live Spotify content with native interactions. It is valid only inside a replacement. Mount it once in its own target; duplicate mounts or moving it across targets are unsupported. Removing it or changing replacement ownership can reset native composition state. Returning `null` intentionally hides a replaced region; it does not restore Spotify. Dispose the registration to restore ownership, or render `Original` to show native content inside your layout.

Drawer and menu roots expose `context.parts`. `NativePart` renders a listed row/content region with Spotify's native icons, badges, interaction, and state. Use each current `part.id` once in its own instance. Avoid combining a full `Original` layout with duplicates of its parts. Parts are not available for arbitrary page regions. A row leased as `NativePart` uses its original renderer directly; individual header/action contributions apply inside the full `Original` menu.

```tsx
import { ScrollView, View, Text } from 'spotifyplus/react';

function Drawer({ context, NativePart }: UIComponentProps) {
  return (
    <ScrollView style={{ flex: 1 }}>
      <View style={{ width: '100%' }}>
        <Text>My drawer</Text>
        {(context.parts ?? []).map(part => (
          <NativePart key={part.id} id={part.id} />
        ))}
      </View>
    </ScrollView>
  );
}
```

A `ScrollView` takes one wrapper view containing its rows. [listInstances()](./list-instances.mdx) reads current contexts without a replacement. [invokeAction()](./invoke-action.mdx) lets a custom button dispatch a current native action even when its row is omitted.

## Recovery and diagnostics

Native content stays visible during asynchronous React startup. Render failures are isolated to contributions; failed replacements fall back to native content. Native commit failures dispose the failed host and restore Spotify. Unsupported targets/operations report an error rather than providing a new adapter.

Use [captureHierarchy()](./capture-hierarchy.mdx) and the [UI inspector](./set-inspector-enabled.mdx) to examine the layout. Use `listTargets()` for capabilities, `inspect()` for availability/operations/conflicts, and `listInstances()` for current contexts. A successful build does not verify a Spotify layout: test opening, closing, back navigation, context changes, and restoration on a supported device.
