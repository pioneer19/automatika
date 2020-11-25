### загрузка с SD
```
cpld reset sd
```
Then wait for "Hit any key to stop autoboot" and press key to interrupt boot.
```
setenv othbootargs default_hugepagesz=2MB hugepagesz=2MB hugepages=512 isolcpus=1-3 bportals=s0 qportals=s0 iommu.passthrough=1
setenv dtb fsl-ls1043a-rdb-usdpaa.dtb
boot
```
#### tftp boot
```
setenv ipaddr   192.168.16.171
setenv serverip 192.168.16.15
#setenv bootargs root=/dev/ram0 rw console=ttyS0,115200 earlycon=uart8250,mmio,0x21c0500 ramdisk_size=0x10000000 default_hugepagesz=2MB hugepagesz=2MB hugepages=512 isolcpus=1-3 bportals=s0 qportals=s0 iommu.passthrough=1
setenv bootargs root=/dev/mmcblk0p1 rw console=ttyS0,115200 earlycon=uart8250,mmio,0x21c0500 ramdisk_size=0x10000000 default_hugepagesz=2MB hugepagesz=2MB hugepages=512 isolcpus=1-3 bportals=s0 qportals=s0 iommu.passthrough=1

tftp 0x82000000 Image-ls1043ardb.bin
tftp 0xa0000000 fsl-image-networking-ls1043ardb.ext2.gz.u-boot
tftp 0x8f000000 fsl-ls1043a-rdb-usdpaa-ls1043ardb.dtb

#booti 0x82000000 0xa0000000 0x8f000000
booti 0x82000000 - 0x8f000000
```
### l2fwd
```
l2fwd -l1,2,3 -n 1 -- -p 0x23 -q 1 -T 0
```
### testpmd
`testpmd -l 1-3 -n 1` - shows dpaa ports
### pktgen
```
# pktgen -l 1-3 -n 1 --proc-type auto --socket-mem 2 --file-prefix pg --log-level 8 -- -T -P -m "2.[0-6]"
# pktgen -l 1-3 -n 1 -m 32 --proc-type auto --file-prefix pg --log-level 8 -- -T -P -m "2.[0-6]"
#pktgen -l 1-3 -n 1 --proc-type auto --file-prefix pg --log-level 8 -- -T -P  -m "[2].6, [3].1"
pktgen -l 1-2 -n 1 --proc-type auto --file-prefix pg --log-level 8 -- -T -P  -m "[2].6"
```
```
set 6 src ip 10.128.1.17/24
set 6 dst ip 10.128.1.2
set 6 dst mac 00:02:c9:08:a8:d0
start 0
```
### ovs
```
# clear old configs
rm /usr/local/etc/openvswitch/conf.db
rm -rf /usr/local/var/run/openvswitch/vhost-user-1
rm -rf /usr/local/var/run/openvswitch/vhost-user-2
mkdir -p /usr/local/etc/openvswitch
mkdir -p /var/log/openvswitch
mkdir -p /usr/local/var/run/openvswitch
# create db
/usr/local/bin/ovsdb-tool create /usr/local/etc/openvswitch/conf.db /usr/local/share/openvswitch/vswitch.ovsschema
# start
/usr/local/sbin/ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
  --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
  --pidfile=/tmp/ovsdb-server.pid --detach --log-file=/var/log/openvswitch/ovs-vswitchd.log

export DB_SOCK=/usr/local/var/run/openvswitch/db.sock
/usr/local/bin/ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true
export SOCK_MEM=200
/usr/local/bin/ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem="$SOCK_MEM"
# Define Cores for OVS Operations
export OVS_SERVICE_MASK=0x1
export OVS_CORE_MASK=0x6
/usr/local/bin/ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-lcore-mask=$OVS_SERVICE_MASK
/usr/local/bin/ovs-vsctl --no-wait set Open_vSwitch . other_config:pmd-cpu-mask=$OVS_CORE_MASK
ovs-vsctl --no-wait set Open_vSwitch . other_config:emc-insert-inv-prob=1
/usr/local/sbin/ovs-vswitchd unix:$DB_SOCK --pidfile --detach
```
net: dpaa: fm1-mac1: 00:04:9f:04:1d:b5
net: dpaa: fm1-mac2: 00:04:9f:04:1d:b6
net: dpaa: fm1-mac3: 00:04:9f:04:1d:b7
net: dpaa: fm1-mac4: 00:04:9f:04:1d:b8
net: dpaa: fm1-mac5: 00:04:9f:04:1d:b9
net: dpaa: fm1-mac6: 00:04:9f:04:1d:ba
net: dpaa: fm1-mac9: 00:04:9f:04:1d:bb
```
ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
ovs-vsctl add-port br0 dpdk0 -- set Interface dpdk0 type=dpdk options:dpdk-devargs=fm1-mac1
ovs-vsctl add-port br0 dpdk1 -- set Interface dpdk1 type=dpdk options:dpdk-devargs=fm1-mac2

ovs-vsctl add-port br0 vhost-user1 -- set Interface vhost-user1 type=dpdkvhostuser
ovs-vsctl add-port br0 vhost-user2 -- set Interface vhost-user2 type=dpdkvhostuser
ovs-ofctl del-flows br0

ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=1,actions=output:2
ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=2,actions=output:1

ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=1,actions=output:3
ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=3,actions=output:1
ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=2,actions=output:4
ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=4,actions=output:2

```
### hugetlbfs
```
mkdir /mnt/hugepages
mount -t hugetlbfs none /mnt/hugepages
```
