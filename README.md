# Домашнее задание к занятию «Работа с Ansible»

## Подготовка

* Ansible установлен: версия `2.16.3`.
* Используется Docker-окружение для проверки playbook.
* Для выполнения заданий используются контейнеры:

  * `centos7` — CentOS 7;
  * `ubuntu` — Ubuntu 22.04;
  * `ubuntu-systemd` — Ubuntu 22.04 с systemd для проверки ролей ClickHouse, Vector и LightHouse.
* На целевых контейнерах установлен Python.

---

# Задание 4. Работа с roles

## Структура проекта

```text
08-ansible-01-base_02.25/
├── README.md
└── playbook/
    ├── site.yml
    ├── requirements.yml
    ├── inventory/
    │   ├── prod.yml
    │   └── test.yml
    ├── group_vars/
    │   ├── all/
    │   │   └── examp.yml
    │   ├── deb/
    │   │   └── examp.yml
    │   └── el/
    │       └── examp.yml
    └── roles/
        ├── clickhouse/
        ├── vector/
        └── lighthouse/
```

---

## Requirements

Для установки ролей создан файл:

```text
playbook/requirements.yml
```

Содержимое:

```yaml
---
roles:
  - src: git@github.com:AlexeySetevoi/ansible-clickhouse.git
    scm: git
    version: "1.13"
    name: clickhouse

  - src: git@github.com:kumarina-max/vector-role.git
    scm: git
    version: "v1.0.0"
    name: vector

  - src: git@github.com:kumarina-max/lighthouse-role.git
    scm: git
    version: "v1.0.0"
    name: lighthouse
```

Установка ролей выполняется командой:

```bash
ansible-galaxy role install -r requirements.yml -p roles
```

В результате устанавливаются:

* `clickhouse` версии `1.13`;
* `vector` версии `v1.0.0`;
* `lighthouse` версии `v1.0.0`.

---

## Роль ClickHouse

Для ClickHouse используется готовая роль:

```text
https://github.com/AlexeySetevoi/ansible-clickhouse
```

Версия роли:

```text
1.13
```

Роль подключена в `site.yml`:

```yaml
- name: Install ClickHouse
  hosts: clickhouse
  become: true
  roles:
    - clickhouse
```

Для проверки ClickHouse использовался отдельный Docker-контейнер `ubuntu-systemd` с systemd.

После выполнения роли версия ClickHouse проверяется командой:

```bash
docker exec ubuntu-systemd clickhouse-client --query "SELECT version()"
```

Результат:

```text
26.9.10.4
```

---

## Роль Vector

Для Vector создана собственная Ansible-роль.

Репозиторий роли:

```text
https://github.com/kumarina-max/vector-role
```

Версия роли:

```text
v1.0.0
```

Роль устанавливает Vector, добавляет официальный APT-репозиторий, устанавливает пакет, разворачивает конфигурацию и запускает сервис.

Роль подключена в `site.yml`:

```yaml
- name: Install Vector
  hosts: vector
  become: true
  roles:
    - vector
```

После выполнения playbook состояние сервиса проверяется командой:

```bash
docker exec ubuntu-systemd systemctl is-active vector
```

Результат:

```text
active
```

Конфигурация Vector также проходит проверку перед применением.

---

## Роль LightHouse

Для LightHouse создана собственная Ansible-роль.

Репозиторий роли:

```text
https://github.com/kumarina-max/lighthouse-role
```

Версия роли:

```text
v1.0.0
```

Роль выполняет:

* установку необходимых пакетов;
* загрузку LightHouse из GitHub;
* размещение файлов LightHouse;
* установку и настройку Nginx;
* настройку виртуального хоста;
* запуск Nginx.

Роль подключена в `site.yml`:

```yaml
- name: Install Lighthouse
  hosts: lighthouse
  become: true
  roles:
    - lighthouse
```

LightHouse настроен на порт:

```text
8080
```

Проверка наличия файлов LightHouse:

```bash
docker exec ubuntu-systemd ls -la /var/www/lighthouse
```

Проверка порта:

```bash
docker exec ubuntu-systemd ss -lntp | grep ":8080"
```

Nginx успешно слушает порт `8080`.

---

## Inventory

Для ролей созданы отдельные группы в `inventory/prod.yml`:

```yaml
clickhouse:
  hosts:
    ubuntu-systemd:
      ansible_connection: docker

vector:
  hosts:
    ubuntu-systemd:
      ansible_connection: docker

lighthouse:
  hosts:
    ubuntu-systemd:
      ansible_connection: docker
```

Один Docker-контейнер используется для проверки всех трёх ролей.

---

## Playbook

Файл:

```text
playbook/site.yml
```

Содержит три play:

```yaml
---
- name: Install ClickHouse
  hosts: clickhouse
  become: true
  roles:
    - clickhouse

- name: Install Vector
  hosts: vector
  become: true
  roles:
    - vector

- name: Install Lighthouse
  hosts: lighthouse
  become: true
  roles:
    - lighthouse
```

Таким образом, установка ClickHouse, Vector и LightHouse выполняется через Ansible Roles.

---

## Проверка playbook

### Проверка синтаксиса

Выполнена проверка:

```bash
ansible-playbook -i inventory/prod.yml site.yml --syntax-check --ask-vault-pass
```

Результат:

```text
playbook: site.yml
```

Ошибок синтаксиса нет.

---

### Первый запуск

Полный playbook был успешно выполнен:

```bash
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
```

Результат:

```text
ubuntu-systemd : ok=46 changed=16 unreachable=0 failed=0 skipped=10
```

Все три роли успешно отработали:

* ClickHouse установлен;
* Vector установлен и запущен;
* LightHouse установлен;
* Nginx настроен и запущен.

---

### Проверка сервисов

ClickHouse:

```bash
docker exec ubuntu-systemd clickhouse-client --query "SELECT version()"
```

Результат:

```text
26.9.10.4
```

Vector:

```bash
docker exec ubuntu-systemd systemctl is-active vector
```

Результат:

```text
active
```

Nginx:

```bash
docker exec ubuntu-systemd systemctl is-active nginx
```

Результат:

```text
active
```

LightHouse:

```bash
docker exec ubuntu-systemd ss -lntp | grep ":8080"
```

Nginx слушает порт `8080`.

---

## Проверка идемпотентности

Playbook был запущен повторно той же командой:

```bash
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
```

Результат:

```text
ubuntu-systemd : ok=44 changed=0 unreachable=0 failed=0 skipped=10
```

Параметр:

```text
changed=0
```

показывает, что повторный запуск не вносит изменений в уже настроенную систему.

Таким образом, playbook является идемпотентным.

---

## Итог

В рамках работы:

* playbook переработан с использованием Ansible Roles;
* подключена готовая роль ClickHouse версии `1.13`;
* создана собственная роль Vector;
* создана собственная роль LightHouse;
* роли Vector и LightHouse опубликованы в отдельных GitHub-репозиториях;
* создан `requirements.yml`;
* выполнена установка ролей через `ansible-galaxy`;
* выполнена проверка синтаксиса playbook;
* выполнен полный запуск playbook;
* проверена работоспособность ClickHouse, Vector и Nginx/LightHouse;
* выполнена повторная проверка playbook с результатом `changed=0`.

### Репозитории ролей

**Vector:**

```text
https://github.com/kumarina-max/vector-role
```

**LightHouse:**

```text
https://github.com/kumarina-max/lighthouse-role
```

**Playbook:**

```text
https://github.com/kumarina-max/ansible-roles-playbook
```

