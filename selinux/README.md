# SELINUX
______________________________________________________________________________________________
## Задачи:
**1. Запустить Nginx на нестандартном порту 3-мя разными способами:**
- переключатели setsebool;
- добавление нестандартного порта в имеющийся тип;
- формирование и установка модуля SELinux.

**2. Обеспечить работоспособность приложения при включенном selinux.**
- развернуть стенд;
- выяснить причину неработоспособности механизма обновления зоны;
- предложить решение (или решения) для данной проблемы;
- выбрать одно из решений для реализации, предварительно обосновав выбор;
- реализовать выбранное решение и продемонстрировать его работоспособность.

_______________________________________________________________________________________________

### 1. Запустить Nginx на нестандартном порту 3-мя разными способами:

**1) Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателя setsebool**

*Проверим статус firewall (должен быть выключен), проверим корректна ли конфигурация nginx, и статус selinux 
(должен быть Enforcing, политики работают, логируются и ограничивают доступ)*

```
[root@selinux ~]#  systemctl status firewalld
○ firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:firewalld(1)
[root@selinux ~]#
[root@selinux ~]# 
[root@selinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@selinux ~]#
[root@selinux ~]#  getenforce
Enforcing
[root@selinux ~]#
```

*Проверим логи и перенаправим вывод в утилиту audit2why*

```
[root@selinux ~]# cat /var/log/audit/audit.log | grep nginx | audit2why
type=AVC msg=audit(1787647987.087:704): avc:  denied  { name_bind } for  pid=6553 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket 
permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly.
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
type=AVC msg=audit(1787748270.865:169): avc:  denied  { name_bind } for  pid=950 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly.
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
```
*Утилита audit2why показывает что можно поменять boolean параметр nis_enabled*
*Переключим nis_enabled в положение on*
```
[root@selinux ~]#  setsebool -P nis_enabled on
```
*Проверим статус переключателя:*

```
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> on
[root@selinux ~]#
```

*Перезапустим nginx*

```
[root@selinux ~]# 
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# 
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Thu 2026-09-03 10:23:17 UTC; 9s ago
    Process: 1392 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1393 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1395 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1396 (nginx)
      Tasks: 3 (limit: 11979)
     Memory: 4.2M
        CPU: 19ms
     CGroup: /system.slice/nginx.service
             ├─1396 "nginx: master process /usr/sbin/nginx"
             ├─1397 "nginx: worker process"
             └─1398 "nginx: worker process"

Sep 03 10:23:17 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Sep 03 10:23:17 selinux nginx[1393]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Sep 03 10:23:17 selinux nginx[1393]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Sep 03 10:23:17 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]#
```
**2) Разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип.**

*Поиск имеющегося типа, для http трафика:* semanage port -l | grep http

```
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[root@selinux ~]# 
```
*Добавим порт в тип http_port_t:* semanage port -a -t http_port_t -p tcp 4881
*и проверим результат*

-a (--add) - добавить 
-t http_port_t - тип http_port_t
-p tcp - протокол tcp

```
[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
[root@selinux ~]# 
[root@selinux ~]# semanage port -l | grep  http_port_t
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[root@selinux ~]#
```
Перезапустим nginx и проверим его работу: 

```
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# 
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Thu 2026-09-03 11:51:00 UTC; 5s ago
    Process: 1650 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1651 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1652 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1653 (nginx)
      Tasks: 3 (limit: 11979)
     Memory: 2.9M
        CPU: 17ms
     CGroup: /system.slice/nginx.service
             ├─1653 "nginx: master process /usr/sbin/nginx"
             ├─1654 "nginx: worker process"
             └─1655 "nginx: worker process"

Sep 03 11:51:00 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Sep 03 11:51:00 selinux nginx[1651]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Sep 03 11:51:00 selinux nginx[1651]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Sep 03 11:51:00 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# 
```
**3) Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux.**



