# bash скрипты

## Цель:
- написать bash-скрипт, который ежечасно формирует и отправляет на email отчёт о работе веб-сервера;

### Задача:

1. Написать скрипт для CRON, который раз в час формирует отчёт и отправляет его на заданную почту.

Отчёт должен содержать:

- IP-адреса с наибольшим числом запросов (с момента последнего запуска);
- Запрашиваемые URL с наибольшим числом запросов (с момента последнего запуска);
- Ошибки веб-сервера/приложения (с момента последнего запуска);
- HTTP-коды ответов с указанием их количества (с момента последнего запуска).

________________________________________________________________________________________________________________________________________
**Скрипт:**

```
#!/bin/bash

# Функция очистки временных файлов

cleanup(){
   rm -f /tmp/log_to_mail.tmp
   rm -f /tmp/script-log.lock
   echo "I cleanup trash, master"
}


# find -inum поиск по номеру inode


# Блокируем запуск более одного экземпляра скрипта одновременно

exec 200>/tmp/script-log.lock
flock 200



#access_4560_644067-586145-be5697.log
log_path=/home/mirana/access_4560_644067-586145-be5697.log


# Если файл с переменными не существует, то создаем его и записывам переменные: номер строки = 1 (т.к читать будем с начала)
# $string_start

if [ ! -f var-script.txt ]; then
   touch var-script.txt
   echo -e "string_start=1" > var-script.txt
fi

source var-script.txt


# Проверяем что есть новые строки в файле логов:
total_lines=$(wc -l < $log_path)
if  [ $string_start -gt $total_lines ]; then
   echo нет новых строк в логах
   exit 0
fi






# Определим последнюю строку в лог файле
string_last=$(sed -n '$=' $log_path)

# Найдем начальную и конечную дату/время исследуемой части лога
start_date=$(awk -v str=$string_start 'NR==str {print}' $log_path | grep -Po '\[\d{1,2}/\w{3}/.*]')
last_date=$(awk -v str=$string_last 'NR==str {print}' $log_path | grep -Po '\[\d{1,2}/\w{3}/.*]')

#  Найдем  наибольшее количество запросов на один уникальный IP
max_requests_per_ip=$(sed -n "$string_start,$string_last"'p' $log_path | awk '{print $1}' | sort | uniq -c | sort -n | tail -n 1 | awk '{print $1}')
max_req_uniq_ip=$(sed -n "$string_start,$string_last"'p' $log_path | awk '{print $1}' | sort | uniq -c | sort -n | awk -v max=$max_requests_per_ip  '$1 >= max {print $0}')

#  Найдем  наибольшее количество запросов на один уникальный url

max_req_per_url=$(sed -n "$string_start,$string_last"'p' $log_path | grep -Po "http[s]{0,1}://\S*" | sort | uniq -c | sort -n | tail -n 1 | awk '{print $1}')
max_req_uniq_url=$(sed -n "$string_start,$string_last"'p' $log_path | grep -Po "http[s]{0,1}://\S*" | sort | uniq -c | sort -n | awk -v max=$max_req_per_url  '$1 >= max {print $0}')


# Найдем в логах ошибки сервера
server_err=$(sed -n "$string_start,$string_last"'p' $log_path  | grep -P 'HTTP/(\d\.\d||[23]{1})\"\s*5')
# Найдем HTTP коды ответов с указанием их количества
http_code=$(sed -n "$string_start,$string_last"'p' $log_path  |  grep -Po 'HTTP/(\d\.\d||[23]{1})\"\s*\d{3}' | grep -Po '\d{3}' | sort | uniq -c | sort -n)
# Отправляем на почту

cat << EOF > /tmp/log_to_mail.tmp
Отчет по логу:

Исследуемое время: $start_date по $last_date

IP-адреса с наибольшим числом запросов:

$max_req_uniq_ip

Запрашиваемые URL с наибольшим числом запросов:

$max_req_uniq_url

Ошибки веб-сервера/приложения:

$server_err

HTTP-коды ответов с указанием их количества:

$http_code
EOF

cat /tmp/log_to_mail.tmp

# Отправляем себе на почту
mail -s "log" nuh_@mail.ru < /tmp/log_to_mail.tmp

# Перезапишем значение переменной в var файле
echo -e "string_start=$((string_last+1))" > var-script.txt

trap cleanup EXIT INT TERM ERR
```

**Результат выполнения скрипта:**

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/bash/screen/script.png)

![](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/bash/screen/script1.png)


