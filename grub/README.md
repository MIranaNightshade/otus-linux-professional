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

Минусы:
- Нужно знать пароль от root

#### 3. Переименуем volume group и отредактируем grub 
Переименуем VG с помощью команды:

```
vgrename vg-debian vg-otus
```

было:

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/vgs.png)

стало:

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/vgs-new.png)

На всякий случай сначала скопируем /boot/grub/grub.cfg к себе в домашнюю директорию перед внесением изменений.

```
sudo cp /boot/grub/grub.cfg ~
```
Отредактируем /boot/grub/grub.cfg поменяем все записи vg--dedian на vg--otus (теперь тут два -) и перезагрузим VM.

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/finfin.gif)


**P.S Еще кастомизировать загрузчик используя GRUB_THEME:**

/etc/default/grub

```
GRUB_DEFAULT=0
GRUB_TIMEOUT=60
#GRUB_TIMEOUT_STYLE=hidden
GRUB_DISTRIBUTOR=`( . /etc/os-release && echo ${NAME} )`
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX=""
GRUB_THEME="/boot/grub/themes/eyes.txt"
GRUB_HIDDEN_TIMEOUT_QUIET=false
```
/boot/grub/themes/eyes.txt

```
desktop-image: "linux-eyes.png"
desktop-image-scale-method: "padding"
desktop-image-h-align: "center"
desktop-image-v-align: "bottom"

+ label {
    id = "__timeout__"
    text = "Загрузка через %d сек."
    left = 0
    top = 50%
    width = 100%
    height = 20
    align = "center"
    color = "#ffffff"
    font = "Sans Regular 14"
}


+ boot_menu {
    left = 20%
    width = 60%
    top = 30%
    height = 60%
    item_color = "#ffffff"
    selected_item_color = "#ffff00"
    item_height = 32
    item_spacing = 4
}
```

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/grub/screen/castom_grub.gif)

