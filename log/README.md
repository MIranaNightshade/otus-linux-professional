# Сбор и анализ логов
## Цель
Научится проектировать централизованный сбор логов. Рассмотреть особенности разных платформ для сбора логов.
### Задачи
1. В Vagrant разворнуть 2 виртуальные машины web и log
2. на web настроить nginx
3. на log настроить центральный лог сервер на любой системе на выбор
journald;
rsyslog;
elk.
4. настроить аудит, следящий за изменением конфигов nginx 

*Все критичные логи с web должны собираться и локально и удаленно.
Все логи с nginx должны уходить на удаленный сервер (локально только критичные).
Логи аудита должны также уходить на удаленную систему.*

_________________________________________________________________________________________________________________________________________________

#### Будем использовать ELK.


![elk](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/log/png/ELK.png)


##### 1. Разворачиваем две vm с помощью Vagrantfile:

```
Vagrant.configure("2") do |config|
    config.vm.box = "bento/ubuntu-22.04"
    config.vm.provider :virtualbox do |v|
        v.memory = 1512
        v.cpus = 2
    end

    boxes = [
        { :name => "web",
          :ip => "192.168.56.10",
        },
        { :name => "log",
          :ip => "192.168.56.15",
        }
    ]
    boxes.each do |opts|
        config.vm.define opts[:name] do |config|
            config.vm.hostname = opts[:name]
            config.vm.network "private_network", ip: opts[:ip]
        end
    end
end
```
##### 2. На vm web настроим nginx и отправку access/error log:

```
apt update
apt install nginx
```
Настроим отправку логов, error логи будут собираться локально и удаленно, access логи только удаленно.

настроим конфиг nginx следующим образом:

*nginx будет отправлять логи на localhost:514 где их будет слушать rsyslog и отправлять на наш сервер с logstash, т.к nginx отправляет логи по udp, а это ненадежно, rsyslog умеет отправлять по tcp*

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log syslog:server=127.0.0.1:514,tag=nginx_access,severity=info combined;
        error_log /var/log/nginx/error.log;
        error_log syslog:server=127.0.0.1:514,tag=nginx_error;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}

```
настраиваем rsyslog:

```
root@web:~# cat /etc/rsyslog.d/nginx_to_tcp.conf 
# Прием логов от Nginx на локальный UDP-порт
module(load="imudp")
input(type="imudp" port="514" address="127.0.0.1")

# Пересылка на удаленный сервер по TCP (две собачки "@@" означают TCP)
if $programname == 'nginx_access' then @@192.168.56.15:5514
if $programname == 'nginx_error' then @@192.168.56.15:5514
root@web:~#
root@web:~# systemctl restart rsyslog
root@web:~#  
```

#####**3. Настроим vm log на прием логов от nginx.**

Установим logstash, elasticsearch и kibana:
Для начала подключим яндекс репозиторий (зеркало) для elk:
```
sudo apt update
sudo apt install gpg curl
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic-archive-keyring.gpg] https://mirror.yandex.ru/mirrors/elastic/8/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```
**Установим default-jdk необходим для ELK т.к компоненты ELK написаны на java**

```
sudo apt install default-jdk
```

```
sudo apt install logstash elasticsearch kibana
```

**Настроим elasticsearch:**
/etc/elasticsearch/elasticsearch.yml

```
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
#
# Path to log files:
#
path.logs: /var/log/elasticsearch

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: false
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: false
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
cluster.initial_master_nodes: ["log"]

# Allow HTTP API connections from anywhere
# Connections are encrypted and require user authentication
http.host: 0.0.0.0

# Allow other nodes to join the cluster from anywhere
# Connections are encrypted and mutually authenticated
#transport.host: 0.0.0.0

#----------------------- END SECURITY AUTO CONFIGURATION -------------------------

```

**Настроим logstash:**

в /etc/logstash/logstash.yml

```
path.data: /var/lib/logstash
path.config: /etc/logstash/conf.d
```

в /etc/logstash/conf.d//etc/logstash/conf.d/logstash-nginx.conf

```
input {
  tcp {
    id => "nginx_log"
    host => "0.0.0.0"
    port => 5514
    type => "nginx_syslog"
  }
}

output {
 elasticsearch {
   hosts => ["http://localhost:9200"]
   index => "weblogs-%{+YYYY.MM.dd}"
 }
 stdout { codec => rubydebug }
}
```

**Настроим kibana:**
/etc/kibana/kibana.yml

```

server.port: 5601
server.host: "0.0.0.0"


# Enables you to specify a file where Kibana stores log output.
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file
```

Перезагрузим сервисы:

```
systemclt restart logstash
systemclt restart elasticsearch
systemclt restart kibana
```
__________________________________________________________________________________________

**Проверка:**

на vm web выполним curl http://localhost и зайдем в веб интерфейс kibana http://192.168.56.15:5601/

Management ----> Index Management
Видим что индекс создался, логи приходят.

![index](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/log/png/kibana-indexpng.png)

Создадим data view: 

management -----> kibana ------> data view


![data](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/log/png/data-view.png)

Проверим логи:

analitycs -----> Discover и выбираем наш созданный view:

![log](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/log/png/logs.png)


переместим index.nginx-debian.html и проверим error logs:

```
mv /var/www/html/index.nginx-debian.html /var/www/
```

![error](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/log/png/error.png)



##### 4. Настроим аудит чтобы следить за изменением конфига nginx. 
Будем использовать auditd

**НА СЕРВЕРЕ WEB:**

Настроим отслеживание изменений в директории /etc/nginx/:
/etc/audit/rules.d/nginx-watch.rules
-w - следить за директорией
-p wa — отслеживать write (запись) и attribute (изменение атрибутов)
-k nginx_config_changes — ключ (метка) логов

```
-w /etc/nginx/ -p wa -k nginx_conf_change
```
настроим filebeat:

```
root@web:~# cat /etc/filebeat/filebeat.yml 
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/audit/audit.log
    include_lines: ['nginx_conf_change']
    fields:
      source: auditd

output.logstash:
  hosts: ["192.168.56.15:5044"]
```

**НА СЕРВЕРЕ LOG:**

Настроим pipelines чтобы логи не смешивались:

```
root@log:~# cat /etc/logstash/pipelines.yml
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
#   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: audit-nginx
  path.config: "/etc/logstash/conf.d/logstash-audit.conf"

- pipeline.id: nginx-weblogs
  path.config: "/etc/logstash/conf.d/logstash-nginx.conf"
```
Настроим logstash чтобы принимать логи от filebeat:

```
root@log:~# cat /etc/logstash/conf.d/logstash-audit.conf  
input {
  beats {
    port => 5044
  }
}

filter {
  if "nginx_conf_change" not in [message] {
    drop {}
  }

#  mutate {
#    add_tag => ["audit_nginx"]
#  }
}

output {
 elasticsearch {
   hosts => ["http://localhost:9200"]
   #cacert => '/etc/logstash/certs/http_ca.crt'
   #ssl => true
   index => "audit-nginx-%{+YYYY.MM.dd}"
 }
 stdout { codec => rubydebug }
}
cat:  : No such file or directory
root@log:~# 
```

