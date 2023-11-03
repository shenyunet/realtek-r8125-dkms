# NOTE

Based on [awesometic/realtek-r8125-dkms](https://github.com/awesometic/realtek-r8125-dkms), enable multi TX queue, enable RSS support, and disable ASPM. PVE 8 need to install `pve-headers` and `dkms` before installing `realtek-r8125-dkms`.

基于[awesometic/realtek-r8125-dkms](https://github.com/awesometic/realtek-r8125-dkms)修改，启用TX多队列，启用RSS，禁用ASPM。PVE8用户在安装`realtek-r8125-dkms`前需要先安装`pve-headers`和`dkms`。

# Realtek r8125 DKMS

![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/devome/realtek-r8125-dkms?sort=semver&style=for-the-badge)

This provides Realtek r8125 driver in DKMS way so that you can keep the latest driver even after the kernel upgrade.

## Before use

This DKMS package is for Realtek RTL8125 (r8125 in module name) Ethernet, which is designed for the PCI interface.

If you are searching for the Realtek 2.5 Gbits **USB Ethernet**, which may use RTL8156 (r8152 in module name), you are in the wrong place. Please refer to another DKMS project for that Realtek driver.

- [Realtek R8152 DKMS](https://github.com/awesometic/realtek-r8152-dkms)

## Installation

There are 3 ways to install this DKMS module. Choose one as your tastes.

Those are not interfering with each other. So you can do all 3 methods but absolutely you don't need to.

Installation using the Debian package is recommended for the sake of getting the newer driver.

### Debian package

### dkms-install.sh

This script is from aircrack-ng team. You can install the DKMS module by a simple command.

Download or clone this repository and move to the extracted directory, then run the script.

```bash

apt install build-* dkms pve-headers-$(uname -r)
git clone https://github.com/shenyunet/realtek-r8125-dkms.git
cd realtek-r8125-dkms
sudo ./dkms-install.sh
```

## Verify the module is loaded successfully

After installing the DKMS package, you may not be able to use the new `r8125` module on the fly. This because the existing `r8169` module will be loaded priority to `r8125` so that it prevents working of the `r8125` module.

Check if the `r8169` module loaded currently.

```bash
lsmod | grep -i r8169
```

If there is a result, maybe the `r8125` module wasn't loaded properly. You can check out modules currently in use via `lspci -k` or `dmesg` too.

To use the `r8125` module explicitly you can add the `r8169` module to not be loaded by adding it to a blacklist file.

Enter the following command to configure the blacklist.

```bash
sudo tee -a /etc/modprobe.d/blacklist-r8169.conf > /dev/null <<EOT
# To use r8125 driver explicitly
blacklist r8169
EOT
```

To apply the new blacklist to your kernel, update your initramfs via 

```bash
sudo update-initramfs -u
```

Finally, reboot to take effect.

> - If you need to load both r8169 and r8125, maybe removing r8125 firmware could make it work. Please enter `sudo rm -f /lib/firmware/rtl_nic/rtl8125*` to remove all the r8125 firmwares on the system. But it is just a workaround, you should have to do this every time installing the new kernel version or new Linux firmware.
> - In the case of the Debian package, I will update the scripts to make it do this during the installation.

## Debian package build

You can build yourself this after installing some dependencies including `dkms`.

```bash
sudo apt install debmake debhelper dkms dh-dkms
```

```bash
dpkg-buildpackage -b -rfakeroot -us -uc
```

## LICENSE

GPL-2 on Realtek driver and the debian packaing.

## References

- [Realtek r8125 driver release page](https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software)
- [ParrotSec's realtek-rtl88xxau-dkms, where got hint from](https://github.com/ParrotSec/realtek-rtl88xxau-dkms)
