# Работа с файловой системой zfs

## Задачи:
1. Определить алгоритм с наилучшим сжатием:
- определить, какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
- создать 4 файловых системы, на каждой применить свой алгоритм сжатия;
- для сжатия использовать либо текстовый файл, либо группу файлов.
2. Определить настройки пула.
- С помощью команды zfs import собрать pool ZFS.
- Командами zfs определить настройки:
    1) размер хранилища;
    2) тип pool;
    3) значение recordsize;
    4) какое сжатие используется;
    5) какая контрольная сумма используется.
3. Работа со снапшотами:
- скопировать файл из удаленной директории;
- восстановить файл локально. zfs receive;
- найти зашифрованное сообщение в файле secret_message.

Список всех дисков:

![lsblk](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/lsblk.png)

#### 1. Определим алгоритм с наилучшим сжатием
Создадим 4 пула типа зеркало:
```
root@debian:~# zpool create otus1 mirror /dev/sdb /dev/sdc
root@debian:~# zpool create otus2 mirror /dev/sde /dev/sdf
root@debian:~# zpool create otus3 mirror /dev/sdg /dev/sdh
root@debian:~# zpool create otus4 mirror /dev/sdd /dev/sdi
```

![zpool_list](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/zpool_list.png)

Применим к пулам разные алгоритмы сжатия: 
```
root@debian:~# zfs set compression=lzjb otus1
root@debian:~# zfs set compression=lz4 otus2
root@debian:~# zfs set compression=gzip-9 otus3
root@debian:~# zfs set compression=zle otus4
```

![get_compression](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/get_compression.png)

скачаем один файл во все пулы:

```
for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```

![copy](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/copy.png)

gzip-9 оказался самым эффективным

![copy](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/best.png)

#### 2. Определение настроек пула

Скачаем архив в домашний каталог и разархивируем его: 

```
 wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'

tar -xzvf archive.tar.gz
```

Проверим возможность импорта:

![import_check](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/import_check.png)

Импортируем пул otus и проверим информацию о его составе:

![import_status](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/import_status.png)

**Всю информацию о пуле можно получить командой** zfs get all *pool*

```
root@debian:~# zfs get all otus
NAME  PROPERTY              VALUE                      SOURCE
otus  type                  filesystem                 -
otus  creation              Пт мая 15  9:00 2020  -
otus  used                  2.04M                      -
otus  available             350M                       -
otus  referenced            24K                        -
otus  compressratio         1.00x                      -
otus  mounted               yes                        -
otus  quota                 none                       default
otus  reservation           none                       default
otus  recordsize            128K                       local
otus  mountpoint            /otus                      default
otus  sharenfs              off                        default
otus  checksum              sha256                     local
otus  compression           zle                        local
otus  atime                 on                         default
otus  devices               on                         default
otus  exec                  on                         default
otus  setuid                on                         default
otus  readonly              off                        default
otus  zoned                 off                        default
otus  snapdir               hidden                     default
otus  aclmode               discard                    default
otus  aclinherit            restricted                 default
otus  createtxg             1                          -
otus  canmount              on                         default
otus  xattr                 on                         default
otus  copies                1                          default
otus  version               5                          -
otus  utf8only              off                        -
otus  normalization         none                       -
otus  casesensitivity       sensitive                  -
otus  vscan                 off                        default
otus  nbmand                off                        default
otus  sharesmb              off                        default
otus  refquota              none                       default
otus  refreservation        none                       default
otus  guid                  14592242904030363272       -
otus  primarycache          all                        default
otus  secondarycache        all                        default
otus  usedbysnapshots       0B                         -
otus  usedbydataset         24K                        -
otus  usedbychildren        2.01M                      -
otus  usedbyrefreservation  0B                         -
otus  logbias               latency                    default
otus  objsetid              54                         -
otus  dedup                 off                        default
otus  mlslabel              none                       default
otus  sync                  standard                   default
otus  dnodesize             legacy                     default
otus  refcompressratio      1.00x                      -
otus  written               24K                        -
otus  logicalused           1020K                      -
otus  logicalreferenced     12K                        -
otus  volmode               default                    default
otus  filesystem_limit      none                       default
otus  snapshot_limit        none                       default
otus  filesystem_count      none                       default
otus  snapshot_count        none                       default
otus  snapdev               hidden                     default
otus  acltype               off                        default
otus  context               none                       default
otus  fscontext             none                       default
otus  defcontext            none                       default
otus  rootcontext           none                       default
otus  relatime              on                         default
otus  redundant_metadata    all                        default
otus  overlay               on                         default
otus  encryption            off                        default
otus  keylocation           none                       default
otus  keyformat             none                       default
otus  pbkdf2iters           0                          default
otus  special_small_blocks  0                          default
otus  prefetch              all                        default
otus  direct                standard                   default
otus  longname              off                        default
root@debian:~#
```

**Для вывода конкретного поля команда будет выглядеть так:** 

*zfs get compression otus - для определения алгоритма сжатия в пуле otus
zfs get available otus - для определения свободного места в пуле otus
и т.д*

#### 3. Работа со снапшотами

Скачаем снапшот, восстановим файловую систему из снапшота и найдем скрытое сообщение:

![download](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/download.png)


![secret](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/zfs/screen/secret.png)

