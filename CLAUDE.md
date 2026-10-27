# espresense (fork)

This is a fork of [ESPresense/ESPresense](https://github.com/ESPresense/ESPresense), forked and
patched for one specific, disposable purpose: capturing a BLE Identity Resolving Key (IRK) from a
phone pairing, as part of `eufyMonitor/ble_scanner`'s IRK-resolution research (see
`../docs/PLAN.md` for the full plan).

**This fork is not meant to track or merge upstream changes, and is not meant to be contributed
back.** Its patches intentionally diverge from upstream's normal operating model:

- **No networking at all.** `setupNetwork()` is skipped entirely in `main.cpp`'s `setup()` — no
  WiFi, no captive portal, no MQTT, no HttpWebServer. Not a bug — the IRK prints to the serial log
  unconditionally once a bond completes, and nothing else in this build needs it.
- **Always-on enrollment.** `Enrollment::Setup()` sets `enrolling`/`enrollingEndMillis` directly,
  so the board advertises connectable (Heart-Rate-Monitor service) from boot — no `/ws`/MQTT
  "enroll" trigger needed or possible in this build.
- **Two general compatibility fixes**, unrelated to the above and not fork-specific quirks — both
  would hit any `esp32s3` build against the locally installed ESP-IDF v5.3.1:
  - `Ble.h`: NimBLE's `os.h` defines unguarded `#define max`/`min` macros that mangle libstdc++
    internals in any file that includes `Ble.h` before `<algorithm>`/`<functional>` — undefined
    once, centrally, right after NimBLE's own includes.
  - `Network.cpp`: `ESP_NETIF_CAPTIVEPORTAL_URI` doesn't exist in this IDF version's
    `esp_netif_dhcp_option_id_t` enum yet — uses the raw RFC 8910 value (114) instead.
- **QIO flash mode** (`sdkconfig.defaults.esp32s3`): this board family P0-boot-loops
  (`RTCWDT_RTC_RST`) on the default DIO mode — see `eufyMonitor/ESP_camwatch/BACKLOG.md` item #8
  for the original investigation.

Board: ESP32-S3 (N16R8, 16MB flash/8MB PSRAM). Toolchain: ESP-IDF v5.3.1 at
`/home/thomas/esp/esp-idf` (`source export.sh` before building). Build:
`./build.sh esp32s3 -p /dev/ttyACM0 flash monitor`.

## Workflow rules

- **Always use a feature branch + PR, never push directly to `main`** on this fork's own remote.
- **Never change the dev PC's own network connectivity without asking first** — see
  `../CLAUDE.md` / `../docs/PLAN.md` for why this matters here specifically.
