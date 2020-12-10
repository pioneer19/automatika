### Hugepages
in grub
```
isolcpus=2,4,6
default_hugepagesz=1G hugepagesz=1G hugepages=4
hugepages=1024
```
or in runtime 
```
echo 512 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
# or for NUMA server
echo 1024 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
echo 1024 > /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
```
### загрузка модуля ядра карты Mellanox
```
sudo rmmod mlx4_en mlx4_ib mlx4_core

sudo modprobe mlx4_core num_vfs=1 log_num_mgm_entry_size=-7
sudo modprobe mlx4_en
```
### карты Intel
```
UUID_VF=$(uuidgen)
sudo modprobe vfio-pci enable_sriov=1
dpdk-devbind.py -b vfio-pci 0000:09:00.0
dpdk-devbind.py -b vfio-pci 0000:0a:00.0
```
### run testpmd
```
# testpmd -l 8-15 -n 4 -a 0000:83:00.0 -a 0000:84:00.0 -- --rxq=2 --txq=2 -i
# sudo dpdk-testpmd -l 2-3 -n 2 -a 0000:01:00.0 -- -i
# mellanox
sudo dpdk-testpmd -l 2-3 -n 2 --vdev net_af_xdp,iface=enp1s0 -- -i
# intel
sudo dpdk-testpmd -l 2-3 -n 2 -a 09:00.0 -a 0a:00.0 --file-prefix=pf -- -i
```
