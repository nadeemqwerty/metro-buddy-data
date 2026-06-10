# metro-buddy-data

Static data + remote config for the **RaahMetro** app, served over GitHub Pages.

Base URL: `https://nadeemqwerty.github.io/metro-buddy-data/`

## `ads-config.json` — remote ads switch

Controls **Google AdMob** ads in the app **without an app update**. The app reads
this file in the background on launch and applies it on the **next** launch
(stale-while-revalidate; works offline from the last cached value).

Live URL: <https://nadeemqwerty.github.io/metro-buddy-data/ads-config.json>

```json
{
  "schemaVersion": 1,
  "adsEnabled": true,
  "bannerRefreshSeconds": 0
}
```

| Field | Type | Meaning |
| --- | --- | --- |
| `schemaVersion` | number | Must be `1`. A payload without it is ignored by the app. |
| `adsEnabled` | **strict boolean** | Master on/off. Use `true` / `false` — **not** the string `"false"` (a quoted value is ignored and will not turn ads off). |
| `bannerRefreshSeconds` | number | `0` = let the AdMob console auto-refresh handle it (recommended). Otherwise valid range is `30`–`300`; `1`–`29` clamps up to `30`, `>300` caps to `300`; invalid → `0`. |

### How to toggle ads (clean workflow)
1. Edit `ads-config.json` here — set `"adsEnabled": false` to stop ads, `true` to resume.
2. Commit & push (GitHub Pages redeploys in ~1 min).
3. Each user picks it up on their **next app launch**. No rebuild, no Play submission.

### ⚠️ Important
- This switch is **fail-open**: if the app can't fetch this file (offline / 404),
  it falls back to ads **ON**. A brand-new install that is offline gets the default.
- **Never delete this file to disable ads** — a 404 makes new installs default to ON.
  Always keep it hosted and set `"adsEnabled": false`.
- The only guaranteed hard-off shipped in the binary is the build-time
  `EXPO_PUBLIC_ADS_ENABLED=false` (requires a rebuild). Remote + build flags are
  ANDed — either one OFF means no ads.

Full details: see `docs/ADMOB.md` in the [RaahMetro app repo](https://github.com/nadeemqwerty/metro-buddy).
