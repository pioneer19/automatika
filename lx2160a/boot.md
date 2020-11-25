### загрузка с mmc/SD
```
# Switch to FlexSPI NOR flash 0 (default):
qixis_reset
# Switch to FlexSPI NOR flash 1:
qixis_reset altbank
# Switch to SD:
qixis_reset sd
# Switch to eMMC:
qixis_reset emmc
```

#### tftp boot
```
qixis_reset altbank
# прерываем загрузку, когда попросят "Hit any key to stop autoboot: 10"
setenv ethact DPMAC17@rgmii-id

setenv ipaddr   192.168.16.171
setenv serverip 192.168.16.15

setenv bootargs console=ttyAMA0,115200 root=/dev/ram0 rw earlycon=pl011,mmio32,0x21c0000 ramdisk_size=0x2000000 default_hugepagesz=1024m hugepagesz=1024m hugepages=8 pci=pcie_bus_perf isolcpus=1-15 iommu.passthrough=1 rcupdate.rcu_cpu_stall_suppress=1
# root in USB
#setenv bootargs console=ttyAMA0,115200 root=/dev/sda4 rw rootdelay=10 earlycon=pl011,mmio32,0x21c0000 ramdisk_size=0x2000000 default_hugepagesz=1024m hugepagesz=1024m hugepages=8 pci=pcie_bus_perf isolcpus=1-15 iommu.passthrough=1 rcupdate.rcu_cpu_stall_suppress=1

# enable DPAA2 Ethernet in Linux
sf probe 0:0
sf read 0x80a00000 0xa00000 0x300000
sf read 0x80e00000 0xe00000 0x100000
fsl_mc start mc 0x80a00000 0x80e00000
sf read 0x80d00000 0xd00000 0x100000
fsl_mc lazyapply dpl 0x80d00000

#load tftp kernel, rootfs and dtb file
tftp 0x82000000 lx2160/Image-lx2160ardb.bin
tftp 0xa0000000 lx2160/fsl-image-networking-lx2160ardb.ext2.gz.u-boot
tftp 0x8f000000 lx2160/fsl-lx2160a-rdb.dtb

booti 0x82000000 0xa0000000 0x8f000000
# root in USB/SD
#booti 0x82000000 - 0x8f000000
```