# Laptop Client Test Plan: Nonary's Moonlight (vrr18) + Vibepollo

**Goal:** find out whether a stronger client gives faster decoding and smoother play than Artemis on the TV, before buying a mini PC or Fire TV.

| | |
|---|---|
| **Host** | Windows 11, RTX 5070, Vibepollo 2.0.4, wired |
| **Client** | Lenovo 14", Intel Core Ultra 7 258V, Arc 140V, gigabit USB-C Ethernet |
| **Display** | TV over HDMI, laptop lid closed |
| **Client app** | Nonary's moonlight-qt fork, release **v6.1.0-vrr18** (Sept 28, 2026). Stay on stable; the `v6.2.0-daily.*` builds are test builds. |
| **Baseline** | Artemis on the TV: 1440p60, HEVC, 35 Mbps, Wi-Fi, decode about 9 ms |

---

## 1. One-time setup

### Laptop
- [ ] **Lid action:** Control Panel → Power Options → "Choose what closing the lid does" → **Plugged in: Do nothing**.
- [ ] **Charger plugged in.** Windows power mode set to **Best performance**.
- [ ] **Intel Graphics Software:** turn off **Display Power Savings** (Panel Self Refresh), a common cause of small stutters.
- [ ] **Lenovo Vantage:** set it to performance mode, not battery saver.
- [ ] **Gigabit Ethernet adapter plugged in.** Close the lid only after Windows switches to the TV as the only screen.

### TV
- [ ] **Game Mode / PC mode on the HDMI input the laptop uses.** TV picture settings are usually per input, so the Game Mode you set for Artemis may not apply here.

### Vibepollo (web UI, v2 view)
- [ ] Pair the laptop in Moonlight and enter the PIN in Vibepollo.
- [ ] **Devices → laptop:** turn on **Launch applications**, plus **Controller**, **Keyboard** and **Mouse** input. New devices only get view and list permissions, which causes a 403 error when you launch a game.

---

## 2. Moonlight settings for the first test (matches Artemis)

| Setting | Value |
|---|---|
| Resolution | **1440p**. Native (4K) comes later. |
| FPS | **60**. The TV is most likely a fixed 60 Hz panel. |
| Bitrate | **35 Mbps**. Moonlight resets the slider whenever you change resolution or codec, so check it every time. |
| Video codec | **HEVC** |
| Video decoder | Automatic (hardware) |
| V-Sync | On |
| Frame pacing | On |
| VRR | **Off**. It does nothing on a fixed 60 Hz TV. |

**Shortcuts during a stream**

| Keys | What it shows |
|---|---|
| **Ctrl+Alt+Shift+S** | Stats: decode time, network latency, dropped frames. |
| **Ctrl+Alt+Shift+F** | Frametime graph (fork only). Flat lines mean smooth. |

---

## 3. Test order

Change **one thing at a time**. Play the same game section in each test.

| # | Test | Settings | What it tells you |
|---|---|---|---|
| 1 | **Baseline match** | 1440p60, HEVC, 35 Mbps | Laptop vs TV decoding with everything else the same. Expect roughly 1–3 ms decode vs about 9 ms. |
| 2 | **More bitrate** | 1440p60, HEVC, 80–100 Mbps | Whether the TV's Wi-Fi was holding you back. |
| 3 | **AV1** | Same as test 2, codec AV1 | AV1 often looks cleaner at the same bitrate. Both the 5070 and the Arc 140V handle it in hardware. |
| 4 | **4K + DLSS** | Native (4K), 60 fps, 80–100 Mbps | Sharper menus, text and HUD. See section 4. |
| 5 | **PyroWave** | See section 5 | Lowest, most consistent latency, if it suits you. |

**Record per test:** decode time, network latency, dropped frames, how the frametime graph looks, and how it feels.

| # | Decode (ms) | Network (ms) | Dropped frames | Graph | Feel |
|---|---|---|---|---|---|
| Artemis | ~9 | | | n/a | |
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

---

## 4. 4K and the RTX 5070

A 4K stream involves two separate jobs:

- **Encoding the video is easy.** NVENC is a dedicated video encoder that's separate from the part that renders games. 4K60 HEVC or AV1 barely touches it and doesn't cost game FPS.
- **Rendering the game at 4K is the hard part.** Vibepollo creates a virtual display at whatever resolution Moonlight asks for, so games run at that resolution.

**Options:**
1. **Stream at 1440p** and let the TV upscale. It's simple and matches Artemis.
2. **Stream at 4K and use DLSS** (Quality or Balanced mode). The game renders internally at about 1440p, which is easy for a 5070, and DLSS upscales it. Menus, text and HUD stay sharp at 4K. This usually looks better than option 1 at about the same GPU cost.
3. **Games without DLSS:** keep the display at 4K and lower the in-game render scale or resolution.

You can also stream heavy games at 1440p and lighter ones at 4K. Vibepollo's per-app settings can set a resolution for each game.

---

## 5. PyroWave (wired only)

PyroWave is practical here because both the PC and the laptop are on gigabit Ethernet. 4K60 needs about 290 Mbps and 1440p60 about 220–290 Mbps.

1. Moonlight Settings → **Video codec → "PyroWave (wired LAN, experimental)"**. The bitrate slider switches to PyroWave's much higher range.
2. Click **Calibrate PyroWave**, choose your PC, and use the bitrate it recommends.
3. Vibepollo needs no changes. It negotiates PyroWave automatically.

**What to expect:**
- **Lower, very consistent latency.** Every frame is encoded on its own, with no reliance on previous frames.
- **Fine detail may look a little softer** than HEVC or AV1.
- **A lost packet gives a brief blur**, instead of a smear of broken blocks that lasts until the next keyframe.
- **It costs some GPU time.** Unlike HEVC and AV1, PyroWave encodes on the GPU's main compute cores rather than NVENC. Compare game FPS with and without it, especially in games that already use the GPU fully.

---

## 6. Troubleshooting

| Symptom | Try |
|---|---|
| 403 error or game won't launch | Vibepollo → Devices → laptop → turn on Launch applications and the input permissions. |
| Laptop sleeps when the lid closes | Lid action → Do nothing (plugged in). |
| Small regular stutters | Turn off Display Power Savings, plug in the charger, set Best performance. |
| Choppy at high bitrate | Check the Ethernet adapter shows 1.0 Gbps in Windows network status. Lower the bitrate. |
| Game FPS too low at 4K | Use DLSS or lower the render scale, or stream at 1440p. |
| PyroWave not offered or falls back to H.264 | Check you're on vrr18 and Vibepollo 2.0.4. Run Calibrate PyroWave. |

---

## 7. Deciding

- **The laptop is clearly smoother or sharper:** a wired mini PC (e.g. Intel N150) or a Fire TV Stick 4K Max on good Wi-Fi or Ethernet would likely fix the TV experience.
- **Little difference at 1440p/35 Mbps, big difference at higher bitrate:** the TV's Wi-Fi is the main bottleneck. Ethernet or a better Wi-Fi client is the fix.
- **No real difference:** Artemis on the TV is already good enough. Keep it.
