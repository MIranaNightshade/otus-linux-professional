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

