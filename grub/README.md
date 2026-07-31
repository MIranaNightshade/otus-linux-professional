# Загрузка системы. работа с GRUB.

## Задание
1. Включить отображение меню Grub.
2. Попасть в систему без пароля несколькими способами.
3. Установить систему с LVM, после чего переименовать VG.

#### 1. Включим отображение меню GRUB.
- Отредактируем /etc/default/grub
- Закомментируем GRUB_TIMEOUT_STYLE=hidden и выставим timeout:

```
GRUB_DEFAULT=0
GRUB_TIMEOUT=60
# GRUB_TIMEOUT_STYLE=hidden
GRUB_DISTRIBUTOR=`( . /etc/os-release && echo ${NAME} )`
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX=""
```
- Выполним update-grub чтобы обновить конфигурацию GRUB.

Проверим результат:

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/menu.gif)

#### 2. Выполним вход в систему без пароля.

**Способ 1.** В начальном меню GRUB нажать e чтобы перейти в edit. Найдем строку начинающуюся с linux и допишем в конце init=/bin/bash, затем ctrl+x

Минусы:
- / при этом способе монтируется в режиме ro:

![ro](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/ro.png)

Можно перемонтировать командой:

```
mount -o remount,rw /
```
![rw](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/rw.png)

**Способ 2.** В меня GRUB выбираем advanced options и далее пункт с recovery mode. 

![recovery](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/recovery.png)

