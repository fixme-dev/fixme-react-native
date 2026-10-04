# FIXME for React Native and Expo

The React Native and Expo helper for [FIXME](https://getfixme.dev), the Mac app that turns
"this looks wrong" into a fix. Tap your app with three fingers, circle what's wrong, say what you see, and
your coding agent on the Mac (Claude Code, Codex, Cursor and others) gets the exact file and line, a
screenshot, the UI tree, console, network requests and a short clip.

It runs only in development. On iOS a development build gets FIXME's own native overlay, the same one FIXME gives a
Swift app, with your React components named in the ticket; in Expo Go (and on Android for now) a JavaScript overlay
does the same job. In production builds every export is a no-op and the native part is not linked at all.

## Install

FIXME's helper is not on npm. It ships as a tarball attached to a release on GitHub
([fixme-dev/fixme-react-native](https://github.com/fixme-dev/fixme-react-native/releases)), and your project depends on that
file's URL.

**The easy way:** open FIXME on your Mac and choose **Add FIXME to your app** in onboarding (or Settings ›
General › On your phone). FIXME shows the change first, installs the release for the version it expects, and you can undo it
any time.

**From a terminal**, once, in your app's folder:

```sh
npx --package https://github.com/fixme-dev/fixme-react-native/releases/latest/download/fixme-react-native.tgz fixme-react-native init
```

`init` installs that release as a dev dependency with your package manager (npm, yarn, pnpm or bun, from your lockfile), and only
when that worked it changes your app. Your `package.json` then names the release by version, so teammates and CI install the
same file, and your lockfile records its checksum:

```json
"devDependencies": { "fixme-react-native": "https://github.com/fixme-dev/fixme-react-native/releases/download/0.1.2/fixme-react-native-0.1.2.tgz" }
```

After that, `npx fixme-react-native undo` removes everything `init` added (and `npx fixme-react-native init --optional` adds the
recommended extras below). To add the dependency by hand, put that line in `package.json` and run your package manager's install,
then `npx fixme-react-native init`. (`npx expo install <url>` can't take a URL: install it with your package manager.)

The one line it adds, as the first statement of your entry file:

```js
if (__DEV__) require('fixme-react-native/auto');
```

For an Expo Router app (`"main": "expo-router/entry"`) it writes `index.js` that loads the helper first and then
`expo-router/entry`, and points `"main"` at it (`undo` puts `"main"` back and removes the file); the root layout is
evaluated too late to wrap the app.

**Expo:** `init` also adds `"fixme-react-native"` to the plugins in `app.json` (an app with `app.config.js` adds it there by hand). Run `npx expo run:ios` (or build a
development build with EAS) to get the native overlay. The plugin adds the Bonjour, Local Network and microphone texts
iOS needs; pass `{ "voice": false }` to leave the microphone and speech texts out. For screenshots in a development
build, also run `npx expo install react-native-view-shot`. Expo Go cannot load native code: it uses the JavaScript
overlay, which already has screenshots.

**Bare React Native:** `init` adds the same Info.plist texts and runs `pod install`. The native part is linked into the
**Debug** configuration only, so a Release build contains no FIXME code.

## Using it

1. Start your app in development on a phone, simulator or emulator, with FIXME open on your Mac on the same
   Wi-Fi (Android over USB works too).
2. The first time, allow the phone on your Mac.
3. Tap with three fingers, circle what's wrong, and send. Tap the mic to say what you see (tap it again to stop),
   or tap Type; each circle keeps its own note. Shake, or say "FIXME, clip that", to send the last 30 seconds as a clip.

## Optional extras

Each is detected at runtime; without it the helper still works and says what you're missing. (On iOS in a development build the native overlay does the circling, the mic and the clips itself; these matter for the JavaScript overlay.)

| Package | Adds |
|---|---|
| `react-native-view-shot` | screenshots, crops and clips |
| `expo-speech-recognition` or `@react-native-voice/voice` | tap the mic to say what's wrong, and "FIXME, clip that" |
| `expo-file-system` or `@react-native-async-storage/async-storage` | tickets kept while your Mac is away |
| `expo-secure-store` or `react-native-keychain` | the pairing key kept in the Keychain |
| `react-native-get-random-values` (bare apps; Expo apps already have a source) | pairing with your Mac from the JavaScript client (Android, Expo Go): Hermes has no secure random numbers of its own |
| `react-native-safe-area-context` | the JavaScript overlay keeps clear of the keyboard and the system bars on Android |
| `react-native-svg` | smoother ink |
| `expo-haptics` or `react-native-haptic-feedback` | haptics |

## Requirements

React Native 0.72 or later, or Expo SDK 47 or later. FIXME for Mac. The native overlay needs iOS 16 or later (an older
iPhone uses the JavaScript overlay); it is linked into Debug builds only.

## Privacy

The helper talks only to your own Mac, on your local network or through FIXME's end-to-end encrypted relay.
Nothing goes to any other server.

## License

See [LICENSE](LICENSE). Free to use in development builds alongside FIXME.
