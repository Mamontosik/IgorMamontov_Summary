# Управление пользователями

## Создание и удаление пользователей

!!! note "Управление учётными записями"

    **Создание пользователей**
    ```bash
    useradd -m -s /bin/bash username    # Создать с домашней директорией
    adduser username                     # Интерактивный способ (Debian/Ubuntu)
    useradd -m -G sudo,www-data username  # Создать и добавить в группы
    ```

    **Удаление пользователей**
    ```bash
    userdel username                     # Удалить пользователя
    userdel -r username                   # Удалить с домашней директорией
    ```

    **Изменение учётной записи**
    ```bash
    usermod -l newname oldname            # Переименовать
    usermod -L username                   # Заблокировать
    usermod -U username                   # Разблокировать
    usermod -aG groupname username         # Добавить в группу
    ```

## Управление паролями

!!! note "Пароли и сроки действия"

    **Установка пароля**
    ```bash
    passwd username                       # Изменить пароль пользователя
    passwd                                # Изменить свой пароль
    ```

    **Проверка сроков действия**
    ```bash
    chage -l username                     # Показать параметры пароля
    chage -M 90 username                  # Максимум 90 дней
    chage -E 2025-12-31 username          # Срок действия до даты
    chage -W 7 username                   # Предупреждение за 7 дней
    ```

## Управление группами

!!! note "Группы пользователей"

    **Создание и удаление групп**
    ```bash
    groupadd groupname                    # Создать группу
    groupdel groupname                     # Удалить группу
    groupmod -n newname oldname           # Переименовать
    ```

    **Добавление пользователей в группу**
    ```bash
    usermod -aG groupname username         # Добавить в группу
    gpasswd -a username groupname          # Альтернативный способ
    gpasswd -d username groupname          # Удалить из группы
    ```

    **Просмотр групп**
    ```bash
    groups                                # Группы текущего пользователя
    groups username                       # Группы конкретного пользователя
    getent group                           # Все группы системы
    cat /etc/group                         # Файл групп
    ```

## Настройка sudo

!!! note "Права суперпользователя"

    **Файл sudoers**
    ```bash
    visudo                                # Редактирование /etc/sudoers
    ```

    **Примеры записей в sudoers:**
    ```
    # Полный доступ для пользователя
    username ALL=(ALL:ALL) ALL

    # Без пароля для конкретной команды
    username ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

    # Группа без пароля
    %sudo ALL=(ALL) NOPASSWD: ALL

    # Только чтение
    username ALL=(ALL) /bin/ls, /bin/cat
    ```

    **Проверка прав**
    ```bash
    sudo -l                               # Показать разрешения
    sudo -l -U username                    # Для конкретного пользователя
    sudo -k                               # Сбросить кэш пароля
    ```

## Активные пользователи

!!! note "Мониторинг входа"

    **Кто в системе**
    ```bash
    who                                   # Текущие сессии
    w                                     # Подробная информация
    whoami                                # Текущий пользователь
    id                                    # UID, GID, группы
    ```

    **История входов**
    ```bash
    last                                 # Все входы
    last username                        # Входы конкретного
    lastb                                 # Неудачные попытки
    lastlog                               # Последний вход каждого
    ```

    **Подсчёт пользователей**
    ```bash
    who | wc -l                          # Количество онлайн
    getent passwd | wc -l                # Всего пользователей
    awk -F: 'print $1' /etc/passwd       # Список имён
    ```

## Файлы учётных записей

!!! note "Системные файлы"

    | Файл | Назначение |
    | --- | --- |
    | `/etc/passwd` | Учётные записи (login, UID, GID, shell) |
    | `/etc/shadow` | Зашифрованные пароли |
    | `/etc/group` | Группы и участники |
    | `/etc/gshadow` | Защищённые данные групп |
    | `/etc/sudoers` | Права sudo |
    | `/home/` | Домашние директории |
    