```
mkdir /mnt/hugepages
mount -t hugetlbfs none /mnt/hugepages

export DPAA_NUM_RX_QUEUES=1
fmc -x
cd /usr/share/dpdk/dpaa
fmc -c ./usdpaa_config_ls1043.xml -p ./usdpaa_policy_hash_ipv4_1queue.xml -a

```
