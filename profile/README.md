<div align="center">

# Xiaomi Redmi 9A Linux

**True Native GNU/Linux for the Xiaomi Redmi 9A (`dandelion` / `blossom`)**  
Powered by the MediaTek Helio G25 (MT6765) · Alpine Linux (`aarch64` / `musl`) · Sxmo (`dwm`)

[![Alpine Linux](https://img.shields.io/badge/OS-Alpine_Linux_aarch64-0D597F?style=flat-square&logo=alpinelinux&logoColor=white)](https://alpinelinux.org/)
[![Sxmo](https://img.shields.io/badge/UI-Sxmo_(dwm)-1793D1?style=flat-square)](https://sxmo.org/)
[![Kernel](https://img.shields.io/badge/Kernel-Linux_4.19-FCC624?style=flat-square&logo=linux&logoColor=black)](https://github.com/xiaomi-redmi-9a-linux/kernel_xiaomi_blossom_linux)
[![GPU](https://img.shields.io/badge/GPU-PowerVR_GE8320_3D-2EA44F?style=flat-square)](https://github.com/xiaomi-redmi-9a-linux/libhybris)
[![Audio](https://img.shields.io/badge/Audio-PipeWire_%2B_MT6357-0A66C2?style=flat-square)](https://pipewire.org/)
[![Boot](https://img.shields.io/badge/Boot-MicroSD_Only_(eMMC_Safe)-1F883D?style=flat-square)](#-safety--non-destructive-boot)

</div>

---

## Overview

**`xiaomi-redmi-9a-linux`** builds and maintains the kernel, `libhybris` patches, native system daemons, and Alpine Linux + **Sxmo** rootfs overlay required to run **bare-metal native Linux** on the **Xiaomi Redmi 9A** (MediaTek Helio G25 / MT6765).

Unlike Android chroots or PRoot containers, the system boots directly into a native Linux userspace from an external **MicroSD card** while leaving the device's internal flash completely untouched:

* **Zero eMMC Risk:** Boots via `fastboot` into a custom Linux 4.19 `boot-linux.img` and mounts an `ext4` loopback rootfs from MicroSD (`/dev/mmcblk1p1` → `/dev/loop0`). Removing the SD card and flashing stock boot restores Android immediately.
* **Hardware 3D Acceleration (`musl` + `libhybris`):** Custom 64-bit `musl` libc compatibility and Treble `/dev/fb0` gralloc fallback enable PowerVR Rogue GE8320 hardware GLES rendering (~395 FPS raw benchmark, 650 MHz DVFS).
* **Lightweight Native Daemons:** Purpose-built Rust and C daemons replace heavy Android/Python services for WMT connectivity loading (`< 4 kB` VmRSS), BlueZ 5.86 bridging, FreeDesktop sensor proxying, FM radio, jack detection, and framebuffer power control.
* **Autonomous In-Kernel Power & Battery:** In-kernel MT6357 IR-compensated OCV fuelgauge plus runtime CPU hotplugging and efficiency-core standby (`cpu4` @ 400 MHz).

---

## Repositories

| Repository | Description |
| :--- | :--- |
| [**`kernel_xiaomi_blossom_linux`**](https://github.com/xiaomi-redmi-9a-linux/kernel_xiaomi_blossom_linux) | Linux 4.19 kernel for Redmi 9A (`blossom_linux_defconfig`) with PowerVR Rogue DDK `1.13`, autonomous MT6357 fuelgauge, VKMS PRIME, CMDQ compat, embedded firmware, early initramfs, and `boot-linux.img` build/flash tools. |
| [**`xiaomi-redmi-9a-linux`**](https://github.com/xiaomi-redmi-9a-linux/xiaomi-redmi-9a-linux) | Alpine Linux + Sxmo rootfs overlay, native Rust & C userspace daemons (`src/`), standalone `linux-rootfs.ext4` & MicroSD installers (`scripts/`), and detailed hardware subsystem docs (`docs/`). |
| [**`libhybris`**](https://github.com/xiaomi-redmi-9a-linux/libhybris) | Patched `libhybris` fork adding 64-bit `musl` libc hooks, Android 10/11 dynamic linker relocation support, and direct `/dev/fb0` double-buffered gralloc fallback for Treble HALs. |
| [**`gst-droid`**](https://github.com/xiaomi-redmi-9a-linux/gst-droid) | GStreamer 1.0 Android camera & hardware codec integration stack (`droidmedia`, `nemo-gst-interfaces`, `gst-droid`). |

---

## Hardware Support Matrix

| Subsystem | Implementation | Status | Notes |
| :--- | :--- | :---: | :--- |
| **SoC / CPU** | MediaTek MT6765 (Helio G25) | Working | 8× Cortex-A53 up to 2.0 GHz with DVFS |
| **Display & Backlight** | `mtkfb` (`/dev/fb0`) + PWM `lcd-backlight` | Working | 720 × 1600 @ 60 Hz, X11 + Sxmo (`dwm`), `mtk-fb-blank` |
| **3D GPU** | PowerVR Rogue GE8320 (DDK `1.13@5776728`) | Working | Patched 64-bit `musl` `libhybris`, ~395 FPS raw GLES, 650 MHz DVFS |
| **Touchscreen** | Novatek / FocalTech multi-touch | Working | Firmware embedded in kernel image |
| **Audio & Jack** | MT6357 ALSA SoC + PipeWire (S16LE) | Working | Speaker + automatic 3.5 mm jack switching (`sxmo_jackmon`) |
| **FM Radio** | MT6631 FM receiver (`mtk_fm`) | Working | Native Rust GTK3 touch app (`fm_radio_gui`), seek/scan, speaker & headphone routing |
| **Wi-Fi** | MediaTek WMT SDIO (`wlan0`) | Working | Native Rust `mtk-wmt-loader`, `wpa_supplicant` + `nl80211` `nmcli` bridge |
| **Bluetooth** | MediaTek MT6631 Bluetooth 5.0 | Working | BlueZ 5.86 via native Rust `mtk-bt-bridge` |
| **Sensors** | `net.hadess.SensorProxy` | Working | Native Rust `mtk-sensor-proxy` (accelerometer, ambient light, proximity) |
| **Battery & Power** | In-kernel MT6357 fuelgauge + hotplug | Working | IR-compensated OCV tracking, screen-off `cpu4` @ 400 MHz standby |
| **Buttons & Input** | `mtk-kpd` + `bonsaid` | Working | Power (lock/wake + long-press menu) & Volume Up/Down (`xob` OSD) |
| **USB Gadget** | ConfigFS RNDIS / CDC-NCM & ACM | Working | USB Ethernet networking + serial root shell (`/dev/ttyGS0`) |
| **GPS / GNSS** | MT6631 STP (`/dev/stpgps`, `/dev/gps_emi`) | Investigated | Kernel devices active; requires MediaTek `mnld` for NMEA fix calculation |
| **Camera / Media** | `gst-droid` (`droidmedia`) | WIP | Integration repository for Android camera & Stagefright codecs |

---

## Custom Native Daemons & Tools

Built natively for `aarch64` Alpine (`musl`) inside [`xiaomi-redmi-9a-linux/src/`](https://github.com/xiaomi-redmi-9a-linux/xiaomi-redmi-9a-linux/tree/main/src):

* **`mtk-wmt-loader`** *(Rust)* — Replaces `wmt-pyloader` for MediaTek MT6765/MT6631 STP & firmware loading (`< 4 kB` RSS).
* **`mtk-bt-bridge`** *(Rust)* — Bridges `/dev/stpbt` to BlueZ 5.86 over HCI.
* **`mtk-sensor-proxy`** *(Rust)* — Exposes accelerometer, ALS, and proximity over `net.hadess.SensorProxy`.
* **`fm_radio_gui` & `mtk_fm`** *(Rust GTK3 + C)* — Touch-friendly FM radio UI and hardware tuner/routing daemon.
* **`sxmo_jackmon` & `mtk-fb-blank`** *(C)* — Instant 3.5 mm jack audio routing and framebuffer/touchscreen sleep control.

---

## Quick Start

1. **Prepare a MicroSD card** (8 GB+) or build a standalone `linux-rootfs.ext4` loopback image using [`xiaomi-redmi-9a-linux`](https://github.com/xiaomi-redmi-9a-linux/xiaomi-redmi-9a-linux):
   ```bash
   git clone https://github.com/xiaomi-redmi-9a-linux/xiaomi-redmi-9a-linux.git
   cd xiaomi-redmi-9a-linux
   sudo ./scripts/install-sd.sh /dev/sdX   # Replace /dev/sdX with your SD card
   ```
2. **Build or download `boot-linux.img`** from [`kernel_xiaomi_blossom_linux`](https://github.com/xiaomi-redmi-9a-linux/kernel_xiaomi_blossom_linux):
   ```bash
   git clone https://github.com/xiaomi-redmi-9a-linux/kernel_xiaomi_blossom_linux.git
   cd kernel_xiaomi_blossom_linux
   ./build.sh kernel && ./build.sh boot
   ```
3. **Boot via Fastboot** (with the MicroSD card inserted):
   ```bash
   fastboot flash boot out-linux/boot-linux.img
   fastboot reboot
   ```

Full step-by-step instructions and subsystem architecture docs live in [`xiaomi-redmi-9a-linux/docs/`](https://github.com/xiaomi-redmi-9a-linux/xiaomi-redmi-9a-linux/tree/main/docs).

---

## Safety & Non-Destructive Boot

> [!IMPORTANT]
> The kernel initramfs, rootfs builder, and installer scripts are engineered **never to touch, mount, or modify internal eMMC (`/dev/mmcblk0*`)**. Your stock Android/MIUI installation and userdata remain intact on internal storage.

---

## AI-Assisted Engineering

This port was co-engineered in collaboration with **Antigravity (Google DeepMind)** across kernel driver bring-up (CMDQ compat ioctls, VKMS PRIME, MT6357 fuelgauge), 64-bit `musl` `libhybris` adaptation, native Rust/C daemon development, and Sxmo/PipeWire system integration.

---

## Contributing

Issues, PRs, and hardware test reports are welcome—especially for **Redmi 9A (`blossom` / `dandelion`)** and sibling **MT6765 / MT6762** devices (`angelica`, `cattail`). Please check the subsystem documentation in [`xiaomi-redmi-9a-linux/docs/`](https://github.com/xiaomi-redmi-9a-linux/xiaomi-redmi-9a-linux/tree/main/docs) before opening pull requests.
