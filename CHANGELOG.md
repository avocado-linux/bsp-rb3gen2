# Changelog

All notable changes to avocado-bsp-rb3gen2 are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0]

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
