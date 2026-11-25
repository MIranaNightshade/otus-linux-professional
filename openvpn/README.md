# OPENVPN 

## Цель
Создать домашнюю сетевую лабораторию. Научится настраивать VPN-сервер в Linux-based системах.

### Задачи
1. Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ

**1. TUN/TAP режимы VPN**

TUN работает на уровне L3, TAP работает на уровне L2. TUN передает IP пакеты, TAP ethrnet кадры. 

**Настройка сервера:**

Создаем ключ:

```
 openvpn --genkey secret /etc/openvpn/static.key
```

TAP:
vim /etc/openvpn/server.conf

```
dev tap 
ifconfig 10.10.10.1 255.255.255.0
route 10.10.10.0 255.255.255.0 
topology subnet 
secret /etc/openvpn/static.key 
comp-lzo 
status /var/log/openvpn-status.log 
log /var/log/openvpn.log  
verb 3
```


TUN:
vim /etc/openvpn/server.conf

```
dev tun 
ifconfig 10.10.10.1 255.255.255.0
route 10.10.10.0 255.255.255.0 
topology subnet 
secret /etc/openvpn/static.key 
comp-lzo 
status /var/log/openvpn-status.log 
log /var/log/openvpn.log  
verb 3
```

**Настройка клиента:**

Копируем созданный на сервере ключ.

TAP:
vim /etc/openvpn/client.conf

```
dev tap
remote 192.168.56.10
ifconfig 10.10.10.2 255.255.255.0
topology subnet
secret /etc/openvpn/static.key
comp-lzo
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
```

TUN:
vim /etc/openvpn/client.conf

```
dev tun
remote 192.168.56.10
ifconfig 10.10.10.2 255.255.255.0
topology subnet
secret /etc/openvpn/static.key
comp-lzo
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3
```

На сервере и клиенте создаем systemd юнит:
```bash
vim /etc/systemd/system/openvpn@.service
```

```
[Unit] 
Description=OpenVPN Tunneling Application On %I 
After=network.target 
[Service] 
Type=notify 
PrivateTmp=true 
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf 
[Install] 
WantedBy=multi-user.target

```

Замерим скорость в режиме tap:

```
root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 42320 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec   135 MBytes   227 Mbits/sec  222    502 KBytes       
[  5]   5.00-10.00  sec   131 MBytes   221 Mbits/sec  120    224 KBytes       
[  5]  10.00-15.00  sec   135 MBytes   226 Mbits/sec    0    490 KBytes       
[  5]  15.00-20.00  sec   134 MBytes   224 Mbits/sec   98    355 KBytes       
[  5]  20.00-25.00  sec   141 MBytes   236 Mbits/sec   49    293 KBytes       
[  5]  25.00-30.00  sec   134 MBytes   224 Mbits/sec  183    311 KBytes       
[  5]  30.00-35.00  sec   133 MBytes   222 Mbits/sec   40    276 KBytes       
[  5]  35.00-40.00  sec   136 MBytes   228 Mbits/sec    0    517 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec  1.05 GBytes   226 Mbits/sec  712             sender
[  5]   0.00-40.05  sec  1.05 GBytes   225 Mbits/sec                  receiver

iperf Done.
root@client:~#
```

Замерим скорость в режиме tun:

```
vagrant@client:~$ iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  5] local 10.10.10.2 port 40652 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-5.00   sec   136 MBytes   228 Mbits/sec  227    180 KBytes       
[  5]   5.00-10.00  sec   132 MBytes   222 Mbits/sec   36    258 KBytes       
[  5]  10.00-15.00  sec   137 MBytes   230 Mbits/sec    5    473 KBytes       
[  5]  15.00-20.00  sec   135 MBytes   226 Mbits/sec   70    247 KBytes       
[  5]  20.00-25.00  sec   132 MBytes   222 Mbits/sec   31    222 KBytes       
[  5]  25.00-30.00  sec   136 MBytes   228 Mbits/sec   13    371 KBytes       
[  5]  30.00-35.00  sec   138 MBytes   232 Mbits/sec    0    581 KBytes       
[  5]  35.00-40.00  sec   136 MBytes   229 Mbits/sec   98    414 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-40.00  sec  1.06 GBytes   227 Mbits/sec  480             sender
[  5]   0.00-40.14  sec  1.06 GBytes   226 Mbits/sec                  receiver

iperf Done.
```

***TUN режим быстрее из-за отсуствия накладных расходов***


**2. Поднять RAS**

