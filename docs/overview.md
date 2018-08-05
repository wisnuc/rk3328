# Overview

There are several "upstreams" for engineering the embedded system on rk3328.

Rockchip has official open source web site and documentation. 

Pine64's ROCK64 sbc has been around for quite a while. 

ROCK64 is also supported by armbian community officially.

Recently, Firefly also launched a rk3328-based sbc, codenamed ROC-RK3328-CC, which is very similar to ROCK64. Libre Computer re-branded it as Renegade board on western market.

The choices and possibilities are abundant for design. To minimize the development and maintenance cost, we should keep in mind that our approach should NOT repeat any work that has been done by rockchip, armbian, and pine64 community. 

The modification to existing software should be kept as little as possible. It is the best to utilize u-boot and kernel source without any modification.

# Layout and Components

rk3328 emmc layout is divided into several regions and (GPT) partitions. A partition table is required for kernel to mount rootfs. The kernel is put in a separate raw partition (boot.img), which is commonly used for android.

Such layout can be kept mostly, except that we need a different boot strategy and more rootfs and data partitions.

## A/B model for boot and firmware upgrade

We need two rootfs partitions, aka, A and B for two version of firmwares. Another  partition (`data partition`) are used for persistent data surviving firmware upgrades.

The boot.img should have minimal drivers built-in, as well as an initramfs for:

1. boot from USB drive
2. boot from A or B according to flags stored in data partition

## firmwares

Storing firmware in data partition allows firmware upgrade without a disk.

If overlayroot is not implemented, data partition should be large enough to store 2 firmwares. One for latest and one for tempory downloading file.

If overlayroot is implemented, then no space for storing firmware is required since the file can be extracted to A or B partition on the fly.

## factory reset

If overlayroot is not implemented, a factory reset means:

1. erasing the other rootfs partition with latest firmwware
2. erasing all user data
3. flip the boot flag
4. kexec to the other rootfs

If overlay root is implemented, the first step is avoided.

## user data

user data should have two copies, one for A and the other for B. This ensures a flip operation is atomic.

# Kernel

The boot.img kernel should be built separately, including an initramfs for:

1. boot from USB
2. boot from A or B

The rootfs kernel should come from armbian distribution.

# USB Boot

This function can be pushed to late stage. In early stage, mmc card is suffice for developing purpose.

USB boot is used for two purpose.

## Running system directly from USB drive.

In this way, the USB should have an ext4 partition with rootfs. The ext4 partition should have a special name and UUID.

## Flashing emmc

Forcefully update rootfs









