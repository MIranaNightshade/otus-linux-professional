# SELINUX
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
