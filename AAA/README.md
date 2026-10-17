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

```
root@pam:~# ssh otus@192.168.57.10
otus@192.168.57.10's password: 
Permission denied, please try again.
otus@192.168.57.10's password: 
Permission denied, please try again.
otus@192.168.57.10's password: 

root@pam:~# ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-198-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

```

**⭐️ Предоставим пользователю otus доступ к Docker и право перезапускать Docker-сервис.**

отредактируем /etc/sudoers с помощью команды visudo
*visudo проверяет sudoers на ошибки перед записью изменений*
*ALL=(ALL:ALL) "на каких хостах разрешено"=("от имени какого пользователя":"от имени какой группы")*

```
otus   ALL=(ALL:All) /usr/bin/docker, /usr/bin/systemctl restart docker
```
