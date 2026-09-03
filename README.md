# Hearth

A cozy local media center for your own videos and music. Point it at files on
your computer, then watch and listen. Your library, thumbnails, and playback
position are saved between sessions. Nothing is uploaded anywhere.

## 1. Run it while developing

You need **Node.js** first (which includes `npm`). Get it from https://nodejs.org
(the "LTS" version). Check it worked by running `node -v` in a terminal.

From inside this folder:

```bash
npm install     # downloads Electron the first time (a few minutes)
```

## 2. Make a real app you can pin to the taskbar

When you want a proper installable app with the Hearth icon (no `npm start`):

```bash
npm run dist
```

This builds an installer into a new `dist/` folder. On Windows you'll get
`dist/Hearth-Setup`. Run it once to install Hearth; it creates a
Start Menu and desktop shortcut carrying the flame icon. Right-click that
shortcut and choose **Pin to taskbar**, and you're done.

Build on the operating system you're targeting: make the Windows `.exe` on
Windows, the Mac `.dmg` on a Mac. The first `npm run dist` downloads some
packaging tools, so it needs an internet connection.

The icon is already wired up — `build/icon.ico` (Windows) and `build/icon.png`
(Mac/Linux) are set in `package.json` under `"build"`.

### If `npm run dist` fails with "A required privilege is not held by the client"

This is a Windows permission issue, not a bug in the app. electron-builder
unpacks a code-signing toolset that contains symlinks, and Windows only lets you
create symlinks with elevated rights. Fix it once, either way:

- **Turn on Developer Mode** (permanent): Settings → Privacy & security → For
  developers → Developer Mode → On. Then reopen your terminal.
- **or run the terminal as Administrator** (right-click → Run as administrator).

Then clear the half-unpacked cache and rebuild:

```powershell
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\electron-builder\Cache\winCodeSign"
npm run dist
```

`npm run pack` builds an unpacked app into `dist/win-unpacked/` (a runnable
`Hearth.exe` with no installer) and is handy for a quick test.

## How it's put together

| File | What it does |
| --- | --- |
| `main.js` | The desktop process: window, file dialog, saving, menu removal, icon. |
| `preload.js` | A safe bridge giving the UI exactly three abilities: pick, load, save. |
| `renderer/index.html` | The page structure. |
| `renderer/styles.css` | The look — theme colours and animations. |
| `renderer/app.js` | UI logic: library grid, delete, thumbnails, player, volume, fullscreen. |
| `build/` | App icons used when packaging. |

The library is stored as `library.json` in your OS's app-data folder (Electron's
`userData` path), so it survives restarts. Removing an item from the library
does **not** delete the file from your disk.

## Notes

- Playback uses the built-in HTML5 player, which covers MP4 (H.264/AAC) and MP3
  out of the box. Formats like MKV may not play until you add a player library.
- Volume is remembered while the app is open; it resets to full on restart.

## The download link on your website

The site's Download button points at a single stable URL:

```
https://github.com/lumo1975/Hearth/releases/latest/download/Hearth-Setup.exe
```

GitHub redirects `/releases/latest/download/<file>` to the newest release, so this
link never needs editing — as long as the installer attached to each release is
named `Hearth-Setup.exe`. The build config handles that with
`"artifactName": "Hearth-Setup.${ext}"`, so `npm run dist` outputs
`dist\Hearth-Setup.exe`. Attach that file to each GitHub release and make sure the
release is marked "Latest."
