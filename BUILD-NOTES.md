# X60 Pro (MT7986) firmware build notes

Target device: `ruijie-rg-x60-pro` (MT7986 + AX6000)

## 5G TX power: 26.5 -> 29 dBm

| Patch | Change |
|---|---|
| `026-x60-pro-5g-txpower-29dbm.patch` | 5G SKU table: all channels (36-165) limit raised to 29 dBm. 2.4G untouched. |
| `027-x60-pro-5g-powerup-boost-29dbm.patch` | Fills the PowerUp table for band1 (5G) at AP start-up: base + 2.5 dB, clamped by the 29 dBm SKU ceiling -> 29 dBm. |
| `mt7986-ax6000.dbdc.b1.dat` | `SKUenable=1` so the 5G SKU table actually takes effect. |

Result: HT20/40, VHT20/40/80/160, HE484/HE996/HE996X2 all report `0x3a (058)` = 29.0 dBm.

## OAF - app filtering / parental control

- feed: `src-git openappfilter https://github.com/destan19/OpenAppFilter.git;master`
- packages: `kmod-oaf`, `appfilter`, `luci-app-oaf`, `luci-i18n-oaf-zh-cn`
- `kmod-oaf` was already present in the defconfig but no feed provided it, so
  `make defconfig` silently dropped it. With the OAF feed it is built for real.

## iStore app store

- feed: `src-git store https://github.com/linkease/istore.git;main`
- packages: `luci-app-store`, `luci-lib-taskd`, `luci-lib-xterm`, `taskd`
- `luci-compat` is required by both `luci-app-oaf` and `luci-app-store`.

Note: OAF relies on netfilter hooks, so hardware flow offload (MTK WARP/HNAT)
can bypass its DPI. Disable hardware offload in LuCI if app filtering appears
not to work.
