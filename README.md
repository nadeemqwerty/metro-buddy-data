# raah-metro-data

Static data + remote config for the **RaahMetro** app, served over GitHub Pages.

Base URL: `https://micromanplay.github.io/raah-metro-data/`

## `ads-config.json` — remote ads switch

Controls **Google AdMob** ads in the app **without an app update**. The app reads
this file in the background on launch and applies it on the **next** launch
(stale-while-revalidate; works offline from the last cached value).

Live URL: <https://micromanplay.github.io/raah-metro-data/ads-config.json>

```json
{
  "schemaVersion": 2,
  "adsEnabled": true,
  "bannerRefreshSeconds": 0,
  "banner": { "enabled": true },
  "interstitial": { "enabled": false, "minIntervalSeconds": 180, "maxPerSession": 3 },
  "rewarded": { "enabled": true }
}
```

### Global fields

| Field | Type | Meaning |
| --- | --- | --- |
| `schemaVersion` | number | `1` or `2`. A payload without a supported version is ignored. v1 files (no nested blocks) still work — the nested formats just fall back to their defaults (banner ON, interstitial OFF, rewarded ON). |
| `adsEnabled` | **strict boolean** | Master on/off for **all** ad formats. Use `true` / `false` — **not** the string `"false"` (a quoted value is ignored and will not turn ads off). |
| `bannerRefreshSeconds` | number | `0` = let the AdMob console auto-refresh handle it (recommended). Otherwise valid range is `30`–`300`; `1`–`29` clamps up to `30`, `>300` caps to `300`; invalid → `0`. |

### Per-format control (the part you tune most)

Every format has its own strict-boolean `enabled`. All toggles are **ANDed** with
`adsEnabled` and the build flag, so the master switch still kills everything.

| Format | Field | Default | Notes |
| --- | --- | --- | --- |
| Banner | `banner.enabled` | `true` | Anchored adaptive banner above the tab bar. |
| Interstitial | `interstitial.enabled` | **`false`** | Full-screen ad shown **only** when the user closes the Settings screen. **Fail-CLOSED**: stays off unless `enabled` is a real `true` **and** both caps are valid. |
| Interstitial | `interstitial.minIntervalSeconds` | `180` | Min seconds between two interstitials. Clamped `60`–`3600`. |
| Interstitial | `interstitial.maxPerSession` | `3` | Max interstitials per app session. Clamped `1`–`20`. |
| Rewarded | `rewarded.enabled` | `true` | The opt-in "Remove ads for 24h" card in Settings. Turn off to hide the card. |

> **Strict-boolean rule (every `enabled`):** only a literal JSON `true` enables a
> format. `"true"`, `1`, `"false"`, `null`, or a missing/garbage value → treated
> as **off** for intrusive formats. This means a typo can never *accidentally
> enable* interstitials.

### How to toggle ads (clean workflow)
1. Edit `ads-config.json` here:
   - Stop **all** ads → `"adsEnabled": false`.
   - Turn one format on/off → e.g. `"interstitial": { "enabled": true, ... }`.
   - Tune interstitial frequency → change `minIntervalSeconds` / `maxPerSession`.
2. Commit & push (GitHub Pages redeploys in ~1 min).
3. Each user picks it up on their **next app launch**. No rebuild, no Play submission.

> You can also pause/serve any format from the **AdMob dashboard** (per ad unit)
> independently of this file. Use this file for product decisions (which formats,
> how often); use AdMob for fill/mediation/payment-side controls.

### ⚠️ Important
- This switch is **fail-open**: if the app can't fetch this file (offline / 404),
  it falls back to ads **ON**. A brand-new install that is offline gets the default.
- **Never delete this file to disable ads** — a 404 makes new installs default to ON.
  Always keep it hosted and set `"adsEnabled": false`.
- The only guaranteed hard-off shipped in the binary is the build-time
  `EXPO_PUBLIC_ADS_ENABLED=false` (requires a rebuild). Remote + build flags are
  ANDed — either one OFF means no ads.

Full details: see `docs/ADMOB.md` in the [RaahMetro app repo](https://github.com/micromanplay/raah-metro).
