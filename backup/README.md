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
apt install borg-backup
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

