# mbp55-linux

A tailor-made Debian 13 ("trixie") live/installable image with a light **XFCE**
desktop for a **MacBookPro5,5 (mid-2009)**. It boots to a login screen and ships
a few basic apps. It is meant to replace one spare Linux partition.

## Why it is built this way

| Hardware | Choice |
|---|---|
| Core 2 Duo P7550, no SSE4.2 | Debian amd64 is plain x86-64 baseline, so it runs. Android-for-PC builds (BlissOS 15/16, PrimeOS) need x86-64-v2 and will not boot. |
| GeForce 9400M, shared RAM | Xorg + `nouveau` (X11, not Wayland), XFCE with window compositing off, `mesa-vdpau-drivers` for VLC. Debian 13 is pinned: Mesa 25.3+ removed VDPAU. |
| Buttonless bcm5974 trackpad | libinput: tap-to-click and two-finger right-click (`clickfinger`). |
| Broadcom BCM4322 Wi-Fi, Bluetooth | in-kernel `brcmsmac` + `firmware-brcm80211`, `bluez`/`blueman`, NetworkManager applet. |
| Apple fans, battery | `mbpfan`, `tlp`, lid close suspends. |
| 7.5 GB RAM, SSD | zram swap, weekly `fstrim` (Debian default), `noatime`. |
| Quebec | `fr_CA.UTF-8`, `America/Toronto`, `ca` keyboard layout (change in Settings if your keyboard differs). |

Apps: Firefox ESR (French), VLC, Thunar, Mousepad, Ristretto (images), Atril (PDF),
File Roller, Galculator, AbiWord, Gnumeric, GParted, XFCE terminal.

## Get the ISO

Build it on GitHub (no local setup): **Actions -> build-iso -> Run workflow**,
then download the `mbp55-linux-iso` artifact. Pushing a tag like `v0.1.0`
publishes it as a release asset.

Local build (Debian host with `live-build`, root): `sudo lb config && sudo lb build`

## Install over a partition

1. Write the ISO to a USB stick (`dd if=...iso of=/dev/sdX bs=4M conv=fsync`; check the device with `lsblk`).
2. Hold **Option** at power-on and pick the EFI boot entry for the stick. The live desktop starts automatically (user `user`, passwordless sudo).
3. Open a terminal and run:

   ```sh
   sudo mbp-install /dev/sda3
   ```

   It refuses the EFI partition, mounted/swap partitions and the live medium,
   expects the partition label `rootBodhi` (override with `--force`), and
   requires you to type `ERASE /dev/sda3`. **Only that partition is formatted.**
   It asks for a username and password for the new system, then prints a GRUB
   menu entry. Add it to `/etc/grub.d/40_custom` of your main Linux and run
   `sudo update-grub`. `--with-efi` also installs GRUB into `EFI/mbp55` on the
   ESP without touching the default boot entry.

4. Optional: `sudo mbp-nouveau-fw` (needs network) extracts the VP3 video firmware
   from NVIDIA's 325.15 driver on your own machine, for hardware decoding in VLC.
   Check with `vdpauinfo | grep -i h264`.

## Status

The configuration builds and is linted; behaviour on the real machine (Apple EFI
boot, suspend/resume with nouveau, Wi-Fi, nouveau firmware extraction) has not
been tested. If Wi-Fi is flaky with `brcmsmac`, the `broadcom-sta-dkms` (wl) driver
is the usual alternative. Please open issues with logs.

## License

MIT
