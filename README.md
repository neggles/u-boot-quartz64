# u-boot-quartzpro64 (RK3588 boards)
==========

This is the Rockchip downstream ("BSP") version of u-boot for the RK3588,
with patches on top as I see fit for the Pine64 QuartzPro64 & Radxa Rock5 B boards.

For the original u-boot README, see the "README" file.

This repository is provided "as-is", I may delete it at any time or abandon it,
no guarantees about its quality, functionality or fitness-for-purpose are provided.

## Building

Make sure you have an arm64 cross-compiler in `$PATH`; the prefix should be `aarch64-linux-gnu-`, 
e.g. `aarch64-linux-gnu-gcc`. Most distro compilers use this prefix. 
If your cross-compiler's prefix is different, you will need to edit `make.sh` appropriately.

1. Clone this repository and enter the repo:

```sh
git clone --branch=rk3588 https://github.com/neggles/u-boot-quartz64
cd u-boot-quartz64
```

2. Run the u-boot build, using defaults:

```sh
make mrproper
./make.sh rk3588
```

## Next Steps

Flash `rk3588_spl_loader_v*.bin` and `uboot.img` to your SD card / eMMC. Details can be found 
[on the pine64 wiki here](https://wiki.pine64.org/wiki/QuartzPro64_Development)
along with with [some other info around debian installation](https://wiki.pine64.org/wiki/Installing_Debian_on_the_Quartz64)
(not specific to this board), but here's five-second howto:

### **⚠️❗ THIS WILL ERASE EVERYTHING ON THE TARGET DEVICE. ❗⚠️ **

```sh
sd_path='/dev/somedevice' # e.g. /dev/mmcblk0, /dev/sda - triple-check this!

# Make partition table on device
sudo parted -s -a optimal -- "$sd_path" \
    mklabel gpt \
    mkpart loader 64s 8MiB \
    mkpart uboot 8MiB 16MiB \
    mkpart env 16MiB 32MiB \
    mkpart efi fat32 32MiB 544MiB \
    set 4 esp on \
    set 4 boot on \
    mkpart root ext4 544MiB 100%

# zero out loader/uboot
sudo dd if=/dev/zero of="${sd_path}p1"
sudo dd if=/dev/zero of="${sd_path}p2"

# format env and efi
sudo mkfs.fat -F 32 "${sd_path}p3"
sudo mkfs.fat -F 32 -n "EFI" "${sd_path}p4"

# format rootfs
sudo mkfs.ext4 "${sd_path}p5"

# apply loader to loader part
sudo dd if=rk3588_spl_loader_v*.bin of="${sd_path}p1" bs=64K

# apply u-boot to u-boot part
sudo dd if=uboot.img of="${sd_path}p2" bs=64K
```

How you set up /boot and / on that card is left as an exercise for the reader, but the debian install page and/or the [quartzpro64 development page](https://wiki.pine64.org/wiki/QuartzPro64_Development#U-Boot_+_Kernel_On_SD,_RootFS_On_eMMC) on the wiki should give you some idea.
