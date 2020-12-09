### Hugepages
in grub
```
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
