# Ansible - инструмент для автоматизации задач по управлению, настройке и обслуживанию серверов или сетевых устройств

## Команды для работы с Ansible

| Команда | Описание | Краткие параметры | Пример |
|---|---|---|---|
| `ansible` | Одноразовые ad‑hoc команды | `-i inventory`, `-m <module>`, `-a <args>` | `ansible web -i hosts -m ping` |
| `ansible-playbook` | Запуск playbook (yaml) | `-i`, `--limit`, `--check`, `--diff` | `ansible-playbook -i hosts site.yml` |
| `ansible-galaxy` | Работа с ролями/коллекциями | `install`, `init` | `ansible-galaxy install geerlingguy.nginx` |
| `ansible-vault` | Шифрование секретов | `create`, `encrypt`, `decrypt` | `ansible-vault encrypt secrets.yml` |
| `ansible-lint` | Статический анализ playbook | — | `ansible-lint site.yml` |

## Структура каталога Ansible

| Корневой уровень | Назначение | Вложенный уровень |
|---|---|---|
| `inventories/` | Файлы инвентаря / групповые переменные | `hosts`, `group_vars/`, `host_vars/` |
| `playbooks/` | Основные playbook'и | `site.yml`, `deploy.yml` |
| `roles/<role>/` | Роли (tasks/handlers/files/templates) | Стандартизированная структура: `tasks/`, `handlers/`, `defaults/` |
| `group_vars/` | Параметры групп хостов | Перекрываются `host_vars` |
| `host_vars/` | Переменные конкретного хоста | Использовать для уникальных конфигураций |
| `library/` | Локальные модули | Для кастомных модулей |
| `collections/` | Коллекции Ansible | `ansible-galaxy` коллекции |

### Примеры использования Ansible

| Сценарий | Что делает | Быстрая команда/фрагмент |
|---|---|---|
| Деплой приложения | Копирует артефакт, обновляет systemd, перезапускает сервис | `ansible-playbook -i hosts deploy.yml` |
| Установка пакета | Модуль `apt`/`yum` управляет пакетами | `- name: install nginx\n  apt: name=nginx state=present` |
| Управление конфигом | Шаблон Jinja2 → `/etc/app/config` | `template: src=config.j2 dest=/etc/app/config` |
| Ротация логов / cron | Создать cron job или конфиг | `cron:` модуль или шаблон в `/etc/cron.d/` |
| Секреты | Шифрование с `ansible-vault` и использование в playbook | `ansible-vault encrypt secrets.yml` и `vars_files:` |

<details>
<summary>Install and start Nginx</summary>
```yaml
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Update apt cache (Debian/Ubuntu)
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Ensure Nginx is installed
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Ensure Nginx service is enabled and running
      service:
        name: nginx
        state: started
        enabled: yes
```
</details>

<details>
<summary>Deploy simple app binary</summary>
```yaml
- name: Deploy simple app binary and systemd service
  hosts: app
  become: true
  vars:
    app_name: myapp
    deploy_dir: /opt/myapp

  tasks:
    - name: Create deploy directory
      file:
        path: "{{ deploy_dir }}"
        state: directory
        owner: root
        group: root
        mode: "0755"

    - name: Copy application binary
      copy:
        src: files/myapp          # локальный файл в ansible/files/
        dest: "{{ deploy_dir }}/{{ app_name }}"
        mode: "0755"

    - name: Upload systemd unit from template
      template:
        src: templates/myapp.service.j2
        dest: /etc/systemd/system/{{ app_name }}.service
        mode: "0644"

    - name: Reload systemd
      command: systemctl daemon-reload
      args:
        warn: false

    - name: Ensure app service is started and enabled
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes
```

</details>

<details>
<summary>Create simple cron job</summary>
```yaml
- name: Create simple cron job on target hosts
  hosts: utils
  become: true

  tasks:
    - name: Ensure daily backup cron exists
      cron:
        name: "daily-db-backup"
        user: root
        minute: "30"
        hour: "2"
        job: "/usr/local/bin/db-backup.sh >> /var/log/db-backup.log 2>&1"
```
</details>
