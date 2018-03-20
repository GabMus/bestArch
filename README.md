# bestArch

Configurations and todos to make your Arch Linux the best Arch Linux


- [System](#system)
  - [Use systemd-boot](#use-systemd-boot)
  - [Compress initramfs with lz4](#compress-initramfs-with-lz4)
  - [Change IO Scheduler](#change-io-scheduler)
- [Networking](#networking)
  - [DNSCrypt](#dnscrypt)

# System

## Use systemd-boot

## Compress initramfs with lz4

## Change IO Scheduler

# Networking

## DNSCrypt

[DNSCrypt on Arch Wiki](https://wiki.archlinux.org/index.php/DNSCrypt)

Encrypt your DNS traffic so your ISP can't spy on you.

### Install

```bash
sudo pacman -S dnscrypt-proxy
```

### Configure

`sudo vim /etc/dnscrypt-proxy.conf` and change the `ResolverName` row as follows: `ResolverName dnscrypt.eu-nl`

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