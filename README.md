# avocado-bsp-rb3gen2

Board support for the Qualcomm **RB3 Gen 2 Vision Kit** (QCS6490).

## Provenance — read this before trusting the package list

This extension was **derived from `bsp-rubikpi3`**, not written from the RB3's
own schematics. The two boards are the same SoC (QCS6490, 4x A78 + 4x A55), so
the SoC-level content — the QUP/GENI buses, USB, coresight, the interconnect
and clock drivers, the Qualcomm firmware blobs — is genuinely shared and
carries over unchanged.

What does NOT automatically carry over is anything the *board* adds. Known
differences to verify on hardware before calling this complete:

* **Networking.** ~~Assume the network path is wrong.~~ Resolved: the RubikPi
  reaches the network through a USB NIC behind a Renesas bridge and the RB3
  Gen 2 does not have that part, but the board's real NIC -- a QPS615 PCIe
  switch driven by `qps615-dlkm` plus a firmware blob -- is already built into
  the feed and named by `packagegroup-avocado-qcom-extra`. Both are now listed
  in this extension, along with the WCN6750 wifi/BT, LT9611UXC HDMI and
  qcs6490-modem firmware that block covers. Still unverified *on hardware*,
  but the packages are no longer missing.
* **Cameras.** The Vision Kit is a camera board; none of its sensor drivers are
  here. Nothing in the VM demo needs them, so they are deliberately absent
  rather than guessed at.
* **Display / GPU.** Same reasoning as the RubikPi: not wired up.

## Status

First cut, unbuilt and unbooted. It exists so the dual-VM reference project has
a BSP extension to name for this target; treat every package as a hypothesis
inherited from a sibling board until a boot confirms it.

## Core kit only — mezzanines are separate extensions

This extension covers the RB3 Gen 2 **core kit**. The board ships as a core kit
plus one of several mezzanines, and the mezzanine is what varies between
products, so each one is its own extension that `depends_on` this:

| | |
|---|---|
| `avocado-bsp-rb3gen2` | this — QPS615 Ethernet, WCN6750 wifi/BT, LT9611UXC |
| `avocado-bsp-rb3gen2-vision` | CAMSS + Sony IMX577 |
| `avocado-bsp-rb3gen2-industrial` | WCD9370 audio, ST33 SPI TPM (prep, unverified) |

Same split the EXMP-Q911 uses, where the module is the MACHINE and the EXCC
carrier is an extension. Anything that is a property of the removable board
goes in an extension, or every product combination needs its own machine.

Do not add mezzanine parts here.

## Known: the core kit's own Ethernet needs a device-tree overlay

`qps615-dlkm` ships here and the module loads, but on hardware it does not
probe:

```
tc956x_pci-eth 0001:05:00.1: error -ENOENT: Failed to get phy-reset-gpios
tc956x_pci-eth 0001:05:00.1: Platform probe error -2
```

The base `qcs6490-rb3gen2.dts` carries only the pinctrl state
(`tc9563_resx_n`). The `qps615` PCI node that supplies `phy-reset-gpios` for
both ports lives in upstream's `qcs6490-rb3gen2-staging.dtso`, which this build
does not compile and the Qualcomm flow would not apply anyway.

That overlay is core-kit, not mezzanine — it is the board's own NIC — so it
belongs with the machine rather than in an extension. It is the same
overlay-application gap the mezzanine extensions document.
