# Резервное копирование.
## Цель:
Настроить резервное копирование с помощью утилиты Borg.
### Задачи:

Настроить удаленный бекап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:

1. Директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования;
2. Репозиторий для резервных копий должен быть зашифрован ключом или паролем;
3. Имя бекапа должно содержать информацию о времени снятия бекапа;
4. Глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех.
5. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
6. Резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
7. Написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а;
8. Настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом.

----------------------------------------------------------------------------------------------------------------------
На сервере и клиенте устанавливаем borg:

```
apt install borgbackup
```

На сервере backup:
создаем пользователя borg, монтируем диск в /var/backup и назначаем на него права пользователя borg:

```
adduser --disabled-password borg
mkdir /var/backup && mount /dev/sdb /var/backup
chown borg:borg /var/backup/
```

На сервере client генерируем пару ключей ssh:

```
ssh-kegen
```

Отправляем на сервер backup наш pub key:

```
mkdir ~borg/.ssh
echo 'command="/usr/local/bin/borg serve" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCdPv0XFpjRpaNcCRDqCvfRGoqimWmvuj4oQglgS09cAUuVesby50vu9s43GAnlYZxFabNxQB9usgTojoXx01NjbdyqUWecDxuwKizC940nysmiopGLbyZ4RcZNchbKm3pDFiKTQPU7TElEFoNKIbFQuIMI/xQ7DRo1Hr4TUumUN3yH6LGDuEIidhtlkpRgnBXEGDJfgOO8ra8rolYFKQIxwLgUumeT1CRdDQH3dg1fSNVkfyURt6sjlDYDTcww402knhegPyezK5JXaiCIUD6SzkqcpYSUZkILdBrAPMsOecoY+MN1jsp2v3BfwkuBLXBxZWFuIxonw9jIv7VGzWRtnocvXBbZBB3e9Ww857CfU/AVOqZhhrGl8YLtFvDquw8cdwEE+DV7WdzEkpjWq3iHg7hU6KYCoa/A76vDGz1IPoGDliHWRMHwVCb4/ryK83odl4fjMOcAMd8uUvPWqcjnzynm9tEJALHnTCTrShFE6571Az4uYaY6qWZlIyYskvE=' > ~borg/.ssh/authorized_keys
chown -R borg:borg ~borg/.ssh
```

На сервере backup создадим симлинк:

```
ln -s /usr/bin/borg /usr/local/bin/borg
```

**C client server:**

Инициализируем репозиторий borg на backup сервере с client сервера:

```
borg init --encryption=repokey borg@192.168.56.160:/var/backup/
```

Запускаем проверку создания бэкапа: 

```
borg create --stats --list borg@192.168.56.160:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc
```

Проверим результат:

```
root@client:~# borg list borg@192.168.56.160:/var/backup/
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
etc-2026-10-30_05:02:47              Fri, 2026-10-30 05:03:07 [08c4e4da853676bc92235484a05f6e51f79c8a2a3980ac53fea48ea0caf05918]
root@client:~# 
```

Проверим список файлов:

```
root@client:~# borg list borg@192.168.56.160:/var/backup/::etc-2026-10-30_05:02:47
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
drwxr-xr-x root   root          0 Fri, 2026-10-30 04:47:46 etc
lrwxrwxrwx root   root         19 Wed, 2024-09-11 14:22:06 etc/mtab -> ../proc/self/mounts
lrwxrwxrwx root   root         21 Tue, 2024-09-10 11:18:05 etc/os-release -> ../usr/lib/os-release
lrwxrwxrwx root   root         39 Wed, 2024-09-11 14:22:05 etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
lrwxrwxrwx root   root         13 Tue, 2023-12-05 05:15:51 etc/rmt -> /usr/sbin/rmt
lrwxrwxrwx root   root         23 Wed, 2024-09-11 14:22:40 etc/vtrgb -> /etc/alternatives/vtrgb
drwxr-xr-x root   root          0 Wed, 2024-09-11 14:24:23 etc/ModemManager
drwxr-xr-x root   root          0 Wed, 2023-12-20 05:35:16 etc/ModemManager/connection.d
drwxr-xr-x root   root          0 Wed, 2023-12-20 05:35:16 etc/ModemManager/fcc-unlock.d
drwxr-xr-x root   root          0 Thu, 2025-10-23 22:27:15 etc/PackageKit
-rw-r--r-- root   root        706 Thu, 2022-02-17 13:13:28 etc/PackageKit/PackageKit.conf
-rw-r--r-- root   root       1718 Mon, 2022-03-14 19:11:19 etc/PackageKit/Vendor.conf
drwxr-xr-x root   root          0 Wed, 2024-09-11 14:22:12 etc/X11
drwxr-xr-x root   root          0 Thu, 2025-10-23 22:31:03 etc/X11/Xsession.d
-rw-r--r-- root   root       1062 Mon, 2021-06-28 12:15:25 etc/X11/Xsession.d/20dbus_xdg-runtime
-rw-r--r-- root   root        880 Sat, 2021-12-18 21:45:15 etc/X11/Xsession.d/90gpg-agent
-rw-r--r-- root   root       2127 Thu, 2025-10-23 22:31:03 etc/X11/Xsession.d/98vboxadd-xclient
drwxr-xr-x root   root          0 Wed, 2021-08-18 09:26:23 etc/X11/xkb
drwxr-xr-x root   root          0 Thu, 2025-10-23 22:32:52 etc/alternatives
lrwxrwxrwx root   root         23 Wed, 2024-09-11 14:24:12 etc/alternatives/arptables -> /usr/sbin/arptables-nft
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:12 etc/alternatives/arptables-restore -> /usr/sbin/arptables-nft-restore
lrwxrwxrwx root   root         28 Wed, 2024-09-11 14:24:12 etc/alternatives/arptables-save -> /usr/sbin/arptables-nft-save
lrwxrwxrwx root   root         13 Wed, 2024-09-11 14:22:17 etc/alternatives/awk -> /usr/bin/gawk
lrwxrwxrwx root   root         29 Wed, 2024-09-11 14:22:56 etc/alternatives/awk.1.gz -> /usr/share/man/man1/gawk.1.gz
lrwxrwxrwx root   root         38 Wed, 2024-09-11 14:22:57 etc/alternatives/builtins.7.gz -> /usr/share/man/man7/bash-builtins.7.gz
lrwxrwxrwx root   root         22 Wed, 2024-09-11 14:24:12 etc/alternatives/ebtables -> /usr/sbin/ebtables-nft
lrwxrwxrwx root   root         30 Wed, 2024-09-11 14:24:12 etc/alternatives/ebtables-restore -> /usr/sbin/ebtables-nft-restore
lrwxrwxrwx root   root         27 Wed, 2024-09-11 14:24:12 etc/alternatives/ebtables-save -> /usr/sbin/ebtables-nft-save
lrwxrwxrwx root   root          9 Wed, 2024-09-11 14:24:10 etc/alternatives/editor -> /bin/nano
lrwxrwxrwx root   root         29 Wed, 2024-09-11 14:24:10 etc/alternatives/editor.1.gz -> /usr/share/man/man1/nano.1.gz
lrwxrwxrwx root   root         18 Wed, 2024-09-11 14:24:27 etc/alternatives/ex -> /usr/bin/vim.basic
lrwxrwxrwx root   root         28 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.1.gz -> /usr/share/man/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.da.1.gz -> /usr/share/man/da/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.de.1.gz -> /usr/share/man/de/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.fr.1.gz -> /usr/share/man/fr/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.it.1.gz -> /usr/share/man/it/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.ja.1.gz -> /usr/share/man/ja/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.pl.1.gz -> /usr/share/man/pl/man1/vim.1.gz
lrwxrwxrwx root   root         31 Wed, 2024-09-11 14:24:27 etc/alternatives/ex.ru.1.gz -> /usr/share/man/ru/man1/vim.1.gz
lrwxrwxrwx root   root         14 Wed, 2024-09-11 14:24:10 etc/alternatives/ftp -> /usr/bin/tnftp
lrwxrwxrwx root   root         30 Wed, 2024-09-11 14:24:10 etc/alternatives/ftp.1.gz -> /usr/share/man/man1/tnftp.1.gz
...
```
Достанем файл из бекапа:

```
root@client:~# borg extract borg@192.168.56.160:/var/backup/::etc-2026-10-30_05:02:47 etc/hostname
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: 
root@client:~# 
root@client:~# 
root@client:~# 
root@client:~# cat etc/hostname 
client
root@client:~# 
```

**Автоматизируем создание бэкапов с помощью systemd (service+timer)**

*Какие бекапы храним:*
*--keep-within 3m* Все бекапы за последние 3 месяца
*--keep-monthly 12* По одному бекапу за последние 12 месяцев (кроме первых трех месяцев правило для которых в первом правиле)


```
root@client:~# systemctl cat borg-backup.service
# /etc/systemd/system/borg-backup.service
[Unit]
Description=Borg Backup

[Service]
Type=oneshot

# Парольная фраза
Environment="BORG_PASSPHRASE=Otus1234"
# Репозиторий
Environment=REPO=borg@192.168.56.160:/var/backup/
# Что бэкапим
Environment=BACKUP_TARGET=/etc

# Создание бэкапа
ExecStart=/bin/borg create \
    --stats                \
    ${REPO}::etc-{now:%%Y-%%m-%%d_%%H:%%M:%%S} ${BACKUP_TARGET}

# Проверка бэкапа
ExecStart=/bin/borg check ${REPO}

# Очистка старых бэкапов
ExecStart=/bin/borg prune \
    --keep-within 3m \
    --keep-monthly 12 \
    ${REPO}
```

```
root@client:~# systemctl cat borg-backup.timer
# /etc/systemd/system/borg-backup.timer
[Unit]
Description=Borg Backup

[Timer]
OnBootSec=15sec
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```

Проверим логи:

```
root@client:~# journalctl -u borg-backup -f
Oct 30 06:25:40 client systemd[1]: Starting Borg Backup...
Oct 30 06:25:41 client borg[1828]: ------------------------------------------------------------------------------
Oct 30 06:25:41 client borg[1828]: Repository: ssh://borg@192.168.56.160/var/backup
Oct 30 06:25:41 client borg[1828]: Archive name: etc-2026-10-30_06:25:40
Oct 30 06:25:41 client borg[1828]: Archive fingerprint: b49bf9ccef056e6d2dba125138ed7946bcd4666f995f24be9285501a85a29964
Oct 30 06:25:41 client borg[1828]: Time (start): Fri, 2026-10-30 06:25:41
Oct 30 06:25:41 client borg[1828]: Time (end):   Fri, 2026-10-30 06:25:41
Oct 30 06:25:41 client borg[1828]: Duration: 0.12 seconds
Oct 30 06:25:41 client borg[1828]: Number of files: 805
Oct 30 06:25:41 client borg[1828]: Utilization of max. archive size: 0%
Oct 30 06:25:41 client borg[1828]: ------------------------------------------------------------------------------
Oct 30 06:25:41 client borg[1828]:                        Original size      Compressed size    Deduplicated size
Oct 30 06:25:41 client borg[1828]: This archive:                2.29 MB              1.03 MB              1.25 kB
Oct 30 06:25:41 client borg[1828]: All archives:                4.57 MB              2.06 MB              1.15 MB
Oct 30 06:25:41 client borg[1828]:                        Unique chunks         Total chunks
Oct 30 06:25:41 client borg[1828]: Chunk index:                     758                 1584
Oct 30 06:25:41 client borg[1828]: ------------------------------------------------------------------------------
Oct 30 06:25:42 client systemd[1]: borg-backup.service: Deactivated successfully.
Oct 30 06:25:42 client systemd[1]: Finished Borg Backup.
```
