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
/usr/bin/ovs-dpdk/ovsdb-tool create /usr/local/etc/openvswitch/conf.db /usr/bin/ovs-dpdk/vswitch.ovsschema
# start
/usr/bin/ovs-dpdk/ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
  --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
  --pidfile=/tmp/ovsdb-server.pid --detach --log-file=/var/log/openvswitch/ovs-vswitchd.log

export DB_SOCK=/usr/local/var/run/openvswitch/db.sock
/usr/bin/ovs-dpdk/ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true
export SOCK_MEM=2048
/usr/bin/ovs-dpdk/ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-socket-mem="$SOCK_MEM"
# Define Cores for OVS Operations
export OVS_SERVICE_MASK=0x1
export OVS_CORE_MASK=0xFC
/usr/bin/ovs-dpdk/ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-lcore-mask=$OVS_SERVICE_MASK
/usr/bin/ovs-dpdk/ovs-vsctl --no-wait set Open_vSwitch . other_config:pmd-cpu-mask=$OVS_CORE_MASK
/usr/bin/ovs-dpdk/ovs-vsctl --no-wait set Open_vSwitch . other_config:emc-insert-inv-prob=1
/usr/bin/ovs-dpdk/ovs-vswitchd unix:$DB_SOCK --pidfile --detach

/usr/bin/ovs-dpdk/ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev
/usr/bin/ovs-dpdk/ovs-vsctl add-port br0 mac3 -- set Interface mac3 type=dpdk options:dpdk-devargs=dpni.2
/usr/bin/ovs-dpdk/ovs-vsctl add-port br0 mac4 -- set Interface mac4 type=dpdk options:dpdk-devargs=dpni.3

# тут я остановился, потому что прописывать правила маршрутизации было не нужно
/usr/bin/ovs-dpdk/ovs-vsctl add-port br0 vhost-user1 -- set Interface vhost-user1 type=dpdkvhostuser
/usr/bin/ovs-dpdk/ovs-vsctl add-port br0 vhost-user2 -- set Interface vhost-user2 type=dpdkvhostuser
/usr/bin/ovs-dpdk/ovs-ofctl del-flows br0

/usr/bin/ovs-dpdk/ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=1,actions=output:2
/usr/bin/ovs-dpdk/ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=2,actions=output:1

/usr/bin/ovs-dpdk/ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=1,actions=output:3
/usr/bin/ovs-dpdk/ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=3,actions=output:1
/usr/bin/ovs-dpdk/ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=2,actions=output:4
/usr/bin/ovs-dpdk/ovs-ofctl add-flow br0 -O OpenFlow13 table=0,in_port=4,actions=output:2

```