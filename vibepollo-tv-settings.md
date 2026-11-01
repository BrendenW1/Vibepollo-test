# Vibepollo + Artemis: TV Streaming Settings Cheat Sheet

Setup: wired PC running Vibepollo, TV on Wi-Fi running Artemis. This is for streaming at home only.

> **Rule of thumb:** set resolution, frame rate, bitrate, codec and HDR **in Artemis on the TV**. The Vibepollo defaults are built to follow whatever the client asks for. Only change host settings to fix a specific problem.

---

## 1. Do these first (biggest impact)

| Where | Setting | Why |
|---|---|---|
| TV | **Game Mode / "PC" input mode** | TV image processing is often the largest source of lag (20–80 ms). |
| Network | **Ethernet to the TV**, or 5/6 GHz Wi-Fi near the router | Removes Wi-Fi jitter. Even a 100 Mbps TV Ethernet port is plenty for HEVC. |
| Artemis | **Performance overlay** on while tuning | Shows decode time, network latency and dropped frames, so you can see whether a change helped. Turn it off afterwards. |

---

## 2. Artemis (client on the TV)

Labels may differ slightly between Artemis versions.

| Setting | Recommended | Notes |
|---|---|---|
| **Resolution** | TV native (usually 4K) | Vibepollo creates a virtual display at exactly this size. Try 1440p if 4K over Wi-Fi stutters. |
| **Frame rate** | 120 if the TV supports it, otherwise 60 | 120 delivers frames sooner even when a game runs at 60. |
| **Bitrate** | 40–80 Mbps (4K, HEVC) | Higher isn't better on Wi-Fi. Too much bitrate causes jitter that feels like lag. Lower it if the overlay shows dropped frames. |
| **Video codec** | HEVC (or AV1 if the TV decodes it well) | Avoid H.264 at 4K. Required for HDR. |
| **HDR** | On, if the TV and game support it | Vibepollo follows this toggle automatically. |
| **Frame pacing** | Try both "lowest latency" and "smoothest/balanced" | One gives slightly less lag, the other more even motion. Pick what feels better. |
| **Surround sound** | 5.1/7.1 if you have a soundbar/AVR | Needs the host's "Install Steam Audio Drivers" setting on (it is by default). |
| **Use virtual display** | On | Matches Vibepollo's default. Your PC monitors go dark during the stream. |
| **Gamepad / controller options** | Defaults | Vibepollo auto-picks Xbox or DualSense emulation based on your controller. |

**Skip:** PyroWave. It needs 220–300+ Mbps on a wired link and Nonary's desktop Moonlight fork, so it isn't practical on a Wi-Fi TV.

---

## 3. Vibepollo (host web UI: `https://<pc-ip>:47990`)

### Leave at default unless there's a problem
| Setting (location) | Default | Change only if… |
|---|---|---|
| **Streaming display** (Settings → Everyday setup) | Virtual display per client | You use a physical monitor or dummy plug instead. |
| **How it joins the desktop** | Virtual display only | Someone needs to use the PC while you stream (pick "Extend"). |
| **Screen resolution / refresh rate** | Match the client | Never. Set these in Artemis. |
| **HDR / HDR requested by client** | Auto | The TV or a game reports HDR wrongly (Force on/off). |
| **Video encoder** | Auto | You have more than one GPU and it picks the wrong one. |
| **HEVC / AV1 support** (Video & quality) | Auto | The TV's AV1 decoding struggles (disable AV1). |
| **Maximum bitrate** | 0 (client decides) | You want a hard cap no matter what the client asks for. |
| **Network recovery (FEC)** | 20% | Wi-Fi packet loss causes blocky artifacts (try 30–50%). |
| **Smoother game capture** | Enabled | A game misbehaves (try "Compatibility" or "Off"). VRR needs Nonary's client. |
| **Emulated gamepad type** (Input) | Auto | A game needs a specific controller type. |

### Worth a deliberate choice
| Setting | What it does | Suggestion |
|---|---|---|
| **Restore as soon as the client disconnects** | Off: game and virtual display stay alive about 2 h so you can resume. On: monitors come back immediately. | Your preference. |
| **End all streams hotkey** (Windows) | Keyboard shortcut on the PC to kill a stuck stream. | Handy to set. |
| **Steam library sync** / **Playnite** (Integrations) | Puts your installed games in the Artemis app list automatically. | Use Playnite for multi-store launching. Set Playnite to start with Windows. |
| **Device → Display mode override** (Devices → Edit) | Forces one mode for one client, e.g. `3840x2160x120`. | Use this instead of changing global resolution settings. |
| **Device → Allow this device to put the PC to sleep** | Lets Artemis sleep the PC when nothing is streaming. | Optional. Off by default. |

### Security settings (quick wins)
| Setting | Suggestion |
|---|---|
| **Web UI password** | Long and random. Reset from an admin terminal with `vibepollo --creds <user> <pass>`. Then revoke old sessions and API tokens in Maintenance, because a reset doesn't log anyone out. |
| **Origin Web UI Allowed** (Network & security) | `lan` (default) is fine. `pc` blocks other devices from the settings page, but then pairing PINs must be entered on the PC. Streaming is unaffected either way. |
| **UPnP** | Leave off. Don't forward ports. Use Tailscale if you ever want remote play. |
| **Devices → Client permissions** | The first paired device gets every permission. Turn off clipboard and server commands for the TV if you don't use them. |
| **Update notifications** (Maintenance) | Consider stable-only. Prereleases are on by default and daily builds use unsigned drivers. |
| **Log level** | Keep at `info`. Never leave it on `verbose`, which logs stream keys. |

---

## 4. Safe to ignore

Encoder presets (`nvenc_*`, `amd_*`, `qsv_*`), PyroWave, RTX HDR, capture method, display targeting and dummy-plug options, resolution remapping, Remote Monitor / Remote Input, frame limiter / RTSS / Lossless Scaling (handled automatically), keyboard/mouse/pen tweaks, ports and network internals, Files & certificates, API tokens, Browser stream.

---

## 5. Quick troubleshooting

| Symptom | Try |
|---|---|
| Feels laggy | TV Game Mode → Ethernet → 120 fps in Artemis → test frame-pacing options. |
| Blocky artifacts / stutter | Lower bitrate in Artemis → raise FEC to 30–50% → move to 5/6 GHz or Ethernet. |
| Washed-out or dark HDR | Toggle HDR in Artemis. On the PC, run the Windows HDR Calibration app through the stream. |
| PC monitors stuck dark after streaming | Maintenance → Golden display recovery, or turn on "Restore as soon as the client disconnects". |
| Stream won't start | Check **Logs** in the web UI first. |
