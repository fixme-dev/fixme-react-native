# FIXME for React Native and Expo

The React Native and Expo helper for [FIXME](https://getfixme.dev), the Mac app that turns
"this looks wrong" into a fix. Tap your app with three fingers, circle what's wrong, say what you see, and
your coding agent on the Mac (Claude Code, Codex, Cursor and others) gets the exact file and line, a
screenshot, the UI tree, console, network requests and a short clip.

It runs only in development, and the overlay is FIXME's own native one, the same you get in a Swift or Kotlin app, with your
React components named in the ticket. That needs a development build: Expo Go cannot load native code, so there it does nothing
and says so in one line. In production builds every export is a no-op and the native part is not linked at all.

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
"devDependencies": { "fixme-react-native": "https://github.com/fixme-dev/fixme-react-native/releases/download/0.1.5/fixme-react-native-0.1.5.tgz" }
```

After that, `npx fixme-react-native undo` removes everything `init` added. To add the dependency by hand, put that line in `package.json` and run your package manager's install,
then `npx fixme-react-native init`. (`npx expo install <url>` can't take a URL: install it with your package manager.)

The one line it adds, as the first statement of your entry file:

```js
if (__DEV__) require('fixme-react-native/auto');
```

For an Expo Router app (`"main": "expo-router/entry"`) it writes `index.js` that loads the helper first and then
`expo-router/entry`, and points `"main"` at it (`undo` puts `"main"` back and removes the file); the root layout is
evaluated too late to wrap the app.

**Expo:** `init` also adds `"fixme-react-native"` to the plugins in `app.json` (an app with `app.config.js` adds it there by hand). Run
`npx expo run:ios` or `run:android` (or build a development build with EAS) to get the overlay: it is native code, so a reload does
not add it. The plugin adds the Bonjour, Local Network and microphone texts iOS needs; pass `{ "voice": false }` to leave the
microphone and speech texts out. **Expo Go** cannot load native code: the helper prints `FIXME needs a development build: npx expo
run:ios / run:android` and does nothing else.

**Bare React Native:** `init` adds the same Info.plist texts and runs `pod install` (Android links by autolinking). The native part
is linked into Debug builds only, so a Release build contains no FIXME code. Rebuild the app once afterwards.

## Using it

1. Start your app in development on a phone, simulator or emulator, with FIXME open on your Mac on the same
   Wi-Fi (Android over USB works too).
2. The first time, allow the phone on your Mac.
3. Tap with three fingers, circle what's wrong, and send. Tap the mic to say what you see (tap it again to stop),
   or tap Type; each circle keeps its own note. Clip last 30 s and Record are on the overlay too.

The helper's own part is small: when you send, the overlay asks "what do you know about these circles?" and this package answers
with the React component, its file and line, the console, the screen and the UI tree. From code: `Pointer.capture()`,
`Pointer.clip()`, `Pointer.toggleRecording()`, `Pointer.setScreen(name)`, `Pointer.setState(key, value)` and
`Pointer.trackNavigation(navigationRef)` (types in `index.d.ts`). The dev menu has "mark this screen", "clip last 30s" and "start or
stop recording".

## Requirements

React Native 0.72 or later, or Expo SDK 47 or later, in a development build. FIXME for Mac. The iOS overlay needs iOS 16 or
later; the Android one API 24 or later. Both are linked into development builds only.

## Privacy

The overlay talks only to your own Mac, on your local network or through FIXME's end-to-end encrypted relay. This package itself
opens no connection: it only answers the overlay, and asks your own Metro server (`/symbolicate`) for file and line.
Nothing goes to any other server.

## License

See [LICENSE](LICENSE). Free to use in development builds alongside FIXME.
