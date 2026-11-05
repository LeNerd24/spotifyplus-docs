---
sidebar_label: 'Publishing Your Extension'
sidebar_position: 2
---

Once you finish your extension, you probably want to publish it for others to use. This will go over how to publish it!

# Publishing Your Extension

To publish your extension to the marketplace, you need to have a GitHub repo for it. So you need to create a GitHub repo with your main extension file, and a `manifest.json`. Your manifest will tell the marketplace how/what to install, and extra information about it.

In order for your extension to show up in the maretkplace, you must give your repo the `spotifyplus-extensions` topic, and you must have a valid `manifest.json`.

# Example Manifest

Below is an example of a full `manifest.json`.

```json title="manifest.json"
{
  "id": "com.lenerd.bookmarks",
  "name": "Bookmarks",
  "description": "Allows you to save songs, albums, and artists for later!",
  "version": "1.0",
  "author": [
    {
      "name": "LeNerd46",
      "url": "https://www.github.com/LeNerd46"
    }
  ],
  "tags": ["bookmarks", "library"],
  "preview": "screenshot.png",
  "banner": "banner.png",
  "api": 2,
  "assets": ["assets/**/*"],
  "main": "dist/index.js",
  "native": {
    "apk": "native.apk",
    "pluginClass": "com.lenerd.bookmarks.NativePlugin"
  }
}
```

# Manifest Properties

This is a list of all properties available in `manifest.json`

| Property      | Type             | Description                                               | Required |
| :------------ | :--------------- | :-------------------------------------------------------- | :------- |
| `id`          | `string`         | A unique identifier for your extension                    | ✓        |
| `name`        | `string`         | The name of your extension                                | ✓        |
| `description` | `string`         | A description of your extension                           |       ✓   |
| `version`     | `string`         | What version your extension is on                         | ✓        |
| `author`      | `Author[]`       | A list of authors who worked on this extension            |     ✓     |
| `tags`        | `string[]`       | Tags used to categorize your extension in the marketplace  |  ✓        |
| `api`         | `number`         | What version of the Spotify Plus API this extension uses  | ✓         |
| `main`        | `string`         | Path to the main JavaScript file                          | ✓        |
| `preview`     | `string`         | Path to image used for a preview image in the marketplace |          |
| `banner`      | `string`         | Path to image used for a banner image in the marketplace  |          |
| `assets`      | `string[]`       | An array of assets used by your extension                 |          |
| `native`      | `NativeMetadata` | Information about the native code your extension uses     |          |
| `targetSpotifyVersion` | `string` | The Spotify version or range your native code targets | |

`targetSpotifyVersion` is informational only. Spotify Plus does not check if the version matches before installing. It will only show a warning in the marketplace. This is mainly useful if you are hooking native Spotify code yourself, instead of the officially supported TypeScript API.

### Author

| Property | Type     | Description                                   | Required |
| :------- | :------- | :-------------------------------------------- | :------- |
| `name`   | `string` | The name of the author                        | ✓        |
| `url`    | `string` | A URL to the user's GitHub (or other) profile | ✓        |

### Native Metadata

| Property      | Type     | Description                        | Required |
| :------------ | :------- | :--------------------------------- | :------- |
| `apk`         | `string` | Path to the APK file               | ✓        |
| `pluginClass` | `string` | Full name of the entry point class | ✓        |

If you publish a new GitHub release, the markdown contents of that release will appear in the marketplace as a change log. This is optional, but it can give extra information about your extension and what has changed. The marketplace will not download anything from your releases. It will only ever download from the files in your repo.

## Assets and Updates

Spotify Plus will check if your extension has an update every time the user opens the marketplace. It will only trigger an update if the main JavaScript file has changed. Simply changing an asset does not count as an update. Making a new GitHub release does not trigger an update either. The marketplace will display whatever markdown content you put in the latest release as a changelog, but it does not count as an update. Spotify Plus will see an update if your main JavaScript file changed. So only build and publish your built extension to GitHub if you are ready for it to be an update.