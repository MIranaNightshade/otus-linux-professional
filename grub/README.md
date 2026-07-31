# Загрузка системы. работа с GRUB.

## Задание
1. Включить отображение меню Grub.
2. Попасть в систему без пароля несколькими способами.
3. Установить систему с LVM, после чего переименовать VG.

#### 1. Включим отображение меню GRUB.
Отредактируем /etc/default/grub
Закомментируем GRUB_TIMEOUT_STYLE=hidden и выставим timeout:

```
GRUB_DEFAULT=0
GRUB_TIMEOUT=60
# GRUB_TIMEOUT_STYLE=hidden
GRUB_DISTRIBUTOR=`( . /etc/os-release && echo ${NAME} )`
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX=""
```
   


