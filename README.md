# Описание [playbook](./site.yml)

## Содержание [playbook](./site.yml)

### Plays
Установка Clickhouse,vector,lighthouse
-  Install Clickhouse
-  Install vector

### Tasks

#### Play - Install Clickhouse
- Get clickhouse distributives - скачивает clickhouse, версия укакзывается в [clickhouse_vars.yml](./group_vars/clickhouse/clickhouse_vars.yml)
- Install clickhouse packages - устанавливает скаченный дистрибутив clickhouse
- Flush handlers - вызывает hundlers, который в свою очередь запускает clickhouse после установки.
- Create database - создает БД.

#### Play - Install vector
- Get vector distrib - скачивает clickhouse, версия укакзывается в [vector_vars.yml](./group_vars/vector/vector_vars.yml)
- Install vector - устанавливает скаченный дистрибутив vector
- Configure vector - конфигурирация vector, переменные конфига подставляемые в [шаблон](./template/vector/vector.toml.j2) конфигурационного файла указанны в [vector_vars.yml](./group_vars/vector/vector_vars.yml)

#### Play - Install nginx and lighthouse
- start/restart nginx service - запускает стандартный сервис nginx
- setenforce - отключаем SELinux
- Create a directory if it does not exist - создаем дирректорию для лайтхауса если ее нет
- install git - устанавливаем гит
- copy lighthouse service from git - копируем репо гита
- copy lighthouse files1 to nginx workdir - копируем гит в рабочую дирректорию nginx-а
- create lighthouse config - билдим конфиг для nginx с помощью шаблона

## Tags
- `vector` по умолчанию выполняется, установлен на Play - `Install vector`
- `clickhouse` по умолчанию выполняется, установлен на Play - `Install Clickhouse`
- `lighthouse` по умолчанию выполняется, установлен на Play - `Install nginx and lighthouse`

## Используемые переменные

Файлы со значениями переменных лежат в директории `playbook/group_vars`

### Clickhouse 
[clickhouse_vars.yml](./group_vars/clickhouse/clickhouse_vars.yml) тут мы указываем версию clickhouse которая будет установленна и пакеты.
```yml
---
clickhouse_version: "22.3.3.44"
clickhouse_packages:
  - clickhouse-client
  - clickhouse-server
  - clickhouse-common-static
```

### Vector 
[vector_vars.yml](./group_vars/vector/vector_vars.yml) тут мы указываем архитектуру rpm-пакета vector который будет скачен и далее установлен и тип sourse.in для конфигурации vector.
```yml
---
vector_arch: "x86_64"
sourse_in_type: "stdin"
```

## Группы хостов
[prod.yml](./inventory/prod.yml) 
```yml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_connection: docker
vector:
  hosts:
    clickhouse-01:
      ansible_connection: docker
```
## Lighthouse
[all.yml](./group_vars/all.yml) тут мы предоставляем доступ и указываем где логи будут
```yml
---
access_log_dir:  /var/log/nginx/lighthouse-access.log
error_log_dir: /var/log/nginx/lighthouse-error.log
```
У нас три группы хостов 
- `clickhouse` на все хосты данной группы будет установлен clickhouse и его пакеты и далее запущен
- `vector` на все хосты данной группы будет установлен vector и сконфигурирован
- `lighthouse` - на все хосты данной группы будет установлен lighthouse и сконфигурирован под nginx

Данный playbook работает только на centos, а точнее с пакетным менеджером yum.