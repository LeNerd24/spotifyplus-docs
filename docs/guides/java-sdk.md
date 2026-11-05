---
sidebar_position: 5
sidebar_label: 'Using the Java SDK'
---

# Using the Java SDK

If your extension already has native code, you can call Spotify Plus directly from Java! You can control playback, read metadata, listen for events, and install hooks. Start with [Writing native views](./native-ui.md) if you still need to set up the native project.

## Get the API

The context passed to your plugin gives you the API, events, and hook scope.

```java
import com.lenerd.spotifyplus.sdk.SpotifyPlusApi;
import com.lenerd.spotifyplus.sdk.SpotifyPlusContext;
import com.lenerd.spotifyplus.sdk.SpotifyPlusPlugin;
import com.lenerd.spotifyplus.sdk.SpotifyPlusRegistry;

public final class NativePlugin implements SpotifyPlusPlugin {
    @Override
    public void register(SpotifyPlusRegistry registry, SpotifyPlusContext context) {
        SpotifyPlusApi api = context.getApi();
        api.Search.search("Daft Punk").thenAccept(results -> api.log(results.toString())).exceptionally(error -> { api.error(error.toString()); return null; });
    }
}
```

Use the SDK as `compileOnly`. Spotify Plus provides it at runtime. If you use DexKit, that should also be `compileOnly`; do not package another copy into your APK.

```kotlin
dependencies {
    compileOnly(files("$rootDir/lib/spotifyplus-sdk.aar"))
    compileOnly("org.luckypray:dexkit:2.0.0")
}
```

You need a matching SDK and host for new APIs. The native project generator downloads the published SDK. When developing against an unreleased host, build `:spotifyplus-sdk:assembleRelease` in the Spotify Plus repository and copy its AAR into your extension's `native/lib/spotifyplus-sdk.aar`.

## API groups

Most groups use the same names as TypeScript. JSON responses use the same field names. `context.getPlayer()` and `context.getApi().Player` expose the same player interface; its existing `getCurrentTrack()` returns `SpotifyTrack`.

| Group | Methods |
| --- | --- |
| Player | play, pause, togglePlay, skipNext, skipPrevious, seek, getCurrentTrack, getProgress, getPlaybackPosition, getState, playContext, setShuffle, toggleShuffle, setRepeat, cycleRepeat |
| Search | search(query), search(query, limit, locale) |
| Connect | getDevices, getCurrentDevice, transfer |
| Playlists | get, create, delete, move, addTracks, removeTracks, moveTracks |
| Library | list, save, remove, contains, like, unlike, isLiked |
| Queue | get, add, remove, move, clear |
| UpNext | get, getExpanded, replace, replaceExpanded, add, remove, move, clear |
| User | getCurrent |
| Internal | getTrack, getAlbum, getArtist, getPlaylist |
| Settings | list, get, set, toggle, registerSetting, registerSettings, test |
| Navigation | open, openSpotify, openExternal, back |
| Platform | getPlatformData, getSession, Clipboard, Storage |
| Clipboard | readText, writeText, clear |
| Storage | set, get, remove, write, read, delete, Cache.write/read/delete |
| Assets | resolve, image, font, fontFamily, readText, readJson, readBytes |
| UI | replace, replaceParts, insertBefore, insertAfter, overlay, inspect, listTargets, listInstances, invokeAction |
| ContextMenu | register, open, openNowPlaying |
| SideDrawer | register, open |
| Surfaces | register, close |

The Java facade does not yet have a dedicated method for every TypeScript addition. `api.requestRuntime("Theme.get")`, for example, calls a public method through the extension's runtime and returns a future. Elevated operations still require the extension's existing elevated trust.

Local player controls, settings values, Queue, ordinary UpNext edits, Connect getters, clipboard, storage, and local library/playlist edits return synchronously. Search, metadata, profile, library/playlist reads, expanded UpNext reads, transfer, context playback, UI, assets, and extension-setting registration return `CompletableFuture`.

Do not wait for a future on Android's main thread, inside a hook, or during `register()`. The runtime cannot answer Java-to-TypeScript requests while it is still executing your registration. Use `thenAccept`, `thenApply`, and `exceptionally` instead.

Queue index edits do not take snapshots. `api.UpNext.add(0, uri)` inserts first and returns a `JSONArray` of new occurrences. `getExpanded()` defaults to 10,000; `getExpanded(int limit)` accepts 1–100,000. Reorder an ordinary snapshot with `replace(revision, orderedUids...)`, or an expanded snapshot with `replaceExpanded(revision, snapshotId, orderedUids...)`. The same [snapshot lifetime and revision rules](../spotifyplus-api/up-next/replace.mdx) apply.

Playlist removal and movement use row IDs, not track URIs. File storage supports strings, JSON-compatible values, and `byte[]`. Storage is scoped to your extension ID and shared with its TypeScript half.

## Events

```java
context.getEvents().on("songChanged", payload -> {
    org.json.JSONObject event = (org.json.JSONObject) payload;
    context.getApi().log(event.optString("uri") + " / " + event.optString("reason"));
});
```

`context.getEvents()` has `on`, `once`, `off`, and `emit`. Spotify events reach native extensions too. Custom events share the same extension's TypeScript emitter. Listener failures are logged independently.

Spotify events run on the publishing thread; runtime replies and custom events normally run on the Node bridge thread. Move Android view updates onto the UI thread. Use `api.onAndroidBack(event -> event.preventDefault())` for cancellable Back events and close the returned subscription to unsubscribe. Cancellation must happen synchronously.

## Hooks

`context.getHooks()` gives you class lookup, reflection helpers, and hooks scoped to your extension.

```java
import com.lenerd.spotifyplus.sdk.hooks.SpotifyPlusHooks;
import java.lang.reflect.Method;

SpotifyPlusHooks hooks = context.getHooks();
Class<?> target = hooks.findClass("com.spotify.example.Target");
Method method = hooks.findMethod(target, "someMethod", String.class);
hooks.before(method, call -> {
    call.getArgs()[0] = "replacement";
});
```

The class and method above are placeholders. Resolve the actual target for the Spotify version you support. `getClassLoader()` returns Spotify's loader.

| Method | Use |
| --- | --- |
| findClass, findMethod, findConstructor, findField | Resolve classes and members; method/field lookup can find inherited private members |
| hook, hookMethod, hookConstructor | Install callbacks on a member |
| hookAllMethods, hookAllConstructors | Hook declared overloads or constructors; installation rolls back if one fails |
| before, after | Install a lambda callback |
| invokeOriginal | Invoke a method while bypassing installed hooks |
| getDexKit | Access the shared DexKit bridge |

A `HookCallback` can implement both `before(HookCall)` and `after(HookCall)`. The call exposes `getMember()`, `getThisObject()`, mutable `getArgs()`, `getExtras()`, `getResult()`, and `getThrowable()`. `getThisObject()` is null for static methods.

Use `returnAndSkip(value)` or `throwAndSkip(error)` in a before callback to skip the original. `setResult(value)` alone does not skip it. In an after callback, use `setResult` to replace or recover a result, or `setThrowable` to replace the error. Result and throwable setters are mutually exclusive. After callbacks run for thrown and skipped calls as well.

Each invocation has separate call state, including recursive and concurrent calls. Callbacks run on the target's thread. Higher priorities run first; the default is 50. Callback failures are logged and argument/result/throwable/skip changes are rolled back. Changes inside referenced objects cannot be rolled back. Spotify's exceptions propagate unless your hook recovers them. ART and libxposed restrictions still apply.

The returned `HookHandle` has `unhook()` and `close()`. Both are idempotent, and unloading removes the hook automatically.

## DexKit

You can search the loaded APK using the shared DexKit bridge:

```java
import org.luckypray.dexkit.query.FindMethod;
import org.luckypray.dexkit.query.matchers.MethodMatcher;

Method resolved = hooks.getDexKit().findMethod(FindMethod.create().matcher(MethodMatcher.create().usingStrings("distinctive string"))).single().getMethodInstance(hooks.getClassLoader());
hooks.after(resolved, call -> {
    context.getApi().log(String.valueOf(call.getResult()));
});
```

Use Spotify's class loader when resolving results. The bridge belongs to Spotify Plus, so do not close it. You do not need to add an Xposed dependency to your extension.

## Native UI and cleanup

Register your component through `registry.registerComponent(...)`, then use its name in `api.UI.insertAfter(target, componentName)` or another supported operation. These return futures containing a registration with `dispose()`.

`UI.replace(target, componentName, true)` places the live original inside your component; expose a `ViewGroup` through `getChildren()` for it. `replaceParts()` places live native parts in that container. Component prop updates include the live instance context. Target support, ordering, conflicts, and instance lifetimes are shared with TypeScript.

Context menu registration supports a synchronous visibility predicate, disabled state, entity types, and an optional asset icon. `SideDrawer.register(name, componentName, icon, onClick)` opens a native component as a scripted drawer. Keep visibility predicates and Back handlers quick because they run on the UI thread.

Hooks, listeners, pending futures, components, menus, surfaces, and UI registrations belong to the extension generation. Reloading removes them and revokes the old context. Override `onUnload()` to release your own resources; context operations are already revoked when it runs.
