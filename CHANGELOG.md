# Changelog

All notable changes to avocado-bsp-rb3gen2 are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.9]

### Added
- Everything the board's enabled device tree calls for, found with the
  board-bringup coverage audit (its DT-INSTALL list is now empty):
  - Audio: the card's machine driver (`snd-soc-sc8280xp`) and its AudioReach
    path (`snd-q6apm`, `q6apm-dai`, `q6apm-lpass-dais`, `q6prm`,
    `q6prm-clocks`), the WSA and VA macros, `soundwire-qcom`, the WSA883x
    speaker amps, `reset-gpio` (their shared reset line) and HDMI audio, plus
    the card's UCM profile (`alsa-ucm-conf`). The card registers and a tone plays to
    completion on the UCM Speaker PCM (acoustic output not verified: the board
    DMICs hear no tone, likely no speakers fitted); it also needs the DP-audio-link
    overlay fix.
  - CAN: `mcp251xfd` for the on-board MCP2518FD (a loopback frame round trip
    passes).
  - `nvmem-reboot-mode` (`reboot bootloader` / `reboot edl`), `usb-conn-gpio`
    (micro-USB OTG role), `usb4604`, and `clk-gp-mnd` (PDM GP MN clock).

### Changed
- The audio notes: the board DTS has had its `&sound` card for a while; the
  gap was the missing machine driver and AudioReach modules.
- The "deliberately not listed" note is gone: the video codec and reboot
  modes are installed, and the coverage audit is the check.

### Not installed, on purpose
- `onboard-usb-dev`: the DT calls for it, but loaded it re-enumerates the
  Genesys hub on the PCIe xHCI in an endless loop and took Ethernet down.

## [0.3.6]

### Added
- The video codec: `qcom-iris` with `videocc-sc7280` (the video clock
  controller, without which the codec waits for its supplier forever),
  `videobuf2-dma-contig` and the `qcom/vpu/vpu20_p1.mbn` firmware
  (`linux-firmware-qcom-vpu`). The kernel built the drivers but nothing
  installed them, so `/dev/video*` had no encoder or decoder. With them,
  GStreamer's `v4l2h264enc`, `v4l2h265enc`, `v4l2h264dec`, `v4l2h265dec`
  and `v4l2vp9dec` work: 1280x720 H.264 encodes at about 190 fps.

## [0.3.5]

### Added
- The CDSP's FastRPC shell (`hexagon-dsp-binaries-thundercomm-rb3gen2-cdsp`),
  without which nothing runs on the NPU: every FastRPC session failed with
  0x80000600. Needs a feed built with avocado-linux/meta-avocado#561.

## [0.3.4]

### Added
- The drivers for three functions the kernel builds as modules but the BSP never
  installed, each found by a failing Rubicon board test: the USB-C retimer
  (`nb7vpq904m`, without which no Type-C port registers), the WCN6750 Wi-Fi
  (`ath11k-ahb`, which also boots the WPSS) and the APSS watchdog (`qcom-wdt`).
- The rest of the WCN6750: `qrtr-smd` and `qcom-pd-mapper`, without which the
  WPSS runs but no Wi-Fi radio registers, and `pwrseq-qcom-wcn`, `bluetooth`,
  `btqca` and `hci-uart` for Bluetooth. Loaded together on the board, they
  brought up phy0 (Wi-Fi) and hci0 (Bluetooth).

## [0.3.3]

### Added
- Adreno 643 GPU. Display bring-up modules `phy-qcom-edp` (mini-DP) and
  `lontium-lt9611uxc` (DSI->HDMI), without which the msm display aggregate
  never binds and there is no `/dev/dri`; the a660 GMU/SQE microcode
  (`linux-firmware-qcom-adreno-a660`); and the Mesa userspace: Turnip
  (`mesa-vulkan-drivers`) with the Vulkan loader (`vulkan-loader`), the
  Gallium DRI driver, GLES, EGL and GBM.
  Verified on RB3 Gen 2 hardware: card0 + renderD128, DP-1 and HDMI-A-1, across
  a cold provisioned boot and an A/B OS update. Needs meta-avocado's
  `qcs6490-rb3gen2-dp-disable` UKI overlay (avocado-linux/meta-avocado#396).
- CDSP compute offload: `fastrpc` and `kernel-module-fastrpc`. Verified on RB3
  Gen 2 hardware under KVM with the 00142 boot binaries
  (avocado-linux/meta-avocado#561) and the CDSP's FastRPC shell (0.3.5):
  `fastrpc_test` passes on the CDSP. Under the older 00137 boot binaries the
  CDSP times out at start, so the packages install with nothing behind
  `/dev/fastrpc-cdsp`.

### Removed
- The coresight and STM trace modules. Loading them races the SMMU's
  `amba_dma_configure()` into a NULL dereference that leaves the GPU without an
  IOMMU and no `/dev/dri`. They are debug-only and stay out until the kernel
  fix lands.

## [0.2.1]

### Fixed
- `avocado deploy` failing at slot detection. The stone config runs
  `avocado-bls current-slot`, but avocado-bls is feed-only and nothing installed
  it. The BSP now installs `avocado-bls`.

## [0.2.0]

### Removed
- The 2024 feed from CI, and the `kernel-6.6.*` package block that served it.
  This board does not ship on 2024, so that matrix leg could only ever fail and
  the block's packages resolve against no feed we build.

### Changed
- Install `kernel-module-tc956x-pcie-eth` rather than `qps615-dlkm`. The recipe's
  base package is built once per multiconfig under one name differing only by
  PR, so dnf takes the highest and it resolves on only one kernel; the
  `kernel-module-*` name carries the kernel version and resolves against
  whichever sysroot is being built.

## [0.1.0]

### Added
- Core-kit board support for the Qualcomm Robotics RB3 Gen 2: the QPS615 PCIe
  Ethernet driver and its firmware, WCN6750 wifi/BT firmware, and the LT9611UXC
  HDMI bridge firmware.
- `kernel-module-avocado-shm` for the inter-VM shared-memory reference.

### Notes
- Derived from avocado-bsp-rubikpi3, which shares the SoC. Fifteen packages that
  came across in that derivation do not exist for this board and were removed:
  renaming artifacts (`firmware-rb3gen2`, `systemd-rb3gen2-masks`,
  `rb3gen2-{init-services-wifi,init-services-bt,thermal}`, the Thundercomm audio
  firmware) and RubikPi-only hardware (Broadcom `brcmfmac`/`brcmutil`,
  `wiringrp*`, `rubikpi-bt-staticdev`).
- `enable_services` is absent for the same reason: its two units came from
  packages this board does not have, and a unit no package ships fails
  `ext build`.
