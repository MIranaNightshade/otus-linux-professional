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
>  #!/bin/bash
>
>  WORD=\$1
>  LOG=\$2
>  DATE=\`date\`
>
>  if grep \$WORD \$LOG &> /dev/null
>  then
>  logger "\$DATE: I found word, Master!"
>  else
>  exit 0
>  fi
> EOF
root@debian-lvm:~# 
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
> ExecStart=/opt/watchlog.sh \$WORD \$LOG
> EOF
root@debian-lvm:~# 
```

**Создадим таймер:**
- OnActiveSec=1 выполнить первый раз через 1 секунду после запуска таймера, если не выставить то нужно будет первый раз запустить сервис руками иначе не будет точки отсчета для OnUnitActiveSec
-  OnUnitActiveSec=30 - запускаем юнит каждые 30 сек
-  Unit=watchlog.service - какой юнит запускаем каждые 30 сек
-  WantedBy=multi-user.target - таймер запускается на уровне multi-user.target

```
root@debian-lvm:~# cat << EOF > /etc/systemd/system/watchlog.timer
> [Unit]
> Description=Run watchlog script every 30 second
>
> [Timer]
> # Run every 30 second
> OnActiveSec=1
> OnUnitActiveSec=30
> Unit=watchlog.service
>
> [Install]
> WantedBy=multi-user.target
> EOF
root@debian-lvm:~# 
```
Проверка: 

![word](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/systemd/screen/found_word.png)

#### 2. Установим spawn-fcgi и создадим unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).

Устанавливаем необходимые пакеты:

```
apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y
```
Создадим файл конфигурации для spawn-fcgi:
SOCKET - *сокет через который веб сервер (например nginx) будет обращаться к процессам php-cgi*
-u www-data -g www-data - *запускаться от пользователя www-data группы www-data*
-S *создать UNIX сокет от root, если этого не сделать то spawn-fcgi попытается создать сокет в /var/run от пользователя www-data где у него нет прав.*
-M 0600 - *права на rw у пользователя www-data, остальные без прав*
-С 32 - *создать 32 процесса php-cgi*
-F 1 - *создать 1 (fork) управляющий процесс самого spawn-fcgi*

```
root@debian-lvm:~# mkdir /etc/spawn-fcgi && touch /etc/spawn-fcgi/fcgi.conf
root@debian-lvm:~# 
root@debian-lvm:~# cat > /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
root@debian-lvm:~# 
```
**Создадим unit-файл для самого spawn-fcgi:**

```

```
