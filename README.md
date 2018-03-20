# bestArch

Configurations and todos to make your Arch Linux the best Arch Linux


- [System](#system)
  - [Use systemd-boot](#use-systemd-boot)
  - [Compress initramfs with lz4](#compress-initramfs-with-lz4)
  - [Change IO Scheduler](#change-io-scheduler)
- [Networking](#networking)
  - [DNSCrypt](#dnscrypt)
- [Graphics](#graphics)
  - [NVIDIA driver DRM kernel mode setting](#nvidia-driver-drm-kernel-mode-setting)

# System

## Use systemd-boot

## Compress initramfs with lz4

## Change IO Scheduler

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

Edit `/etc/mkinitcpio.conf` prepending to the MODULES array (delimited by `()`) the following: `nvidia nvidia_modeset nvidia_uvm nvidia_drm`.

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