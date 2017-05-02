
# How to clone virtualbox vm

1. clone vm
* select linked clone
* check Reinitialize mac address
* input vm new host

2. clear mac address bindings

`rm -rf /etc/udev/rules.d/70-persistent-net.rules`

3. change ip
`vim /etc/sysconfig/network-scripts/ifcfg-eth1`

change 192.168.56.10 to 192.168.56.11

3. rename name of host

`vim /etc/sysconfig/network`

change vm0 to vm1

reboot & enjoy!

