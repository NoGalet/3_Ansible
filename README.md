Первые шаги с Ansible
Цель домашнего задания
Написать первые шаги с Ansible.
Описание домашнего задания
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible необходимо развернуть nginx со следующими условиями:
- необходимо использовать модуль yum/apt
- конфигурационный файлы должны быть взяты из шаблона jinja2 с
переменными
- после установки nginx должен быть в режиме enabled в systemd
- должен быть использован notify для старта nginx после установки
- сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible

1) Создадим свой первый inventory файл ./staging/hosts. Со следующим содержимым:
[web]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_private_key_file=/home/stas/ansible/.vagrant/machines/nginx/virtualbox/private_key

2) убедимся, что Ansible может управлять нашим хостом:
stas@ubuntu2:~/ansible$ ansible nginx -i staging/hosts -m ping
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:tOxuu1kaR6byeLtAn10F5FA8UXninwN22TNC/oaQ95U.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

3) создадим файл ansible.cfg со следующим содержанием:
[defaults]
inventory = staging/hosts
remote_user = vagrant
host_key_checking = False
retry_files_enabled = False

4) Теперь из инвентори можно убрать информацию о пользователе и убедимся, что можем управлять нашим хостом, только теперь без явного указания inventory файла:
stas@ubuntu2:~/ansible$ ansible nginx -m ping
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

5) Посмотрим, какое ядро установлено на хосте:
stas@ubuntu2:~/ansible$ ansible nginx -m command -a "uname -r"
nginx | CHANGED | rc=0 >>
5.15.0-91-generic

6) Проверим статус сервиса firewalld
stas@ubuntu2:~/ansible$ ansible nginx -m systemd -a name=firewalld
nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "name": "firewalld",
    "status": {

7) Напишем и выполним playbook nginx.yml:
stas@ubuntu2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] ***********************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [nginx]

TASK [update] ****************************************************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] *************************************************************************
changed: [nginx]

PLAY RECAP *******************************************************************************************
nginx                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

8) Дополним и выполним playbook nginx.yml:
stas@ubuntu2:~/ansible$ ansible-playbook nginx.yml

PLAY [NGINX | Install and configure NGINX] ***********************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [nginx]

TASK [update] ****************************************************************************************
changed: [nginx]

TASK [NGINX | Install NGINX] *************************************************************************
ok: [nginx]

TASK [NGINX | Create NGINX config file from template] ************************************************
ok: [nginx]

TASK [NGINX | Restart nginx] *************************************************************************
changed: [nginx]

TASK [NGINX | Reload nginx] **************************************************************************
changed: [nginx]

PLAY RECAP *******************************************************************************************
nginx                      : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

9) Проверяем на хосте, что сайт поднялся:
vagrant@nginx:~$ curl http://192.168.11.150:8080
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
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
