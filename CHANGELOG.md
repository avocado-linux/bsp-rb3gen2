# Changelog

All notable changes to avocado-bsp-rb3gen2 are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.4]

### Added
- The drivers for three functions the kernel builds as modules but the BSP never
  installed, each found by a failing Rubicon board test: the USB-C retimer
  (`nb7vpq904m`, without which no Type-C port registers), the WCN6750 Wi-Fi
  (`ath11k-ahb`, which also boots the WPSS) and the APSS watchdog (`qcom-wdt`).

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
