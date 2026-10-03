# Ansible
## Задача:
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible, необходимо развернуть nginx со следующими условиями:
1. Необходимо использовать модуль yum/apt;
2. Конфигурационные файлы должны быть взяты из шаблона jinja2 с переменными;
3. После установки nginx должен быть в режиме enabled в systemd;
4. Должен быть использован notify для старта nginx после установки;
5. Сайт должен слушать на нестандартном порту — 8080, для этого использовать переменные в Ansible.

------------------------------------------------------------------------------------------------------------------------------------------
**Подготовка:**
Установим ansible:

```
sudo apt update
sudo apt install -y pipx
pipx ensurepath
pipx install --include-deps ansible
```
Развернем vm из [vagrantfile](https://drive.google.com/file/d/17MEtg20TFSjKil6ih7PvPez7jmCvo6fb/view?usp=share_link)

Прочитаем параметры ssh vagrant:
```
mirananight@miranakomp:~/otus/ansible$ vagrant ssh-config
Host nginx
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/mirananight/otus/ansible/.vagrant/machines/nginx/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
  PubkeyAcceptedKeyTypes +ssh-rsa
  HostKeyAlgorithms +ssh-rsa

mirananight@miranakomp:~/otus/ansible$ 
```

Создадим inventory файл:

```
mirananight@miranakomp:~/otus/ansible$ cat staging/hosts
[web]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/nginx/virtualbox/private_key
mirananight@miranakomp:~/otus/ansible$ 
```

**Создадим конфиг файл для ansible:**
- *host_key_checking = False* - небезопасно, отключает проверку fingerprint удаленного хоста, вне лабораторных условий лучше использовать ssh-keyscan -H web >> ~/.ssh/known_hosts - чтобы заранее заполнить файл с fingerprint удаленных хостов.  

- retry_files_enabled = False в Ansible отключает создание .retry-файлов при сбое выполнения плейбука.
```
mirananight@miranakomp:~/otus/ansible$ cat ansible.cfg 
[defaults]
inventory = staging/hosts
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False
mirananight@miranakomp:~/otus/ansible$ 
```
**Проверим доступность хоста:**

```
ansible nginx -m ping
[WARNING]: Host 'nginx' is using the discovered Python interpreter at '/usr/bin/python3.10', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
```

**Посмотрим какое ядро установлено на хосте:**

```
mirananight@miranakomp:~/otus/ansible$ ansible nginx -m command -a "uname -r"
[WARNING]: Host 'nginx' is using the discovered Python interpreter at '/usr/bin/python3.10', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
nginx | CHANGED | rc=0 >>
5.15.0-91-generic
mirananight@miranakomp:~/otus/ansible$ 
```

**проверим статус службы firewalld:**

```
mirananight@miranakomp:~/otus/ansible$ ansible nginx -m systemd -a name=firewalld
[WARNING]: Host 'nginx' is using the discovered Python interpreter at '/usr/bin/python3.10', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "name": "firewalld",
    "status": {
        "ActiveEnterTimestamp": "n/a",
        "ActiveEnterTimestampMonotonic": "0",
        "ActiveExitTimestamp": "n/a",
        "ActiveExitTimestampMonotonic": "0",
        "ActiveState": "inactive",
```
Отредактируем ansible.cfg чтобы убрать предупреждение о python интерпретаторе, зададим жестко путь к интерпретатору: 

```
[defaults]
inventory = staging/hosts
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False
interpreter_python = /usr/bin/python3
```

**Создадим шаблон конфигурации nginx используя jinja2 в templates/nginx.conf.j2:**

```
mirananight@miranakomp:~/otus/ansible/templates$ cat nginx.conf.j2 
# {{ ansible_managed }}
events {
    worker_connections 1024;
}

http {
    server {
        listen       {{ nginx_listen_port }} default_server;
        server_name  default_server;
        root         /usr/share/nginx/html;

        location / {
        }
    }
}
```
**Напишем playbook, который установит nginx и подставит шаблон templates/nginx.conf.j2 + добавим необходимые tags:**

```
---
- name: NGINX | install and configure NGINX
  hosts: nginx
  become: true
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: apt update
      apt:
        update_cache=yes
      tags:
       - update apt

    - name: nginx | install
      apt:
        name: nginx
        state: latest
      tags:
        - nginx-package

    - name: NGINX  | Create NGINX config file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags:
        - nginx-configuration       
```

**Добавим hadler и notify чтобы каждый раз когда конфиг nginx мнеяется сервис перезапускался:**

```
---
- name: NGINX | install and configure NGINX
  hosts: nginx
  become: true
  vars:
    nginx_listen_port: 8080

  tasks:
    - name: apt update
      apt:
        update_cache=yes
      tags:
        - update apt

    - name: nginx | install
      apt:
        name: nginx
        state: latest
      notify:
        - restart nginx
      tags:
        - nginx-package


    - name: NGINX  | Create NGINX config file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - reload nginx
      tags:
        - nginx-configuration

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes

    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```
**Запустим playbook:**

![result](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/ansible/png/result.png)


**Проверим работу nginx на порту 8080:**


![res1](https://github.com/MIranaNightshade/otus-linux-professional/blob/main/ansible/png/result1.png)



