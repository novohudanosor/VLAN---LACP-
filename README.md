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
* nano
* traceroute
* tcpdump
* net-tools
3. Установка пакетов на CentOS 8 Stream:  ``` yum install -y nano traceroute tcpdump net-tools  ```
4. Установка пакетов на Ubuntu 22.04:  ``` apt install -y vim traceroute tcpdump net-tools ```


  

