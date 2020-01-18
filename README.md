# otus-vpn
VPN
1. Между двумя виртуалками поднять vpn в режимах
- tun
- tap
Прочуствовать разницу.

2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку

# Работоспособность

Поднимаем кластер `vagrant up`

На сервере посмотрим что запущено:
```
[vagrant@server ~]$ sudo -s
[root@server vagrant]# systemctl -t service | grep vpn
  openvpn-server@server-ras.service                     loaded active running OpenVPN service for server/ras
  openvpn-server@server-tap.service                     loaded active running OpenVPN service for server/tap
  openvpn-server@server-tun.service                     loaded active running OpenVPN service for server/tun
```
Сервер и клиент связаны тунелем в режиме tap интерфейсом br0
```
[root@server vagrant]# cat /etc/sysconfig/network-scripts/ifcfg-br0 
STP=no
TYPE=Bridge
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=192.168.20.10
PREFIX=24
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV4_DNS_PRIORITY=100
IPV6INIT=no
NAME=br0
DEVICE=br0
ONBOOT=yes

[root@clientTap vagrant]# cat /etc/sysconfig/network-scripts/ifcfg-br0 
STP=no
TYPE=Bridge
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=192.168.20.11
PREFIX=24
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV4_DNS_PRIORITY=100
IPV6INIT=no
NAME=br0
DEVICE=br0
ONBOOT=yes
```
Проведем тест iperf3 для tap
```
[root@server vagrant]# iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 192.168.20.11, port 37288
[  5] local 192.168.20.10 port 5201 connected to 192.168.20.11 port 37290
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  8.10 MBytes  68.0 Mbits/sec                  
[  5]   1.00-2.00   sec  8.97 MBytes  75.2 Mbits/sec                  
[  5]   2.00-3.01   sec  9.52 MBytes  79.3 Mbits/sec                  
[  5]   3.01-4.00   sec  9.31 MBytes  78.7 Mbits/sec                  
[  5]   4.00-5.00   sec  9.26 MBytes  77.6 Mbits/sec                  
[  5]   5.00-6.00   sec  8.53 MBytes  71.6 Mbits/sec                  
[  5]   6.00-7.00   sec  9.34 MBytes  78.4 Mbits/sec                  
[  5]   7.00-8.00   sec  8.21 MBytes  68.7 Mbits/sec                  
[  5]   8.00-9.00   sec  8.58 MBytes  72.1 Mbits/sec                  
[  5]   9.00-10.00  sec  9.37 MBytes  78.4 Mbits/sec                  
[  5]  10.00-11.00  sec  9.22 MBytes  77.3 Mbits/sec                  
[  5]  11.00-12.00  sec  8.61 MBytes  72.2 Mbits/sec                  
[  5]  12.00-13.00  sec  8.48 MBytes  71.3 Mbits/sec                  
[  5]  13.00-14.00  sec  9.33 MBytes  78.3 Mbits/sec                  
[  5]  14.00-15.00  sec  9.09 MBytes  76.0 Mbits/sec                  
[  5]  15.00-16.00  sec  8.60 MBytes  72.4 Mbits/sec                  
[  5]  16.00-17.01  sec  8.71 MBytes  72.7 Mbits/sec                  
[  5]  17.01-18.01  sec  9.34 MBytes  78.3 Mbits/sec                  
[  5]  18.01-19.01  sec  8.40 MBytes  70.6 Mbits/sec                  
[  5]  19.01-20.00  sec  9.20 MBytes  77.6 Mbits/sec                  
[  5]  20.00-21.01  sec  9.10 MBytes  75.9 Mbits/sec                  
[  5]  21.01-22.01  sec  8.61 MBytes  72.3 Mbits/sec                  
[  5]  22.01-23.01  sec  8.77 MBytes  73.4 Mbits/sec                  
[  5]  23.01-24.01  sec  8.91 MBytes  75.1 Mbits/sec                  
[  5]  24.01-25.00  sec  8.45 MBytes  71.3 Mbits/sec                  
[  5]  25.00-26.00  sec  9.43 MBytes  78.8 Mbits/sec                  
[  5]  26.00-27.00  sec  9.25 MBytes  77.9 Mbits/sec                  
[  5]  27.00-28.01  sec  9.11 MBytes  75.8 Mbits/sec                  
^C[  5]  28.01-28.37  sec  3.09 MBytes  71.5 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-28.37  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-28.37  sec   253 MBytes  74.8 Mbits/sec                  receiver


[root@clientTap vagrant]# iperf3 -c 192.168.20.10 -t 40 -i 5
Connecting to host 192.168.20.10, port 5201
[  4] local 192.168.20.11 port 37290 connected to 192.168.20.10 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  46.5 MBytes  78.0 Mbits/sec   32    198 KBytes       
[  4]   5.00-10.00  sec  44.0 MBytes  73.8 Mbits/sec   18    205 KBytes       
[  4]  10.00-15.00  sec  44.7 MBytes  74.9 Mbits/sec   36    191 KBytes       
[  4]  15.00-20.00  sec  44.4 MBytes  74.5 Mbits/sec   64    223 KBytes       
[  4]  20.00-25.00  sec  43.6 MBytes  73.1 Mbits/sec   67    170 KBytes       
[  4]  20.00-25.00  sec  43.6 MBytes  73.1 Mbits/sec   67    170 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-25.00  sec   254 MBytes  85.2 Mbits/sec  217             sender
[  4]   0.00-25.00  sec  0.00 Bytes  0.00 bits/sec                  receiver
```
Тест iperf3 для tun
```
[root@server vagrant]# iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.0.2, port 50712
[  5] local 10.10.0.1 port 5201 connected to 10.10.0.2 port 50714
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  3.83 MBytes  32.2 Mbits/sec                  
[  5]   1.00-2.01   sec  5.90 MBytes  49.2 Mbits/sec                  
[  5]   2.01-3.00   sec  5.22 MBytes  44.0 Mbits/sec                  
[  5]   3.00-4.00   sec  7.74 MBytes  65.0 Mbits/sec                  
[  5]   4.00-5.00   sec  8.12 MBytes  68.0 Mbits/sec                  
[  5]   5.00-6.00   sec  8.40 MBytes  70.6 Mbits/sec                  
[  5]   6.00-7.00   sec  8.52 MBytes  71.2 Mbits/sec                  
[  5]   7.00-8.01   sec  7.26 MBytes  60.4 Mbits/sec                  
[  5]   8.01-9.00   sec  8.22 MBytes  69.6 Mbits/sec                  
[  5]   9.00-10.01  sec  8.38 MBytes  69.8 Mbits/sec                  
[  5]  10.01-11.00  sec  8.73 MBytes  73.9 Mbits/sec                  
[  5]  11.00-12.00  sec  7.77 MBytes  65.2 Mbits/sec                  
[  5]  12.00-13.00  sec  8.60 MBytes  72.2 Mbits/sec                  
[  5]  13.00-14.00  sec  8.29 MBytes  69.4 Mbits/sec                  
[  5]  14.00-15.00  sec  9.13 MBytes  76.8 Mbits/sec                  
[  5]  15.00-16.00  sec  9.32 MBytes  78.2 Mbits/sec                  
[  5]  16.00-17.01  sec  8.77 MBytes  73.0 Mbits/sec                  
[  5]  17.01-18.00  sec  9.09 MBytes  76.5 Mbits/sec                  
[  5]  18.00-19.00  sec  8.28 MBytes  69.6 Mbits/sec                  
[  5]  19.00-20.00  sec  8.98 MBytes  75.0 Mbits/sec                  
[  5]  20.00-21.00  sec  8.43 MBytes  70.9 Mbits/sec                  
[  5]  21.00-22.01  sec  8.68 MBytes  72.7 Mbits/sec                  
[  5]  22.01-23.00  sec  8.41 MBytes  70.6 Mbits/sec                  
[  5]  23.00-24.00  sec  8.47 MBytes  71.0 Mbits/sec                  
[  5]  24.00-25.01  sec  8.12 MBytes  68.1 Mbits/sec                  
[  5]  25.01-26.00  sec  8.04 MBytes  67.6 Mbits/sec                  
[  5]  26.00-27.00  sec  8.56 MBytes  71.7 Mbits/sec                  
[  5]  27.00-28.00  sec  7.60 MBytes  63.9 Mbits/sec                  
[  5]  28.00-29.01  sec  7.08 MBytes  58.9 Mbits/sec                  
[  5]  29.01-30.00  sec  8.68 MBytes  73.5 Mbits/sec                  
[  5]  30.00-31.00  sec  8.95 MBytes  75.1 Mbits/sec                  
[  5]  31.00-32.00  sec  8.61 MBytes  72.2 Mbits/sec                  
[  5]  32.00-33.00  sec  7.90 MBytes  66.1 Mbits/sec                  
[  5]  33.00-34.01  sec  8.98 MBytes  74.8 Mbits/sec                  
[  5]  34.01-35.00  sec  7.87 MBytes  66.3 Mbits/sec                  
[  5]  35.00-36.01  sec  8.22 MBytes  68.5 Mbits/sec                  
[  5]  36.01-37.01  sec  9.38 MBytes  78.7 Mbits/sec                  
[  5]  37.01-38.00  sec  9.17 MBytes  77.4 Mbits/sec                  
[  5]  38.00-39.00  sec  9.33 MBytes  78.4 Mbits/sec                  
[  5]  39.00-40.01  sec  8.94 MBytes  74.4 Mbits/sec                  
[  5]  40.01-40.09  sec   622 KBytes  67.1 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-40.09  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-40.09  sec   329 MBytes  68.8 Mbits/sec                  receiver


[root@clientTun vagrant]# iperf3 -c 10.10.0.1 -t 40 -i 5
Connecting to host 10.10.0.1, port 5201
[  4] local 10.10.0.2 port 50714 connected to 10.10.0.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  32.3 MBytes  54.2 Mbits/sec    0    522 KBytes       
[  4]   5.00-10.00  sec  40.4 MBytes  67.7 Mbits/sec    4    625 KBytes       
[  4]  10.00-15.00  sec  43.5 MBytes  73.0 Mbits/sec    2    557 KBytes       
[  4]  15.00-20.00  sec  43.5 MBytes  73.0 Mbits/sec    0    573 KBytes       
[  4]  20.00-25.01  sec  42.4 MBytes  71.1 Mbits/sec    1    541 KBytes       
[  4]  25.01-30.00  sec  40.0 MBytes  67.2 Mbits/sec    1    516 KBytes       
[  4]  30.00-35.00  sec  42.7 MBytes  71.7 Mbits/sec    0    566 KBytes       
[  4]  35.00-40.00  sec  44.5 MBytes  74.6 Mbits/sec    0    646 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-40.00  sec   329 MBytes  69.1 Mbits/sec    8             sender
[  4]   0.00-40.00  sec   329 MBytes  68.9 Mbits/sec                  receiver

iperf Done.
```
