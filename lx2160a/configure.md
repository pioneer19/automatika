### подмонтировать hugetlbfs, если они не подмонтированы
Это не нужно, потому что скрипт dynamic_dpl.sh сделает это сам.
```
# mkdir /mnt/hugepages
# mount -t hugetlbfs none /mnt/hugepages
```
# сделать два интерфейса видимыми для DPDK
```
cd /usr/share/dpdk/dpaa2
. ./dynamic_dpl.sh dpmac.3 dpmac.4
#. ./dynamic_dpl.sh dpmac.4
```
# другие полезные команды
```
# посмотреть существующие скрипты-обертки
ls-main
# создать интерфейс (dynamic_dpl.sh делает что-то еще полезное, кроме этого)
ls-addni dpmac.4
# посмотреть информацию по интерфейсу
restool dpni info dpni.2
# отвязать интерйес от ядра
echo dpni.0 > /sys/bus/fsl-mc/drivers/fsl_dpaa2_eth/unbind
# удалить интерфейс
restool dpni destroy dpni.0
```