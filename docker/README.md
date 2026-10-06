# Docker
## Задачи:

1. Установите Docker и Docker Compose на хост машину https://docs.docker.com/engine/install/ubuntu/ 
2. Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)
3. Определите разницу между контейнером и образом
4. Ответьте на вопрос: Можно ли в контейнере собрать ядро?

--------------------------------------------------------------------------------------------------------------------------------

#### 1. Установим docker + docker compose  на Ubuntu:

Удалим конфликтующие паеты

```
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```
Добавим репозиторий: 

```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Установим необходимые пакеты:

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 2. Создадим кастомный nginx на базе Alpine.

Напишем Dockerfile:

```
FROM alpine:3.24
RUN apk add --no-cache nginx
COPY index.html cat.png /usr/share/nginx/html/
COPY default.conf /etc/nginx/http.d/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

index.html:

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to CATginx!</h1>
<img src="cat.png" alt="кот" width="300" height="278">
<p>If you see this page, the CATginx web server is successfully installed and
working. Further configuration is required.</p>



<p><em>Thank you for using CATginx.</em></p>
</body>
```
Создадим образ:

```
sudo docker build -t mirananightshade/catginx:1.0 .
```
Запустим на хосте:

```
sudo docker run -d -p 80:80 --name catginx  nginx_custom:1.0
```

Запушим на докерхаб:

```
sudo docker image push mirananightshade/catginx:1.0
```

#### 3. Разница между контейнером и образом:

Образ (image) - это неизменяемый шаблон контейнера.
Контейнер - это развернутый экземпляр образа, запущенный на хосте изолированный с помощью namespaces, cgroups процесс.

#### 4. Можно ли собрать в контейнере ядро?

Нет, собрать ядро нельзя, контейнер работает на основе ядра хостовой ОС. 
