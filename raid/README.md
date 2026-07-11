# Работа с рейд массивами
## Задание:
1. Добавить в виртуальную машину несколько дисков
2. Собрать RAID-0/1/5/10 на выбор
3. Сломать и починить RAID
4. Создать GPT таблицу, пять разделов и смонтировать их в системе.

### Выполнение:

1) **Просмотрим данные о блочных устройствах в системе:**
   Команды:

   ```
   lsblk
   ```
   или
   ```
   sudo fdisk -l
   ```

   ![blk_inf](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/blk_inf.png)

2) **Создадим рейд массив:**
    *Обнулить суперблоки:*
   
    ```
    mdadm --zero-superblock --force /dev/sd{b,c,d,e}
    ```
    
    *Создание рейд массива:*
   
    ```
    sudo mdadm --create /dev/md123 -l 10 -n 4 /dev/sd{b,c,d,e}
    ```

    *Проверка информации о  созданном рейд массиве:*

    ```
    cat /proc/mdstat
    ```

   или

   ```
   sudo mdadm -D /dev/md123
   ```
   ![md_stat](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/md_create_stat.png)

   ![mdadm_d](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/mdadm_d.png)

3) **Сломаем и починим рейд массив:**

      *Пометим один диск как fail и проверим состояние массива*

      ```
      sudo mdadm /dev/md123 --fail /dev/sdc
      sudo mdadm -D /dev/md123
      cat /proc/mdstat
      ```
     ![mdadm_fail](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/mdadm_fail.png)

      *Удалим "сломанный" диск из массива и добавим "новый исправный" диск в массив:*

      ```
      sudo mdadm /dev/md123 --remove /dev/sdc
      sudo mdadm /dev/md123 --add /dev/sdc
      ```
     ![mdadm_del_recovery](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/del_recovery.png)

     
4) **Создадим GPT таблицу, пять разделов и смонтируем их в системе.**

     *Создадим 5 логических разделов*
   
     ```
     mirana@debian:~$ sudo parted -s /dev/md123 mklabel gpt
     mirana@debian:~$ sudo parted -s /dev/md123 mkpart primary ext4 0% 20%
     mirana@debian:~$ sudo parted -s /dev/md123 mkpart primary ext4 20% 40%
     mirana@debian:~$ sudo parted -s /dev/md123 mkpart primary ext4 40% 60%
     mirana@debian:~$ sudo parted -s /dev/md123 mkpart primary ext4 60% 80%
     mirana@debian:~$ sudo parted -s /dev/md123 mkpart primary ext4 80% 100%
     ```
     Проверим результат:

   ![mdadm_del_recovery](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/parted_result.png)

   *Создадим фаловую систему на каждом разделе*

    ```
    for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md123p$i; done
    ```

   ![ext4](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/ext4.png)

   *Смонтируем все логические разделы*

   ```
   for i in $(seq 1 5); do sudo mount /dev/md123p$i /mnt/raid/part$i; done
   ```

   ![mount](https://github.com/MIranaNightshade/otus-linux-professional/tree/main/raid/screen/mount.png)

   
