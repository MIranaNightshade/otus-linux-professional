# Цель
Научиться создавать пользователей и добавлять им ограничения

## Описание домашнего

Ограничить доступ к системе для всех пользователей, кроме группы администраторов, в выходные дни (суббота и воскресенье), за исключением праздничных дней.

### ⭐️ Задание со звездочкой
Предоставить определённому пользователю доступ к Docker и право перезапускать Docker-сервис.

----------------------------------------------------------------------------------------------------------------------------------------------------
Создадим vm с помощью Vagrantfile:

```
MACHINES = {
  :"pam" => {
              :box_name => "ubuntu/jammy64",
              :cpus => 2,
              :memory => 1024,
              :ip => "192.168.57.10",
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.network "private_network", ip: boxconfig[:ip]
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s

      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      box.vm.provision "shell", inline: <<-SHELL
          echo 'PasswordAuthentication yes' > /etc/ssh/sshd_config.d/00-enable-password.conf
          systemctl restart sshd.service
          SHELL
    end
  end
end
```

Создадим пользователей otusadm и otus и создадим им пароль Otus2026!, так же создадим группу admin.


```
root@pam:~# sudo useradd otusadm && sudo useradd otus
root@pam:~# 
root@pam:~# passwd otusadm
New password: 
Retype new password: 
passwd: password updated successfully
root@pam:~# 
root@pam:~# 
root@pam:~# passwd otus
New password: 
Retype new password: 
passwd: password updated successfully
root@pam:~# 
root@pam:~# groupadd -f admin
```
Добавим пользователей vagrant,root и otusadm в группу admin:

```
root@pam:~# usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
```
Проверим, что пользователи root, vagrant и otusadm есть в группе admin:

```
vagrant@pam:~$  cat /etc/group | grep admin
admin:x:117:otusadm,root,vagrant
```

Создадим файл-скрипт /usr/local/bin/login.sh

```
#!/bin/bash
#Первое условие: если день недели суббота или воскресенье
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 #Второе условие: входит ли пользователь в группу admin
 if getent group admin | grep -qw "$PAM_USER"; then
        #Если пользователь входит в группу admin, то он может подключиться
        exit 0
      else
        #Иначе ошибка (не сможет подключиться)
        exit 1
    fi
  #Если день не выходной, то подключиться может любой пользователь
  else
    exit 0
fi
```
Сделаем скрипт исполняемым:

```
chmod +x /usr/local/bin/login.sh
```

Добавим строчку со скриптом в vim /etc/pam.d/sshd 

```
auth required pam_exec.so debug /usr/local/bin/login.sh
```

Проверим:

otusadm залогинился успешно:


```
root@pam:~# journalctl -n 0  -f
Oct 17 11:53:55 pam sshd[2468]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Oct 17 11:53:55 pam sshd[2466]: Accepted password for otusadm from 192.168.57.1 port 43554 ssh2
Oct 17 11:53:55 pam sshd[2466]: pam_unix(sshd:session): session opened for user otusadm(uid=1002) by (uid=0)
Oct 17 11:53:55 pam systemd[1]: Created slice User Slice of UID 1002.
Oct 17 11:53:55 pam systemd[1]: Starting User Runtime Directory /run/user/1002...
Oct 17 11:53:56 pam systemd-logind[642]: New session 21 of user otusadm.
Oct 17 11:53:56 pam systemd[1]: Finished User Runtime Directory /run/user/1002.
Oct 17 11:53:56 pam systemd[1]: Starting User Manager for UID 1002...
Oct 17 11:53:56 pam systemd[2473]: pam_unix(systemd-user:session): session opened for user otusadm(uid=1002) by (uid=0)
Oct 17 11:53:56 pam systemd[2473]: Queued start job for default target Main User Target.
Oct 17 11:53:56 pam systemd[2473]: Created slice User Application Slice.
Oct 17 11:53:56 pam systemd[2473]: Reached target Paths.
Oct 17 11:53:56 pam systemd[2473]: Reached target Timers.
Oct 17 11:53:56 pam systemd[2473]: Starting D-Bus User Message Bus Socket...
Oct 17 11:53:56 pam systemd[2473]: Listening on GnuPG network certificate management daemon.
Oct 17 11:53:56 pam systemd[2473]: Listening on GnuPG cryptographic agent and passphrase cache (access for web browsers).
Oct 17 11:53:56 pam systemd[2473]: Listening on GnuPG cryptographic agent and passphrase cache (restricted).
Oct 17 11:53:56 pam systemd[2473]: Listening on GnuPG cryptographic agent (ssh-agent emulation).
Oct 17 11:53:56 pam systemd[2473]: Listening on GnuPG cryptographic agent and passphrase cache.
Oct 17 11:53:56 pam systemd[2473]: Listening on debconf communication socket.
Oct 17 11:53:56 pam systemd[2473]: Listening on REST API socket for snapd user session agent.
Oct 17 11:53:56 pam systemd[2473]: Listening on D-Bus User Message Bus Socket.
Oct 17 11:53:56 pam systemd[2473]: Reached target Sockets.
Oct 17 11:53:56 pam systemd[2473]: Reached target Basic System.
Oct 17 11:53:56 pam systemd[2473]: Reached target Main User Target.
Oct 17 11:53:56 pam systemd[2473]: Startup finished in 38ms.
Oct 17 11:53:56 pam systemd[1]: Started User Manager for UID 1002.
Oct 17 11:53:56 pam systemd[1]: Started Session 21 of User otusadm.
```

otus ожидаемо залогиниться не смог:

```
root@pam:~# journalctl -n 0  -f
Oct 17 11:56:11 pam sshd[2561]: pam_exec(sshd:auth): Calling /usr/local/bin/login.sh ...
Oct 17 11:56:11 pam sshd[2559]: pam_exec(sshd:auth): /usr/local/bin/login.sh failed: exit code 1
Oct 17 11:56:13 pam sshd[2559]: Failed password for otus from 192.168.57.1 port 58332 ssh2
```
