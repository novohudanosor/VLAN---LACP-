# VLAN---LACP-
Сетевые пакеты. VLAN'ы. LACP 
* Описание домашнего задания
в Office1 в тестовой подсети появляется сервера с доп интерфейсами и адресами
в internal сети testLAN: 
- testClient1 - 10.10.10.254
- testClient2 - 10.10.10.254
- testServer1- 10.10.10.1 
- testServer2- 10.10.10.1

Развести вланами:
testClient1 <-> testServer1
testClient2 <-> testServer2

Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов
*По итогу выполнения домашнего задания у нас должна получиться следующая топология сети:
1. ![alt text](./Pictures/1.png)
2. Перед настройкой VLAN и LACP рекомендуется установить на хосты следующие утилиты:
* nano или vim
* traceroute
* tcpdump
* net-tools
3. Установка пакетов на CentOS 8 Stream:  ``` yum install -y nano vim traceroute tcpdump net-tools  ```
4. Установка пакетов на Ubuntu 22.04:  ``` apt install -y nano vim traceroute tcpdump net-tools ```
5. Настройка VLAN на хостах
6. На хосте testClient1 требуется создать файл /etc/sysconfig/network-scripts/ifcfg-vlan1 со следующим параметрами:
```
VLAN=yes
TYPE=Vlan
PHYSDEV=eth1
VLAN_ID=1
VLAN_NAME_TYPE=DEV_PLUS_VID_NO_PAD
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=10.10.10.254
PREFIX=24
NAME=1
DEVICE=eth1.1
ONBOOT=yes
```
7. На хосте testServer1 создадим идентичный файл с другим IP-адресом (10.10.10.1)
8.  После создания файлов нужно перезапустить сеть на обоих хостах:  ``` systemctl restart NetworkManager ```
9. Настройка VLAN на Ubuntu. На хосте testClient2 требуется создать файл /etc/netplan/50-cloud-init.yaml со следующим параметрами:
```
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8: {}
    vlans:
        vlan2:
          id: 2
          link: enp0s8
          dhcp4: no
          addresses: [10.10.10.254/24]
    version: 2
```
10. На хосте testServer2 создадим идентичный файл с другим IP-адресом (10.10.10.1).
11. После создания файлов нужно перезапустить сеть на обоих хостах: ``` netplan apply ```
12. После настройки второго VLAN`а ping должен работать между хостами testClient1, testServer1 и между хостами testClient2, testServer2. **Примечание:** до остальных хостов ping работать не будет, так как не настроена маршрутизация.
```
[root@testClient1 ~]# ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.230 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.374 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.353 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.341 ms
^C
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.230/0.324/0.374/0.058 ms

vagrant@testClient2:~$ ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.224 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.285 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.321 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.247 ms
^C
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3066ms
rtt min/avg/max/mdev = 0.224/0.269/0.321/0.036 ms
```
## Настройка LACP между хостами inetRouter и centralRouter
1. Bond интерфейс будет работать через порты eth1 и eth2. Изначально необходимо на обоих хостах добавить конфигурационные файлы для интерфейсов eth1 и eth2   ``` nano /etc/sysconfig/network-scripts/ifcfg-eth1 ```
```
DEVICE=eth1
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=yes
USERCTL=no
```
* У интерфейса ifcfg-eth2 идентичный конфигурационный файл, в котором нужно изменить имя интерфейса.
2. После настройки интерфейсов eth1 и eth2 нужно настроить bond-интерфейс, для этого создадим файл ```nano /etc/sysconfig/network-scripts/ifcfg-bond0 ```
```
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.255.1
NETMASK=255.255.255.252
ONBOOT=yes
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
NM_CONTROLED=yes
```
* После создания данных конфигурационных файлов необходимо перезапустить сеть: ``` systemctl restart NetworkManager ```  **На некоторых версиях RHEL/CentOS перезапуск сетевого интерфейса не запустит bond-интерфейс, в этом случае рекомендуется перезапустить хост.**
* После настройки агрегации портов, необходимо проверить работу bond-интерфейса, для этого, на хосте inetRouter пустим пинг до centralRouter
```
[root@centralRouter ~]# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
eth0             UP             52:54:00:c9:c7:04 <BROADCAST,MULTICAST,UP,LOWER_UP> 
bond0            UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> 
eth1             UP             08:00:27:97:b2:63 <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> 
eth2             UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> 
eth3             UP             08:00:27:62:11:57 <BROADCAST,MULTICAST,UP,LOWER_UP> 
eth4             UP             08:00:27:03:f4:d5 <BROADCAST,MULTICAST,UP,LOWER_UP> 

[root@inetRouter ~]# ping 192.168.255.1
PING 192.168.255.1 (192.168.255.1) 56(84) bytes of data.
64 bytes from 192.168.255.1: icmp_seq=1 ttl=64 time=0.010 ms
64 bytes from 192.168.255.1: icmp_seq=2 ttl=64 time=0.037 ms
64 bytes from 192.168.255.1: icmp_seq=3 ttl=64 time=0.026 ms
64 bytes from 192.168.255.1: icmp_seq=4 ttl=64 time=0.027 ms
64 bytes from 192.168.255.1: icmp_seq=5 ttl=64 time=0.027 ms
64 bytes from 192.168.255.1: icmp_seq=6 ttl=64 time=0.030 ms

[root@centralRouter ~]# ip link set down eth1
[root@centralRouter ~]# ip -br link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
eth0             UP             52:54:00:c9:c7:04 <BROADCAST,MULTICAST,UP,LOWER_UP> 
bond0            UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> 
eth1             DOWN           08:00:27:97:b2:63 <BROADCAST,MULTICAST,SLAVE> 
eth2             UP             08:00:27:db:b1:00 <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> 
eth3             UP             08:00:27:62:11:57 <BROADCAST,MULTICAST,UP,LOWER_UP> 
eth4             UP             08:00:27:03:f4:d5 <BROADCAST,MULTICAST,UP,LOWER_UP> 

...
64 bytes from 192.168.255.1: icmp_seq=60 ttl=64 time=0.040 ms
64 bytes from 192.168.255.1: icmp_seq=61 ttl=64 time=0.035 ms
64 bytes from 192.168.255.1: icmp_seq=62 ttl=64 time=0.038 ms
64 bytes from 192.168.255.1: icmp_seq=63 ttl=64 time=0.036 ms
64 bytes from 192.168.255.1: icmp_seq=64 ttl=64 time=0.024 ms
64 bytes from 192.168.255.1: icmp_seq=65 ttl=64 time=0.024 ms
64 bytes from 192.168.255.1: icmp_seq=66 ttl=64 time=0.024 ms
...

[root@centralRouter ~]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth2
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:db:b1:00
Slave queue ID: 0

Slave Interface: eth1
MII Status: down
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 08:00:27:97:b2:63
Slave queue ID: 0

```





  

