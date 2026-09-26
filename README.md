💬 **[Join the Discord](https://discord.gg/rDUGPHn47V)**

# Ender-Proxy

**🇰🇷 한국어로 보기: [README.ko.md](README.ko.md)**

Ender-Proxy is a Minecraft Bedrock Edition (MITM) proxy client, for **Windows and Android**. It sits between your Minecraft client and a joined Xbox Live session (a friend's world or your own Realm), forwards packets transparently, and adds an in-game command system and HUD with movement, combat, world, and visual utility modules, plus an optional bridge for external tools.

Everything is driven from in-game chat using a `.` command prefix (for example `/.fly on`) or from the proxy's own console window (on Android, a plain console screen inside the app, no Termux or separate runtime needed).

> **Disclaimer:** Ender-Proxy works against **server-authoritative movement** (mandatory on current Bedrock Dedicated Servers and Xbox Live friend sessions). It steers the server's own simulation of the player instead of teleporting the client. Some modules (combat automation, movement assistance, world manipulation) may violate the rules of servers or Realms you don't own. Only use it on worlds you own, or where every player involved has agreed to it, and always at your own risk.

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Download & Install](#download--install)
- [Before You Start: Windows Loopback Exemption](#before-you-start-windows-loopback-exemption)
- [Windows: Step-by-Step Guide](#windows-step-by-step-guide)
- [Android: Step-by-Step Guide](#android-step-by-step-guide)
- [Android Notes](#android-notes)
- [Command-Line Flags](#command-line-flags)
- [Commands](#commands)
- [Optional Components](#optional-components)
- [Third-Party Components](#third-party-components)
- [Terms of Use / Disclaimer](#terms-of-use--disclaimer)
- [License](#license)

## Features

- **Xbox Live friend-session discovery**: logs in with your own Microsoft account and lists active Bedrock sessions among your Xbox Live friends, no server IP needed.
- **In-game command menu**: a HUD panel (`/.menu`) plus a full chat command set, all under the `.` prefix.
- **Movement**: `fly`, `speed`, `noclip`, `nofall`, `scaffold`, `blink` / `autoblink`.
- **Combat**: `killaura`, `multiaura`, `reach`, `autoclick`, `autototem`.
- **World**: `fastbreak`, `automine`, `nuker`, `chunkloader`.
- **Visual**: `esp`, `xray`, `fullbright`, `tracer`, in-world `video` playback, custom `skin` switching.
- **Chat tools**: spoofed and looped chat (`say`, `spam`, `faketext`).
- **External tool bridge**: a vanilla-`wsserver`-compatible WebSocket endpoint (`-ws` flag) so outside tools can attach with no in-game command needed, plus a built-in relay for `wssay` / `wsspam` / `whisperwspam` that needs no extra runtime to be installed.
- **Custom local resource pack** for the in-game HUD, applied automatically. It never touches the server's own resource packs.
- **Android app**: the same proxy, packaged as a single sideloadable APK with a plain console screen standing in for the Windows console window. No Termux, no separate Android build of Node.js or anything else to install.

## Requirements

### Windows

- Windows 10/11, 64-bit.
- A Microsoft/Xbox Live account signed in to Xbox Live, with the target world or Realm already in your friends list or owned by you. Ender-Proxy discovers sessions through Xbox Live, it does not dial arbitrary server IPs.
- FFmpeg is bundled in the release zip (`bin\ffmpeg.exe`) for the `/.video` module, so there is nothing extra to install for that.
- Nothing else to install. `wssay` / `wsspam` / `whisperwspam` are handled entirely inside `enderproxy.exe`, no Node.js or any other runtime needed.

### Android

- Android 8.0 (API 26) or newer, 64-bit ARM (`arm64-v8a`). That's effectively every real Android phone from the last several years; the handful of 32-bit-only devices still out there aren't supported yet.
- The same Microsoft/Xbox Live account requirement as Windows, above.
- Nothing to install beyond the one APK. FFmpeg (and so `/.video`) isn't available on Android yet, everything else is.

## Download & Install

Both platforms' files are on the same [Releases](../../releases) page.

### Windows

1. Grab the latest `Ender-Proxy-<version>-win64.zip`.
2. Extract it anywhere (avoid `C:\Program Files`, which Windows locks down for writing).
3. Run `enderproxy.exe`.

The extracted folder must keep its structure. `enderproxy.exe` reads `resource_packs/`, `skins/`, and `videos/` as **relative** paths next to itself, so don't move the exe out of its folder on its own.

### Android

1. Grab the latest `Ender-Proxy-<version>-android-arm64.apk`.
2. Open it from your file manager or downloads notification. Android will ask to allow installs from that source the first time, allow it.
3. Open the app. Unlike Windows, there's no separate setup step needed first, no loopback exemption command to run. Just open it.

The APK bundles the local HUD resource pack and default skins already, they're unpacked automatically the first time the app runs.

## Before You Start: Windows Loopback Exemption

Minecraft Bedrock (and Minecraft Preview) are UWP apps, and Windows blocks UWP apps from connecting to `127.0.0.1` (localhost) by default. Since Ender-Proxy runs locally and your Minecraft client connects *through* it, you need to lift that restriction once per PC before your first run. (This step is Windows-only. Android doesn't isolate loopback between apps the same way, so there's nothing equivalent to do there.)

1. Open **PowerShell as Administrator** (right-click the Start button, choose "Terminal (Admin)" or "Windows PowerShell (Admin)").
2. Run the command for your edition:

   ```powershell
   # Minecraft (retail, Microsoft Store)
   CheckNetIsolation LoopbackExempt -a -n="Microsoft.MinecraftUWP_8wekyb3d8bbwe"

   # Minecraft Preview
   CheckNetIsolation LoopbackExempt -a -n="Microsoft.MinecraftWindowsBeta_8wekyb3d8bbwe"
   ```

3. You should see `Ok.` printed. Fully close and reopen Minecraft afterward.

Useful related commands:

```powershell
# List every app currently exempted
CheckNetIsolation LoopbackExempt -s

# Undo it later (remove the exemption again)
CheckNetIsolation LoopbackExempt -d -n="Microsoft.MinecraftUWP_8wekyb3d8bbwe"
```

This is a one-time, standard Windows setting used by any local Bedrock proxy or server tool. It does not modify Ender-Proxy or Minecraft itself, and removing the exemption later has no side effects.

## Windows: Step-by-Step Guide

This section walks through everything from a completely fresh download to your first working connection. If you've never used a tool like this before, start here.

### Step 1: Apply the loopback exemption

Do this first, once per PC. Follow [Before You Start](#before-you-start-windows-loopback-exemption) above. If you skip this step, Minecraft will usually just sit at "Connecting..." and time out.

### Step 2: Extract the zip

Right-click `Ender-Proxy-<version>-win64.zip` and choose "Extract All...". Pick a normal folder such as `Documents\Ender-Proxy` or your Desktop. Don't extract it into a system folder like `C:\Windows` or `C:\Program Files`.

### Step 3: Run the program

Double-click `enderproxy.exe` inside the extracted folder. A console window opens and prints a banner. If Windows SmartScreen shows a blue warning screen (this is normal for a small, unsigned tool), click "More info" and then "Run anyway".

### Step 4: Sign in with Xbox Live

The console prints a Microsoft login link and a short code. Open that link in your browser, enter the code, and sign in with the same Microsoft account you use in Minecraft. After signing in, you can close the browser tab and return to the console window. Your login is cached in `token_cache.json` so you won't need to repeat this every time (use the `-login` flag if you ever need to switch accounts).

### Step 5: Pick a session

Ender-Proxy lists active Bedrock sessions among your Xbox Live friends. Type the number of the session you want to join, or follow the prompt if you want to target your own world/Realm instead. If nothing shows up, make sure your friend has actually opened their world to "Friends" or "Friends of Friends" and that you're both signed in.

### Step 6: Connect Minecraft to the proxy

Open Minecraft Bedrock yourself, go to **Play → Servers**, and add (or use) a server entry pointing at `127.0.0.1` on port `19132` (or whatever you passed to `-port`). Join that server entry. You should land in your friend's world exactly as if you'd joined it directly, except now the proxy is watching and relaying every packet.

### Step 7: Try your first command

In Minecraft chat, type:

```
/.help
```

You should see a list of command categories. Try something harmless first, like:

```
/.pos
```

which prints your current coordinates, or:

```
/.menu
```

which opens the in-game HUD menu. From there, explore the [Commands](#commands) table below for the full list.

### Troubleshooting

| Problem | Likely fix |
|---|---|
| Minecraft hangs on "Connecting..." or fails immediately | Redo the [loopback exemption](#before-you-start-windows-loopback-exemption) step, then fully close and reopen Minecraft. |
| No friend sessions are listed | Ask your friend to open their world to "Friends" or "Friends of Friends" in the pause menu, and confirm you're both online on Xbox Live. |
| Windows SmartScreen blocks the exe | Click "More info" then "Run anyway". This is expected for an independently distributed executable that isn't code-signed. |
| `/.video` does nothing | Make sure `bin\ffmpeg.exe` still exists next to `enderproxy.exe` (don't delete the `bin` folder), and that you put a video/GIF file into `videos/` or used a direct `http(s)://` link. |
| `wssay` / `wsspam` / `whisperwspam` don't show a message in chat | Give it a few seconds after connecting, the real client needs to finish its own handshake first. If it still doesn't work, rejoin the world so Ender-Proxy can reconnect the bridge. |
| Login link/code doesn't work | Make sure you're signing in with the same Microsoft account that owns your Minecraft/Xbox Live profile, and that your PC's clock is set correctly (a wrong system clock breaks Microsoft login). |

## Android: Step-by-Step Guide

Same idea as the Windows guide above, just through the app instead of a console window. There's no loopback exemption step on Android, that's a Windows-only requirement.

### Step 1: Install the APK

Download `Ender-Proxy-<version>-android-arm64.apk` from [Releases](../../releases) onto your phone and open it. Android will prompt to allow installing from that source the first time (Settings → allow), then installs normally.

### Step 2: Open the app

Tap the Ender-Proxy icon. It opens straight into a plain black console screen and starts the proxy immediately, no separate "start" step. If a notification permission prompt appears, allow it, that's just for the persistent "Ender-Proxy running" notification.

### Step 3: Sign in with Xbox Live

Same as Windows: a login link and short code appear on screen. Open the link in your phone's browser, enter the code, sign in with your Minecraft/Xbox Live Microsoft account, then switch back to the Ender-Proxy app.

### Step 4: Pick a session

A list of your Xbox Live friends' active Bedrock sessions appears. Type the number into the text field at the bottom of the screen and tap **Send**, the same as typing into the Windows console.

### Step 5: Connect Minecraft to the proxy

Open Minecraft Bedrock (same phone), go to **Play → Servers**, and add a server pointing at `127.0.0.1` port `19132`. Join it, same as the Windows guide.

### Step 6: Try your first command

In Minecraft chat, type `/.help`, then try `/.pos` or `/.menu`. See the [Commands](#commands) table below for the full list.

### Step 7: Stopping it

Closing the app screen, switching to Minecraft, or even swiping Ender-Proxy out of your recent apps does **not** stop it, that's intentional, so it survives while you're playing. To actually stop it, either tap **Stop** in the app, or tap **Stop** on the persistent notification. See [Android Notes](#android-notes) below.

### Troubleshooting

| Problem | Likely fix |
|---|---|
| Can't install the APK | Make sure you allowed "install from this source" when prompted. On some phones this is under Settings → Apps → Special access → Install unknown apps, per app (browser or file manager). |
| No friend sessions are listed | Same as Windows: ask your friend to open their world to "Friends" or "Friends of Friends", confirm you're both online on Xbox Live. |
| Minecraft can't connect to `127.0.0.1:19132` | Make sure the Ender-Proxy app (or its notification) shows it's still running, and that you typed the port correctly. Unlike Windows there's no loopback exemption to troubleshoot here. |
| Reopened the app and the screen is blank | Give it a second, it replays everything printed so far. If it's still blank, the proxy may have been stopped, check for the persistent notification. |
| `/.video` does nothing | Expected for now, FFmpeg isn't bundled for Android yet. |

## Android Notes

- **Stopping the proxy**: it runs as a background (foreground) service on purpose, so it isn't killed while you're alt-tabbed into Minecraft. That means there's no implicit "close the app to stop it" gesture. Use the **Stop** button in the app, or the **Stop** action on its notification.
- **Reopening the app**: everything printed so far is replayed, you won't lose the login prompt or earlier output just by switching away and back.
- **Permissions**: only Internet access and a notification (for the "running" status) are needed. Neither is a sensitive/dangerous permission requiring a special explanation dialog beyond the one-time system prompts.
- **Command-line flags** (below) aren't exposed in the Android app. It always listens on the same defaults as Windows (`19132` for the RakNet proxy, `127.0.0.1:8000` for the external tool bridge).
- **Current limitations**: `/.video` isn't available yet (no Android FFmpeg build bundled), and only 64-bit ARM (`arm64-v8a`) devices are supported.

## Command-Line Flags

Windows only, the Android app doesn't expose these (see [Android Notes](#android-notes)).

| Flag | Default | Description |
|---|---|---|
| `-login` | off | Force a fresh Xbox Live login, ignoring the cached token |
| `-logout` | off | Clear the cached token and exit |
| `-port` | `19132` | Local RakNet port the proxy listens on |
| `-ws` | `127.0.0.1:8000` | Address to auto-start a vanilla-`wsserver`-compatible WebSocket endpoint on (an empty string disables it) |

## Commands

All commands are typed as `/.name` in Minecraft chat, or just `name` (no `/.`) in the console. Type `/.help` in-game for the live list with aliases.

### General

| Command | Description |
|---|---|
| `help` | Show command help |
| `modules` | List all modules and their on/off state |
| `menu` | Open the in-game HUD menu |
| `list` | List online players |
| `pos` | Show your current coordinates |
| `seed` | Show the world seed |

### Combat

| Command | Description |
|---|---|
| `killaura` | Auto-attack the nearest enemy in range |
| `multiaura` | Attack several enemies in range at once |
| `reach` | Extend your manual attack reach |
| `autoclick` | Auto-click whatever is in your crosshair |
| `autototem` | Keep a Totem of Undying equipped |

### Movement

| Command | Description |
|---|---|
| `speed` | Accelerate your movement |
| `blink` | Choke movement packets, release to warp |
| `autoblink` | Auto-arm Blink and release it on attack |
| `fly` | Fly |
| `nofall` | Negate fall damage |
| `noclip` | Walk through walls |
| `scaffold` | Auto-place blocks under you while walking |

### World

| Command | Description |
|---|---|
| `fastbreak` | Break blocks faster by adding extra mining ticks |
| `automine` | Continuously mine whatever block is under your crosshair |
| `nuker` | Break the block you mine and the blocks around it |
| `chunkloader` | Preload chunks around you |

### Visual

| Command | Description |
|---|---|
| `esp` | Draw 3D boxes around entities |
| `xray` | Draw 3D boxes around a chosen ore/block type through walls |
| `fullbright` | See in the dark |
| `tracer` | Draw lines from you to entities |
| `skin` | Change your player's skin, visible to others on the server |
| `video` | Play a video on an in-world particle screen (needs FFmpeg, see below) |

### Chat

| Command | Description |
|---|---|
| `say` | Send a chat message to the server |
| `spam` | Repeat a chat message (default 10 times, 100 ms apart) |
| `wspam` | Shortcut for `.spam whisper` |
| `faketext` | Send a spoofed chat line |

### External Tool Bridge

| Command | Description |
|---|---|
| `wsconnect` | Connect to an external tool over Minecraft's vanilla `wsserver` protocol |
| `wsdisconnect` | Disconnect the external tool |
| `wsstatus` | Show whether an external tool is connected |
| `wssay` | Notify every connected external WebSocket tool of a message |
| `wsspam` | Repeatedly notify every connected external WebSocket tool of a message |

## Optional Components

### Video playback (FFmpeg, Windows only)

`/.video` renders a video or GIF onto an in-world particle screen by decoding it frame by frame with FFmpeg. The release zip already bundles `bin\ffmpeg.exe`, so this works out of the box. Just drop your own video/GIF files into the `videos/` folder, or pass a direct `http(s)://` URL to `/.video`.

The release zip does **not** bundle any video files, only the FFmpeg binary itself. Supply your own media so you're not redistributing content you don't hold the rights to.

Not available on the Android app yet, there's no Android FFmpeg build bundled with it.

## Third-Party Components

- [sandertv/gophertunnel](https://github.com/sandertv/gophertunnel) (vendored, patched): MIT License
- [df-mc/dragonfly](https://github.com/df-mc/dragonfly): MIT License
- [FFmpeg](https://ffmpeg.org/) (bundled prebuilt binary, `bin\ffmpeg.exe`, "essentials" build from [gyan.dev](https://www.gyan.dev/ffmpeg/builds/)): GNU GPL v3. The corresponding source is freely available from [ffmpeg.org](https://ffmpeg.org/download.html) and [gyan.dev](https://www.gyan.dev/ffmpeg/builds/). The full license text ships in `bin\ffmpeg-LICENSE.txt`.

## Terms of Use / Disclaimer

Ender-Proxy is provided **as-is**, with no warranty of any kind. By downloading or using it, you agree that:

- You are solely responsible for how you use it, including which worlds, Realms, or servers you connect to and which modules you enable.
- The author(s) are **not liable** for any account suspensions or bans, lost items or progress, world damage, or any other loss or damage arising from your use (or misuse) of this software.
- You are responsible for complying with Mojang/Microsoft's [Minecraft End User License Agreement](https://www.minecraft.net/en-us/eula) and the rules of any server, Realm, or world you connect to.
- This project is unofficial and is not affiliated with, endorsed by, or sponsored by Mojang Studios or Microsoft.

## License

Released under the [MIT License](LICENSE). Third-party components keep their own licenses, see [Third-Party Components](#third-party-components) above.
