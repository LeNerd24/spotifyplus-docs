---
sidebar_position: 1
sidebar_label: 'Hot Reload'
---

# Hot Reload for Extensions

Yeah, having to build your extension, send it to your phone, and move it to the right location every time to test a simple change would be quite annoying. Don't worry, Spotify Plus supports hot reloading! Whenever you're developing your extension, just run `npm run dev`, and it will automatically send your extension to your phone, and any time you make a change, it will automatically reload your extension. Files declared in `manifest.assets` are included with the reload, up to 20 MB in total. If you use native code, the CLI also builds and sends the APK when the installed host supports native hot reload.

## Getting Hot Reloading Working

There are a few steps you must take before hot reloading will work. Don't worry though! You might already have a lot of this done already. First, make sure you have developer mode enabled in Spotify Plus. Go to Spotify Plus Settings -> Developer -> Enable developer mode. 

Next, make sure you have adb installed. This is pretty simple, just look up the download link or whatever. You know how to install things on your computer. Then, you have to connect your phone to your computer. If you are using an emulator, you can skip this step. If you are using your real phone, make sure you have developer mode enabled on your phone, and debugging is enabled. If you plug your phone into your computer with a USB cable, it will ask you on your phone if you want to allow USB debugging. Please allow this. If you want to do it wirelessly, enable wireless debugging and pair your phone to your computer. If you need help with this, please see [this](https://medium.com/fludev/enable-wireless-debugging-in-android-ditch-the-usb-cable-74575d0da2f7) article.

You should be all set then! Run `npm run dev` with your device connected, and you should see your extension show up!
## CLI options

You can also run `npx spotifyplus dev` directly from your extension directory. If you have multiple devices connected, choose one with `--device`:

```sh
npx spotifyplus dev --device DEVICE_SERIAL
npx spotifyplus build --minify --sourcemap
```

`dev` accepts one extension directory, ID, or workspace folder selector. `build` can select multiple extensions. `--entry` chooses an entry file, `--adb` chooses the adb executable, `--port` changes the local dev port (default 37846), and `--debounce-ms` controls rebuild debounce (default 150). `--outfile` is only available for builds. Run `npx spotifyplus --help` for the command options.

## Native reloads

The CLI checks the phone's native hot-reload capability before sending an APK. Native APKs have a separate 32 MB limit. If the host does not support it, update Spotify Plus first. Reloading unloads the old extension generation, including its hooks, listeners, UI registrations, and pending work. Release any resources your plugin creates in `onUnload()`.

Hot reload can update your extension. Changes to Spotify Plus's own native APIs still require an updated host APK and restarting Spotify.
