# YumiOS-OpenMediaVault

Ready-to-flash [OpenMediaVault](https://www.openmediavault.org/) NAS image for the **Yumi SmartPi One** (AllWinner H3), built on top of the [SmartPi-armbian](https://github.com/Yumi-Lab/SmartPi-armbian) Debian 12 (Bookworm) server base with [CustomPiOS](https://github.com/Yumi-Lab/CustomPiOS-Yumi).

It automates the manual installation documented in the [SmartPi OpenMediaVault wiki guide](https://wiki.yumi-lab.com/SmartPI/SmartPI_OpenMediaVault/): flash, boot, wait, and your NAS is up.

## Download

Grab the latest image from the [Releases](../../releases) page. Multi-part `.7z.001/.002` archives extract natively with 7-Zip (Windows) or The Unarchiver (macOS).

## Flash

Use your favorite imaging tool, or:

```bash
diskutil list && diskutil unmountDisk /dev/diskN
sudo dd if=<image>.img of=/dev/rdiskN bs=4m status=progress
diskutil eject /dev/diskN
```

## First boot

All OpenMediaVault packages are pre-downloaded inside the image. On the very first boot the device installs them locally (no network required for the install itself), registers the ethernet interface with DHCP in the OpenMediaVault database, hands the network over to `systemd-networkd` and reboots on its own.

**This takes 15-30 minutes. Do not power off the device during the first boot.** Progress is logged to `/var/log/omv-firstboot.log` and shown on the HDMI console.

## Access

| Service | Credentials |
|---------|-------------|
| Web UI — `http://<device-ip>` | `admin` / `openmediavault` |
| SSH | `pi` / `yumi` |

Change both passwords right after the first login.

## How it is built

```
SmartPi-armbian base image (Debian 12 server)
    └── CustomPiOS chroot
        ├── armbian / armbian_net / udev_fix   (base tweaks, network configurator)
        └── omv                                (this repo's module)
            ├── openmediavault keyring + sandworm repository
            ├── pre-download of the full package tree into the image
            └── omv-firstboot.service          (real install at first boot)
```

The `openmediavault` package cannot be installed inside a build chroot: its postinst deploys SaltStack states that need a live systemd. The chroot stage therefore only stages everything, and `omv-firstboot` performs the actual install on the device — identical to the wiki procedure, just unattended.

## Status

- **Validated in CI**: image builds green; e2fsck clean; smoke test verifies the sandworm repository, the dearmored keyring, 143 pre-downloaded `.deb` (including `openmediavault_*.deb`), baked-in apt lists and the armed `omv-firstboot.service`.
- **Not yet validated**: the first boot procedure on real hardware (install from cache, NIC registration, systemd-networkd handover). Test on a device before publishing a release.

## Development

```bash
# Test build (artifact only): push to develop
git push origin develop

# Release: run the Release workflow with a version number
gh workflow run Release.yml -f version=X.Y.Z
```

## License

GPLv3 — see [LICENSE](LICENSE). Build tooling forked from [MainsailOS](https://github.com/mainsail-crew/MainsailOS).
