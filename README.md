# Drupal-stack via Ansible
## 1. Создание виртуальной машины
Создан файл Vagrantfile:
```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.hostname = "drupal-host"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
end
```
Команды:

  - ```vagrant up``` - запуск ВМ
  - ```vagrant ssh``` - подключение к виртуальной иашине

## 2. Установка ролей Ansible
На виртуальной машине:
выполнены команды для установки ролей:
```
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.postgresql
ansible-galaxy install geerlingguy.php
ansible-galaxy install geerlingguy.drupal
```

## 3. Конфигурация Ansible
Создаются файлы конфигурации:
Файл inventory.ini:
```
[drupal]
localhost ansible_connection=local
```

Файл drupal.yml:
```
---
- name: Deploy Drupal CMS
  hosts: drupal
  become: yes
  connection: local

  vars:
    postgresql_postgres_password: "postgres"
    postgresql_login_user: "postgres"
    postgresql_login_password: "postgres"
    postgresql_login_host: "localhost"
    postgresql_users:
      - name: drupal
        password: drupalpass
    postgresql_databases:
      - name: drupal
        owner: drupal

    pre_tasks:
      - name: Set become for PostgreSQL tasks
        set_fact:
          ansible_become: true

    composer_path: /usr/local/bin/composer
    drupal_domain: "drupal.local"
    drupal_install_site: true
    drupal_site_name: "My Drupal Site"
    drupal_admin_user: admin
    drupal_admin_password: adminpass
    drupal_admin_email: admin@example.com
    drupal_db_user: drupal
    drupal_db_password: drupalpass
    drupal_db_name: drupal
    drupal_db_backend: pgsql

  roles:
    - geerlingguy.nginx
    - geerlingguy.postgresql
    - geerlingguy.php
    - geerlingguy.drupal
```

## Что разворачивается
* Debian 12 (Box `debian/bookworm64`)
* NGINX 1.22
* PostgreSQL 15
* PHP 8.1 (Sury repo)
* Drupal 10 (Composer-project) + Drush 10

## 4. Запуск сценария
```
ansible-playbook -i inventory.ini drupal.yml | tee playbook.log
```

## Логи
Полный вывод находится в playbook.log и заканчивается:
```
PLAY RECAP *********************************************************************
localhost                  : ok=80   changed=4    unreachable=0    failed=0    skipped=47   rescued=0    ignored=0 
```
Drupal доступен по http://192.168.56.10/
