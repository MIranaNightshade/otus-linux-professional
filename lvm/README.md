# Работа с LVM.
### Задачи:
1. [Уменьшить том под / до 8G.](#title1)
2. [Выделить том под /home.](#title2)
3. [Выделить том под /var - сделать в mirror.](#title3)
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

1) Подготовим том для временного размещения /:

```
root@debianv2:~# pvcreate /dev/sda
  Physical volume "/dev/sda" successfully created.
root@debianv2:~# vgcreate vg_root /dev/sda
  Volume group "vg_root" successfully created
root@debianv2:~# lvcreate -n lv_root -l +100%FREE vg_root
  Logical volume "lv_root" created.
root@debianv2:~#
```

2) Создадим на созданном lv_root файловую систему.

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

3) Скопируем данные с / на новый созданный /vg_root/lv_root

*rsync -aAXv --progress --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found", "/boot"} / /mnt/*

*--archive, -a  -rlptgoD сохраняем рекурсивно, с сохранением симлинков, разрешений, меток времени, сохраняем принадлежность к группам, сохраняем данные о владельцах, сохраняем данные о файлах устройств* 
*-A сохраняем даннеы об ACL*
*--xattrs, -X сохраняем расширенные аттрибуты* 
*-v --verbose подробный вывод команды*

*Исключаем (создаем каталоги, но исключаем содержимое):
"/dev/*","/proc/*","/sys/*","/run/*" -- это виртуальные каталоги, данные в них лежат в оперативной памяти или формируются при запросе ядром.
"/mnt/*" - чтобы избежать петли в копировании 
"/tmp/*" - тут временные файлы, они нам не нужны
 "/boot" - лежит на другом логическом разделе*
