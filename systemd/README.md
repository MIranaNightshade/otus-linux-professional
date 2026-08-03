# systemd создание unit-файла.
## Цели:
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

### Выполнение: 
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
#### 1. Напишем service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова.
**Создадим конфигурационный файл для нашего сервиса в /etc/default:**

```
root@debian-lvm:~# cat << EOF > /etc/default/watchlog
> # Configuration file for my watchlog service      
> # Place it to /etc/default
>
> # File and word in that file that we will be monit
> WORD="ALERT"
> LOG=/var/log/watchlog.log
> EOF
root@debian-lvm:~# 
```
Создадим файл лога в /var/log/watchlog.log с несколькими строчками одна из которых будет содержать ALERT на которое будет реагировать скрипт:

```
root@debian-lvm:~# cat << EOF > /var/log/watchlog.log
> INFO service started
> INFO everything is going according to plan
> ALARM It's time to "AAAAAAAAA" and running around a table. 
> EOF
root@debian-lvm:~# 
```

**Создадим скрипт который будет выполнять сервис:**

```
root@debian-lvm:~# cat << EOF > /opt/watchlog.sh
> #!/bin/bash
>
> WORD=$1
> LOG=$2
> DATE=`date`
>
> if grep $WORD $LOG &> /dev/null      
> then
> logger "$DATE: I found word, Master!"
> else
> exit 0
> fi
> EOF
```
Сделаем скрипт исполняемым: 

```
chmod +x /opt/watchlog.sh
```

**Создадим юнит для сервиса:**

- type=oneshot запускается один раз, выполняет скрипт и отключается.
- EnvironmentFile=/etc/default/watchlog - покажем где искать конфигурационный файл.
- ExecStart=/opt/watchlog.sh $WORD $LOG запуск скрипта с переменными взятыми из конфигурационного файла. 

```
root@debian-lvm:~# cat << EOF > /etc/systemd/system/watchlog.service
> [Unit]
> Description=My watchlog service
>
> [Service]
> Type=oneshot
> EnvironmentFile=/etc/default/watchlog
> ExecStart=/opt/watchlog.sh $WORD $LOG
> EOF
root@debian-lvm:~# 
```
