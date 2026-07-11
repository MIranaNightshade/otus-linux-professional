# Обновление ядра системы
### Цель домашнего задания
Научиться обновлять ядро в ОС Linux.
### Описание домашнего задания
1) Запустим ВМ c Debian.
2) Обновим ядро ОС на новую версию.

**Текущая версия ядра:**

   ![old](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/linux%20kernel/screenshots/old_kernel.png)


Найдем новую версию ядра в репозитории, у меня последняя stable версия, поэтому поищем в ветке backports: https://packages.debian.org/trixie-backports/amd64/kernel/.
Или через консоль следующими командами:
1. sudo apt search linux-image* | more
2. sudo apt search linux-image-6.19.10* (для конктерной версии)

**Установка:**
1) Добавим ветку backports (строчка deb http://deb.debian.org/debian/ trixie-backports main non-free-firmware) в пакетный менеджер (/etc/apt/sources.list).

   ![apt](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/linux%20kernel/screenshots/apt.png)

2) Установим image kernel (модули подтянутся с образом ядра) и headers:

   ![install](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/linux%20kernel/screenshots/install_headers_image.png)

3) Проверим директорию boot:

   ![boot](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/linux%20kernel/screenshots/boot_files.png)

4) Обновим конфигурацию GRUB:

```
sudo update-grub
sudo grub-set-default 0
```
5) Проверим версию ядра после ребута:

   ![new_kernel](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/linux%20kernel/screenshots/new_kernel.png)    
   
