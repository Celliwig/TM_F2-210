# TM_F2-210
TerraMaster F2-210 code

This repo contains various bits of code related to the TerraMaster F2-210.

## Compilation
Assuming cross-compile installed.

```
export CROSS_COMPILE=aarch64-linux-gnu-
ARCH=arm64 make distclean
ARCH=arm64 make realtek_defconfig
ARCH=arm64 make -j 8
```

## Testing
For testing purposes, a kernel image can be loaded by:
```
mkimage -A arm64 -O linux -T kernel -C gzip -a 0x04000000 -e 0x04000000 -n "Test kernel" -d <kernel dir>/arch/arm64/boot/Image.gz uImage.krnl

Load Kernel:
loady 0x08000000 (kernel uImage)
loady 0x02100000 (fdt dtb)
setenv bootargs 'earlycon=uart8250,mmio32,0x98007800 console=ttyS0,115200 loglevel=7 debug clk_ignore_unused pd_ignore_unused'
bootm 0x08000000 - 0x02100000
```

## Debian install
Installing Debian. From the ARM64 installer ISO, get 'initrd.gz' from '/install.a64/'.
```
mkimage -A arm64 -O linux -T kernel -C gzip -a 0x06000000 -e 0x06000000 -n "5.4 kernel" -d Image.gz uImage.F2-210.kernel
mkimage -A arm64 -O linux -T ramdisk -C none -a 0x2200000 -n "Debian Installer" -d initrd.gz uImage.F2-210.di-initrd

Load Kernel/InitRD:
loady 0x08000000 (kernel uImage)
loady 0x18000000 (initrd uImage)
loady 0x02100000 (fdt dtb)
setenv initrd_high 0xffffffffffffffff
setenv bootargs 'earlycon=uart8250,mmio32,0x98007800 console=ttyS0,115200 loglevel=7 debug clk_ignore_unused pd_ignore_unused'
bootm 0x08000000 0x18000000 0x02100000
```
Add 'DEBIAN_FRONTEND=text' to the bootargs, to get a simpler interface (if you've only got a basic terminal that doesn't support ncurses).

## Disk boot:
loady 0x08000000 (kernel)
loady 0x02100000 (fdt)
setenv bootargs 'earlycon=uart8250,mmio32,0x98007800 console=ttyS0,115200 clk_ignore_unused pd_ignore_unused root=/dev/sda2 ro'
bootm 0x08000000 - 0x02100000

## Existing drivers
Drivers already exist for:
* RTC
* Watchdog
* R8169 (PCI)

Should probably replace Realtek's, with main line drivers.

## Debugging
Code stub to write an 'H' to the serial port (good for debugging early stage kernel).

## Writing character to UART
	mov     x23, #'H'
#	mov     x24, #0x98007800
	ldr     x24, =(0x98007800)
	strb    w23, [x24]

## Helpful links
https://www.modders-inc.com/terra-master-f2-210-nas-review/
https://www.cnx-software.com/2018/09/23/realtek-rtd1296-u-boot-linux-source-code-rtd1619-cortex-a55-soc/
https://blog.danman.eu/zidoo-x8-recovery/
