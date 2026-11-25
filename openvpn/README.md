# OPENVPN 

## Цель
Создать домашнюю сетевую лабораторию. Научится настраивать VPN-сервер в Linux-based системах.

### Задачи
1. Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ

1. TUN/TAP режимы VPN

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
vim /etc/systemd/system/openvpn@.service

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
