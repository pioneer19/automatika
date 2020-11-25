```
testpmd -l 1-3 -n 1 -- -i --portmask=0x3 --nb-cores=2
```
Interactive commands
```
show port info 0
show port info all

clear port stats all

show fwd stats all

set port tm hierarchy default (port_id)
```
```
sudo ip n add 10.128.1.1 lladdr b8:27:eb:e8:bd:26 dev enp1s0
```
