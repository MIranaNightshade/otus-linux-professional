# NFS

## Цель:
 - научиться самостоятельно разворачивать сервис NFS и подключать к нему клиентов;

### Задачи:
1. Запустить 2 виртуальных машины (сервер NFS и клиента);
2. На сервере NFS подготовить и экспортировать директорию. В экспортированной директории создать поддиректорию с именем upload с правами на запись в неё;
3. Настроить автоматическое монтирование экспортированной директории при старте виртуальной машины (systemd, autofs или fstab — любым способом).
Монтирование и работа NFS на клиенте огранизовать с использованием NFSv3.

#### 1. Запустим две вирутальные машины клиента и сервера NFS:

- debian-nfss - 192.168.88.100 - **сервер**
  ```
  apt install nfs-kernel-server
  ``` 
- debian-nfsc - 192.168.88.111 - **клиент**
  ```
  sudo apt install nfs-common
  ```


#### 2. На сервере NFS подготовим и экспортируем директорию, а так де создадим поддиректорию upload c правами на запись: 

```
mkdir -p /srv/share/upload
chown -R nobody:nogroup /srv/share
chmod 0777 /srv/share/upload
echo "/srv/share 192.168.88.111/32(rw,sync,root_squash)" | tee -a /etc/exports
```

*Перечитаем и применим конфигурацию к nfs серверу*

```
exportfs -r
```

*Просмотрим текущие экспорты:*

```
exportfs -s
```
*Скрин настройки сервера:*

![server](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/nfs/screen/nfs-server-conf.png)

#### 3. Настроим монтирование nfs - директории на клиенте. 

*Проверим возможность монтирование nfs файловой системы с клиента:*

![client](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/nfs/screen/client-server-check.png)



