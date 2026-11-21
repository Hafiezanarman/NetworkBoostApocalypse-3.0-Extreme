# NETWORK BOOST APOCALYPSE

## APOCALYPSE ENGINE v3.0 EXTREME
### Extreme Adaptive Network Optimization for Android 10–17

**AUTHOR: CJ7**

```
NETWORK BOOST APOCALYPSE
APOCALYPSE ENGINE
EXTREME ADAPTIVE NETWORK OPTIMIZATION
AUTHOR: CJ7

Android 10–16 intended support
Android 17 forward compatibility

Gaming | Download | Streaming | Hotspot | Balanced

META-INF INCLUDED
Real tuning. Adaptive engine. Zero placebo.
```

---

## 1. What this module actually does

Network Boost Apocalypse tunes the **Linux kernel networking stack** on rooted
Android through a strict, validated sysctl workflow. It does **not** use magic
`build.prop` speed properties, does not inject packets, does not modify game
traffic, and does not bypass carrier, router, ISP or server limits.

Real, measurable goals:

- Use the best **available** TCP congestion control (BBR when the running kernel
  actually provides it and the profile benefits from it; CUBIC as fallback;
  otherwise the vendor default is preserved).
- Bound TCP receive/send buffers using **installed RAM** and a hard ceiling, so
  increasing throughput does not create giant standing queues (bufferbloat).
- Normalize standard RFC features (SACK, window scaling, timestamps) only when
  they exist and are disabled.
- Keep short queues for interactive traffic to reduce latency and jitter where
  the kernel permits.
- Detect capabilities dynamically and skip anything unsupported without
  crashing or boot-delay.
- Back up everything before writing, read back after writing, and revert any
  value that does not apply exactly as intended.

### Honest limitations

- It cannot increase physical signal strength, modem capability, carrier
  allocation, router quality, ISP speed, or remote server throughput.
- It cannot bypass throttling, force NR-only/LTE-only, lock bands, change APNs,
  falsify network type or signal, or modify modem firmware/NV items.
- Most competitive games use **UDP**. TCP tuning cannot directly accelerate UDP
  packets. The Gaming profile works on queuing and on TCP control connections
  (login, matchmaking, voice signalling, chat), not on the game's UDP stream.
- DNS is **not** replaced. Android Private DNS / DoT, VPN DNS routing,
  network-specific DNS and captive portal detection are left alone.
- No speed gain is guaranteed. If your kernel exposes no writable parameters,
  the module safely does nothing measurable.

---

## 2. Compatibility

| Item | Target |
|---|---|
| Android | 10 (API 29) → 16 (API 36) intended |
| Android 17 (API 37) | Forward-compatible design, **not verified on hardware** |
| Kernel | Any Linux kernel with standard procfs sysctls; older kernels skip unsupported features |
| Architecture | arm64, arm, x86, x86_64 (shell-only implementation) |
| Root | Magisk, KernelSU, APatch (where module system is available) |

Compatibility is decided by **feature detection**, not by Android version
numbers. Detection covers SDK/release, kernel release, architecture, root
manager, available sysctls, writable sysctls, TCP congestion algorithms,
network interfaces, RAM, BusyBox/tool availability and existing values.

---

## 3. Installation

### Mode A — Magisk / KernelSU / APatch (recommended)

1. Flash `NetworkBoostApocalypse-3.0-Extreme.zip` through the root manager app.
2. Reboot.
3. The engine starts in **APOCALYPSE BALANCED** automatically.

`customize.sh` prints branding, validates files, sets permissions and creates
runtime directories. It applies **no** network tweaks during installation and
never fails because a kernel feature is missing.

### Mode B — Recovery

The ZIP includes a real `META-INF/com/google/android/update-binary` (shell,
edify interface 1/2/3) and `updater-script`.

The recovery installer:

- prints the required branding and author,
- checks arguments and ZIP integrity with `unzip -t`,
- validates `module.prop` (id and author),
- detects the recovery environment and device architecture,
- detects Magisk / KernelSU / APatch module locations,
- refuses to install if no supported root manager or `/data/adb/modules` is
  available,
- extracts only module files (never `META-INF`) to
  `/data/adb/modules/networkboostapocalypse`,
- sets permissions, validates all required scripts, and reports real success or
  a clear error,
- never writes system partitions and never applies tweaks during install.

If your recovery lacks `unzip` or cannot see decrypted `/data`, use Mode A.

---

## 4. Directory structure

```
NetworkBoostApocalypse/
├── META-INF/com/google/android/
│   ├── update-binary          # functional recovery installer
│   └── updater-script         # valid edify UI messages
├── module.prop
├── customize.sh
├── post-fs-data.sh
├── service.sh
├── uninstall.sh
├── system.prop                # intentionally minimal, no magic properties
├── config/profiles.conf
├── bin/network_engine.sh
├── scripts/
│   ├── detect.sh
│   ├── functions.sh
│   ├── tcp.sh
│   ├── buffer.sh
│   ├── gaming.sh
│   ├── download.sh
│   ├── streaming.sh
│   ├── hotspot.sh
│   ├── balanced.sh
│   ├── restore.sh
│   └── benchmark.sh
└── README.md
```

Runtime state (created on device, outside the module):

```
/data/adb/NetworkBoostApocalypse/
├── backup/          # original values, one file per parameter
├── state/
│   ├── capabilities # cached detection results
│   ├── profile      # persisted selected profile
│   ├── modified.list
│   └── .engine.lock
/data/adb/NetworkBoostApocalypse.log
/data/adb/NetworkBoostApocalypse_Benchmark.txt
```

---

## 5. Profiles

| Profile | Focus | Congestion control | Buffers |
|---|---|---|---|
| `gaming` | Latency, jitter, short queues | CUBIC if available, else preserve | 0.50× memory-derived ceiling, backlog kept short |
| `download` | Sustained throughput | BBR if available, else CUBIC, else preserve | 1.35× ceiling, capped |
| `stream` | Sustained playback + responsive browsing | BBR if available, else CUBIC | 1.00× ceiling |
| `hotspot` | Tether stability, routing untouched | Preserve | 1.00× ceiling |
| `balanced` | Daily use, battery conscious | Preserve vendor default | 1.00× ceiling |

`APOCALYPSE BALANCED` is the automatic fallback. Automatic workload detection is
**disabled by default** because stock Android does not expose a reliable signal;
the module will not pretend to detect gaming or downloading.

### Changing profiles

From a root shell:

```sh
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh profile gaming
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh profile download
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh profile stream
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh profile hotspot
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh profile balanced
```

Other commands:

```sh
.../network_engine.sh status       # current state
.../network_engine.sh detect       # re-run capability detection
.../network_engine.sh apply        # re-apply active profile
.../network_engine.sh rollback     # restore every backed-up parameter
.../network_engine.sh benchmark    # run measurements
.../network_engine.sh validate     # verify files and permissions
.../network_engine.sh help
```

The selected profile survives reboot. Repeated application is idempotent: an
identical value is never rewritten.

---

## 6. Engines

### Kernel Intelligence Engine

For **every** parameter the engine:

1. checks existence,
2. reads the original value,
3. verifies writability,
4. validates the target value,
5. applies the tuning,
6. reads it back,
7. verifies the result,
8. logs the outcome,
9. restores the original if the readback is invalid.

Unsupported parameters are skipped. No value is ever written to an arbitrary
`/proc` or `/sys` path.

### Adaptive TCP Engine

- Detects `/proc/sys/net/ipv4/tcp_available_congestion_control`.
- BBR only when actually listed **and** appropriate for the profile.
- CUBIC fallback; vendor default preserved otherwise.
- Tunes `tcp_rmem` / `tcp_wmem` within RAM-derived ceilings.
- Enables SACK / window scaling / timestamps only when currently `0` and
  `NBA_TCP_NORMALIZE=1`.
- Detects TCP Fast Open but never forces it.
- Optional conservative keepalive retune (`NBA_TUNE_KEEPALIVE=1`, Gaming only).
- Avoids retransmission-behavior experiments and buffer inflation.

### APOCALYPSE Buffer Engine

Inspects only real sysctls:

```
net.core.rmem_max          net.core.wmem_max
net.core.rmem_default      net.core.wmem_default
net.core.netdev_max_backlog
net.ipv4.tcp_rmem          net.ipv4.tcp_wmem
net.ipv4.tcp_mem
```

A RAM-derived base (1–16 MB), a profile multiplier and an optional hard
ceiling (`NBA_BUFFER_MAX_MB`) produce bounded values. Queues are kept short for
interactive profiles. `tcp_mem` tuning is opt-in and disabled by default.

### Gaming / Download / Streaming / Hotspot engines

See the profile table. Hotspot only **observes** `ip_forward` and IPv6
forwarding; it never changes forwarding, NAT, DHCP, routing, firewall, USB
tethering or Wi-Fi AP configuration.

### LTE / 4G / 5G / Wi-Fi / DNS

The module deliberately does not touch:

- modem network selection, band preference, carrier aggregation or handover,
- roaming, APN or preferred network settings,
- Wi-Fi driver parameters, channel, transmit power, scanning or roaming,
- Private DNS, DoT, VPN DNS routing, captive portal detection.

Software cannot raise physical signal strength or change what the carrier
permits, so no such tweaks exist.

---

## 7. Backup, restore and uninstall

Before the first successful change to a parameter, its original value is saved
under `/data/adb/NetworkBoostApocalypse/backup/`. A backup is never overwritten
with an already-tuned value. Only parameters this module changed are tracked.

```sh
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh rollback
```

Uninstalling through the root manager calls `uninstall.sh`, which restores
tracked values where possible and removes only this module's runtime state and
backups. It never deletes unrelated files, modem config or user network
settings. Missing files are handled gracefully.

---

## 8. Logging

Log file: `/data/adb/NetworkBoostApocalypse.log`

Records startup, Android/kernel/architecture/root manager, supported TCP
algorithms, profile selection, applied/skipped tuning, validation results,
rollback and errors. It never records credentials, browsing history, full IP
lists or personal identifiers. The log is trimmed automatically at
`NBA_LOG_MAX_BYTES` (default 256 KiB).

Verbosity: `NBA_LOG_LEVEL` in `config/profiles.conf` (`0` errors, `1` normal,
`2` verbose).

---

## 9. Benchmark engine

```sh
/data/adb/modules/networkboostapocalypse/bin/network_engine.sh benchmark
```

Report: `/data/adb/NetworkBoostApocalypse_Benchmark.txt`

Measures what the device can actually measure with tools already installed
(`ping`, `curl`, `iperf3`, `ip`):

- idle latency (min/avg/max, jitter, loss),
- loaded latency (only when a user-supplied load endpoint exists),
- download throughput (iperf3 `-R` or curl to a user-supplied URL),
- upload throughput (iperf3 to a user-supplied server),
- stability derived from the measured loss/jitter,
- active interfaces where detectable.

Set endpoints in `config/profiles.conf` before benchmarking:

```sh
NBA_BENCH_HOST="1.1.1.1"          # latency target; empty = default gateway
NBA_BENCH_SERVER="192.168.1.50"   # your own iperf3 server
NBA_BENCH_URL="https://example.com/small-test-file"
NBA_BENCH_SECONDS=5
```

No binaries are downloaded. Nothing is fabricated. Every line is labelled
**MEASURED** or **UNAVAILABLE**. Results are always bounded by modem, signal,
router, ISP, VPN and server conditions. Compare runs with the module active vs.
rolled back for an honest before/after picture.

---

## 10. Configuration reference

Edit `config/profiles.conf` (persisted profile in state takes precedence):

| Key | Default | Meaning |
|---|---|---|
| `NBA_PROFILE` | `balanced` | Default profile |
| `NBA_AUTO_PROFILE` | `0` | Conservative hint engine; stays balanced without reliable signals |
| `NBA_USE_BBR` | `1` | Allow BBR only when available and appropriate |
| `NBA_TCP_NORMALIZE` | `1` | Enable SACK/window scaling/timestamps if currently off |
| `NBA_TUNE_KEEPALIVE` | `0` | Opt-in Gaming-only keepalive retune |
| `NBA_FALLBACK_CONG` | `cubic` | Fallback algorithm |
| `NBA_BUFFER_MAX_MB` | `0` | Hard per-socket ceiling; `0` = RAM-derived |
| `NBA_*_BUFFER_MULT` | see file | Per-profile multiplier |
| `NBA_TUNE_BACKLOG` | `1` | Profile-aware netdev backlog |
| `NBA_TUNE_TCP_MEM` | `0` | Opt-in, conservative `tcp_mem` tuning |
| `NBA_LOG_LEVEL` | `1` | Verbosity |
| `NBA_LOG_MAX_BYTES` | `262144` | Log rotation threshold |
| `NBA_MONITOR` | `0` | Optional 60 s bounded watchdog (off by default) |

---

## 11. Safety guard

The module never:

- disables SELinux, thermal protection or Android security,
- disables IPv6 globally, breaks VPN, Wi-Fi calling or tethering,
- modifies modem firmware, NV items, APN, bands, roaming or network selection,
- destroys routing tables, NAT, DHCP or firewall rules,
- kills networking services or background apps,
- writes random values to sysfs/procfs,
- forces unsupported TCP algorithms,
- runs endless CPU loops, delays boot or consumes excessive RAM,
- applies placebo tweaks or makes unverifiable speed claims.

All scripts are idempotent, lightweight, reversible, error tolerant, BusyBox
compatible and safe on unsupported devices.

---

## 12. Build / package

From **inside** the `NetworkBoostApocalypse/` directory (module files must sit
at the ZIP root for Magisk and the recovery installer):

```sh
cd NetworkBoostApocalypse
zip -r ../NetworkBoostApocalypse-3.0-Extreme.zip . -x '.*'
cd ..
unzip -l NetworkBoostApocalypse-3.0-Extreme.zip
```

Executable bits are set on all lifecycle scripts, the engine and every script in
`scripts/`.

---

**NETWORK BOOST APOCALYPSE — APOCALYPSE ENGINE v3.0 EXTREME**
**Real tuning. Adaptive engine. Zero placebo. — CJ7**
