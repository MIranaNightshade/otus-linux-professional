# Работа с LVM.
### Задачи:
1. [Уменьшить том под / до 8G.](#title1)
2. [Выделить том под /var - сделать в mirror.](#title2)
3. [Выделить том под /home.](#title2)
4. [/home - сделать том для снапшотов.](#title4)
5. [Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).](#title5)
6. [Работа со снапшотами:](#title6)
    - *сгенерить файлы в /home/;*
    - *снять снапшот;*
    - *удалить часть файлов;*
    - *восстановиться со снапшота.*
  
Вот так выглядит распределение по дискам:

![lsblk](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/lsblk.png)

#### 1. <a id="title1">Уменьшить том под / до 8G.</a>

**1) Подготовим том для временного размещения /:**

```
root@debianv2:~# pvcreate /dev/sda
  Physical volume "/dev/sda" successfully created.
root@debianv2:~# vgcreate vg_root /dev/sda
  Volume group "vg_root" successfully created
root@debianv2:~# lvcreate -n lv_root -l +100%FREE vg_root
  Logical volume "lv_root" created.
root@debianv2:~#
```

**2) Создадим на созданном lv_root файловую систему.**

```
root@debianv2:~# mkfs.ext4  /dev/vg_root/lv_root
mke2fs 1.47.2 (1-Jan-2025)
Creating filesystem with 2620416 4k blocks and 655360 inodes
Filesystem UUID: 4785a5db-ced6-4e36-a8f9-fbe63265909d
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@debianv2:~#
```

**3) Смонтируем lv_root**

```
mount /dev/vg_root/lv_root /mnt
```

**3) Скопируем данные с / на новый созданный /vg_root/lv_root**

*rsync -aAXv --progress --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/boot"} / /mnt/*

*--archive, -a  -rlptgoD сохраняем рекурсивно, с сохранением симлинков, разрешений, меток времени, сохраняем принадлежность к группам, сохраняем данные о владельцах, сохраняем данные о файлах устройств* 
*-A сохраняем даннеы об ACL*
*--xattrs, -X сохраняем расширенные аттрибуты* 
*-v --verbose подробный вывод команды*

*Исключаем (создаем каталоги, но исключаем содержимое):
"/dev/*","/proc/*","/sys/*","/run/*" -- это виртуальные каталоги, данные в них лежат в оперативной памяти или формируются при запросе ядром.
"/mnt/*" - чтобы избежать петли в копировании 
"/tmp/*" - тут временные файлы, они нам не нужны
 "/boot" - лежит на другом логическом разделе*

```
root@debianv2:/# ls -la /mnt/
итого 76
drwxr-xr-x 19 root root 4096 июл 18 10:31 .
drwxr-xr-x 19 root root 4096 июл 18 10:31 ..
lrwxrwxrwx  1 root root    7 июл 14 13:11 bin -> usr/bin
drwxr-xr-x  4 root root 4096 июл 15 09:44 boot
drwxr-xr-x 20 root root 4096 июл 18 09:39 dev
drwxr-xr-x 73 root root 4096 июл 15 09:44 etc
drwxr-xr-x  3 root root 4096 июл 14 13:16 home
lrwxrwxrwx  1 root root   35 июл 14 13:12 initrd.img -> boot/initrd.img-6.12.95+deb13-amd64
lrwxrwxrwx  1 root root   35 июл 14 13:11 initrd.img.old -> boot/initrd.img-6.12.86+deb13-amd64
lrwxrwxrwx  1 root root    7 июл 14 13:11 lib -> usr/lib
lrwxrwxrwx  1 root root    9 июл 14 13:11 lib64 -> usr/lib64
drwx------  2 root root 4096 июл 14 13:10 lost+found
drwxr-xr-x  3 root root 4096 июл 14 13:10 media
drwxr-xr-x 19 root root 4096 июл 18 10:31 mnt
drwxrwxr-x  2 root root 4096 июл 18 10:31 new_root
drwxr-xr-x  2 root root 4096 июл 14 13:11 opt
dr-xr-xr-x  2 root root 4096 июл 18 09:06 proc
drwx------  3 root root 4096 июл 18 09:50 root
drwxr-xr-x  2 root root 4096 июл 18 09:27 run
lrwxrwxrwx  1 root root    8 июл 14 13:11 sbin -> usr/sbin
drwxr-xr-x  2 root root 4096 июл 14 13:11 srv
dr-xr-xr-x  2 root root 4096 июл 18 10:15 sys
drwxrwxrwt  2 root root 4096 июл 18 09:16 tmp
drwxr-xr-x 12 root root 4096 июл 14 13:11 usr
drwxr-xr-x 11 root root 4096 июл 14 13:16 var
lrwxrwxrwx  1 root root   32 июл 14 13:12 vmlinuz -> boot/vmlinuz-6.12.95+deb13-amd64
lrwxrwxrwx  1 root root   32 июл 14 13:11 vmlinuz.old -> boot/vmlinuz-6.12.86+deb13-amd64
```
**4) Монтируем /proc/ /sys/ /dev/ /run/ /boot/ в новый корень**

```
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
```

**5) Сымитируем текущий root, сделаем в него chroot и обновим grub:**

```
chroot /mnt/
grub-mkconfig -o /boot/grub/grub.cfg
```
**6) Обновим образ initrd.**

```
update-initramfs -u
```

**7) Проверим где находится / после презагрузки:**

![lsblk](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/lsblk_after_reboot.png)

**8) Удалим старый vg-debian, создадим новый vg-debian размером 8G, создадим на нем файловую систему ext4, смотнтируем новый lv в /mnt, скопируем на него /
      и сконфигурируем grub.**

```
lvremove /dev/vg-debian/lv-debian
lvcreate -n vg-debian/lv-debian -L 8G /dev/vg-debian
mkfs.ext4 /dev/vg-debian/lv-debian
mount /dev/vg-debian/lv-debian /mnt
rsync -aAXv --progress --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/boot"} / /mnt/
```
grub

```
root@debianv2:~# for i in /proc/ /sys/ /dev/ /run/ /boot/;  do mount --bind $i /mnt/$i; done
root@debianv2:~# chroot /mnt/
root@debianv2:/# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.12.95+deb13-amd64
Found initrd image: /boot/initrd.img-6.12.95+deb13-amd64
Found linux image: /boot/vmlinuz-6.12.86+deb13-amd64
Found initrd image: /boot/initrd.img-6.12.86+deb13-amd64
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
root@debianv2:/# update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.12.95+deb13-amd64
root@debianv2:/#
```

#### 2. <a id="title2">Выделим том под /var в mirror.</a>

**1) Создадим новый pv vg и lv(mirror) для /var. Создадим на новом lv_var файловую систему ext4 и смонтируем все в /mnt/**

![mirror](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/var_mirror.png)

**2) Скопируем содержимае /var на новый vg_var (который смонтирован в mnt) и смонтируем lv_var в /var**

```
cp -aR /var/* /mnt/
umount /mnt
mount /dev/vg_var/lv_var /var
```
Отредактируем /etc/fstab чтобы lv_var сразу монтировался при загрузке 

![fstab](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/fstab_var.png)

Проверим что получилось после перезагрузки:

![fstab](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/flsblk_var.png)

#### 3. <a id="title3">Выделим том под /home и смонтируем вего в fstab.</a>

**Выделим том под /home**

![home](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/home.png)

#### 4. <a id="title4">Смонтируем /home в /etc/fstab.</a>

```
UUID=5d751fae-c721-478e-88b2-15f33e631af0  /home  ext4  defaults,nosuid,noexec  0  2
```
 - nosuid Запретим действие suid и sgid битов.
 - noexec запретим исполнение бинарных файлов
![home](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/home_fstab1.png)

#### 5. <a id="title5">Работа со снапшотами.</a>

1) Создадим снапшот /home

```
root@debianv2:~# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home_lv   vg-debian -wi-ao----   3,00g
  lv-debian vg-debian -wi-ao----   8,00g
  lv_var    vg_var    rwi-aor--- 900,00m                                    100,00
root@debianv2:~# lvcreate -n home_snap -l 200M -s /dev/vg-debian/home_lv
  Invalid argument for --extents: 200M
  Error during parsing of command line.
root@debianv2:~# lvcreate -n home_snap -L 200M -s /dev/vg-debian/home_lv
  Logical volume "home_snap" created.
root@debianv2:~#
```
2) Удалим файлы из /home и восстановим из снапшот

![rmhome](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/remove_home.png)

3) Восстановим файлы из снапшота

![recovery](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/lvm/screen/recovery_home.png)
