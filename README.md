# bestArch

Configurations and todos to make your Arch Linux the best Arch Linux


- [System](#system)
  - [Use systemd-boot](#use-systemd-boot)
  - [Microcode updates](#microcode-updates)
  - [Compress initramfs with lz4](#compress-initramfs-with-lz4)
  - [Change IO Scheduler](#change-io-scheduler)
  - [Change CPU governor](#change-cpu-governor)
- [Package Management](#package-management)
  - [Switch to better mirrors](#switch-to-better-mirrors)
- [Networking](#networking)
  - [DNSCrypt](#dnscrypt)
- [Graphics](#graphics)
  - [NVIDIA driver DRM kernel mode setting](#nvidia-driver-drm-kernel-mode-setting)

# System

## Use systemd-boot

[Arch Wiki reference](https://wiki.archlinux.org/index.php/Systemd-boot#Installation)

Requires you to be able to boot in UEFI mode (not MBR).

You need to have a `/boot` partition formatted in FAT32 (usually I make it 400 MBs, even if it's a little too much).

Assuming you have all your file systems mounted to their proper locations AND that you are already chroot-ed in your installed system.

```bash
sudo bootctl --path=/boot install
```

Create `/boot/loader/entries/arch.conf` like follows:

```
title		Arch Linux
linux		/vmlinuz-linux
# uncomment this in case you want to install intel microcode
# initrd		/intel-ucode.img
initrd		/initramfs-linux.img
options		root=UUID=ROOT_PARTITION_UUID rw quiet # nvidia-drm.modeset=1 # uncomment this if/when you want to enable nvidia drm kernel mode setting
```

Where `ROOT_PARTITION_UUID` can be obtained from the command `lsblk -f` (use the UUID of the partition mounted as `/`).

You may want to edit `/boot/loader/loader.conf` to set a timeout (format: `timeout TIME_IN_SECONDS`) and to add a line that says `default arch-*`.

Install [`systemd-boot-pacman-hook`](https://aur.archlinux.org/packages/systemd-boot-pacman-hook/)<sup>AUR</sup> to automatically update systemd-boot.

## Microcode updates

[Arch wiki reference](https://wiki.archlinux.org/index.php/Microcode#Enabling_Intel_microcode_updates)

### AMD

From Arch Wiki:

> For AMD processors the microcode updates are available in `linux-firmware`, which is installed as part of the `base` system. **No further action is needed.**

### Intel

```bash
sudo pacman -S intel-ucode
```

Edit `/boot/loader/entries/arch.conf` so that the first `initrd` line is the following:

```
initrd        /intel-ucode.img
```

## Compress initramfs with lz4

Make sure `lz4` is installed.

Edit `/etc/mkinitcpio.conf`:

- Add `lz4 lz4_compress` to the `MODULES` array (delimited by `()`)
- Uncomment or add the line saying `COMPRESSION="lz4"`
- Add a line saying `COMPRESSION_OPTIONS="-9"`

Run `sudo mkinitcpio -p linux` to apply the mkinitcpio.conf changes.

## Change IO Scheduler

## Change CPU governor

[Arch Wiki reference](https://wiki.archlinux.org/index.php/CPU_frequency_scaling)

*NB: the default governor is powersave and you may want to leave it as it is.*

Create `/etc/udev/rules.d/50-scaling-governor.rules` as follows:

```
SUBSYSTEM=="module", ACTION=="add", KERNEL=="acpi_cpufreq", RUN+=" /bin/sh -c ' echo performance > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor ' "
```

# Package Management

## Switch to better mirrors

[Arch Wiki reference](https://wiki.archlinux.org/index.php/Reflector)

```bash
sudo pacman -S reflector
sudo reflector --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

# Networking

## DNSCrypt

[Arch Wiki reference](https://wiki.archlinux.org/index.php/DNSCrypt)

Encrypt your DNS traffic so your ISP can't spy on you.

### Install

```bash
sudo pacman -S dnscrypt-proxy
```

### Configure

Edit `/etc/dnscrypt-proxy.conf` and change the `ResolverName` row as follows: `ResolverName dnscrypt.eu-nl`

*Note: you can find more "Resolvers" in `/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv` or [here](https://github.com/dyne/dnscrypt-proxy/blob/master/dnscrypt-resolvers.csv)*

Make sure dnscrypt can run without root privileges: edit `/usr/lib/systemd/system/dnscrypt-proxy.service` to include:

```
[Service]
DynamicUser=yes
```

Reload systemd daemons, enable and start service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dnscrypt-proxy.service
sudo systemctl start dnscrypt-proxy.service
```

Edit your NetworkManager configuration to point to the following IPs for respectively IPv4 and IPv6 DNSes:

```
127.0.0.1
::1
```

# Graphics

## NVIDIA driver DRM kernel mode setting

[Arch Wiki reference](https://wiki.archlinux.org/index.php/NVIDIA#DRM_kernel_mode_setting)

Edit `/boot/loader/entries/arch.conf` appending `nvidia-drm.modeset=1` to the `options` row.

Edit `/etc/mkinitcpio.conf` prepending to the `MODULES` array (delimited by `()`) the following: `nvidia nvidia_modeset nvidia_uvm nvidia_drm`.

Run `sudo mkinitcpio -p linux` to apply the mkinitcpio.conf changes.

Add a pacman hook to rebuild initramfs after an NVIDIA driver upgrade, create `/etc/pacman.d/hooks/nvidia.hook`:

```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia

[Action]
Depends=mkinitcpio
When=PostTransaction
Exec=/usr/bin/mkinitcpio -P
```

*NB: Make sure the Target package set in this hook is the one you have installed (`nvidia`, `nvidia-lts` or some other different or legacy driver package name).*

Edit `/etc/gdm/custom.conf` uncommenting the row that says `WaylandEnable=false` (enabling DRM kernel mode setting usually improves performance but enables NVIDIA Wayland support for GNOME, but currently NVIDIA Wayland performance is terrible and makes for an unusable experience. While this option is not mandatory, it's highly recommended).

