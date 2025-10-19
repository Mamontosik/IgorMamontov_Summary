## Linux — структура и этапы загрузки операционной системы

### Критичные каталоги

| Путь | Назначение | Примечание |
|---|---|---|
| / | Корень файловой системы | Точка монтирования всего дерева |
| /boot | Файлы загрузчика/ядра (vmlinuz, initramfs, grub) | Должна быть доступна загрузчику |
| /etc | Конфигурация системы | Бэкапируйте конфиги перед изменениями |
| /lib, /lib64 | Библиотеки ранних утилит и ядра | Критично для загрузки |

### Приложения и пользовательские каталоги

| Путь | Назначение |
|---|---|
| /usr, /usr/bin, /usr/lib | Приложения и библиотеки пользователей |
| /usr/local | Локально установленное ПО |
| /home | Домашние директории пользователей |

### Runtime и виртуальные FS

| Путь | Назначение |
|---|---|
| /var, /var/log | Изменяемые данные и логи |
| /run | Runtime данные (PID, locks, sockets) — tmpfs |
| /tmp | Временные файлы (часто очищается при reboot) |
| /proc, /sys | Виртуальные FS (процессы и устройства) |
| /dev | Device nodes (udev) |

### Команды для быстрого просмотра

<details>
<summary>Нажмите, чтобы раскрыть команды</summary>

```bash
# Обзор точек монтирования и свободного места
df -hT
mount | column -t

# Содержимое root и критичных директорий
ls -la /boot /etc /var/log

# Информация о дисках и устройствах
lsblk -f
blkid

# Информация ядра/памяти
cat /proc/cpuinfo
cat /proc/meminfo
```

</details>

### Этапы загрузки Linux — краткая схема

| Этап | Что происходит | Диагностика |
|---|---|---|
| 1. Power on / POST | Инициализация BIOS/UEFI | Проверить настройки прошивки, secure boot |
| 2. Boot device selection | Выбор загрузочного устройства (MBR/GPT) | `efibootmgr -v` (UEFI) |
| 3. Bootloader (GRUB) | Работа GRUB, выбор ядра | Проверить `/boot/grub/grub.cfg` |
| 4. Kernel + initramfs | Загрузка ядра и initramfs, монтирование ранних модулей | `cat /proc/cmdline`, `dmesg` |
| 5. switch_root / pivot_root | Переключение на реальный root | `mount` после загрузки |
| 6. init (PID 1 — systemd) | Запуск юнитов и таргетов | `systemctl --failed`, `journalctl -b` |
| 7. Services | Демоны (sshd, cron, network) | `systemctl list-units --state=failed` |
| 8. User login | getty / display manager | `systemctl status graphical.target` |
| 9. Runtime | Стабильная работа, таймеры, cron | `top`, `journalctl -f` |

### Полезные команды диагностики загрузки

<details>
<summary>Нажмите, чтобы раскрыть команды</summary>

```bash
uname -a
cat /proc/cmdline

# Журнал текущей загрузки
journalctl -b
journalctl -b -u sshd.service

# Последние ошибки
journalctl -p err -b

# Статус systemd
systemctl --failed
systemctl list-dependencies multi-user.target

# Проверка GRUB/UEFI
ls /boot
efibootmgr -v

# Для проблем с root/инициализацией
dmesg | less
cat /proc/mounts
mount
```

</details>

## Список команды для работы в UNIX Terminal

| Команда | Описание | Параметры | Пример |
|---|---|---|---|
| `ls` | Список файлов/папок | `-l` (long — подробный список); `-a` (all — показать скрытые); `-h` (human — размеры в удобном виде) | `ls -la -h /var/www` |
| `cd` | Перемещение по каталогам | (без параметров) — путь или `-` (предыдущая директория) | `cd /etc/nginx` / `cd -` |
| `cp` | Копирование файлов/каталогов | `-r` (recursive — рекурсивно для директорий); `-p` (preserve — сохранить права/время) | `cp -r -p src/ dest/` |
| `mv` | Перемещение/переименование | (без параметров) | `mv old.conf new.conf` |
| `rm` | Удаление файлов/каталогов | `-r` (recursive); `-f` (force — без подтверждения) — вместе `-rf` очень опасно | `rm -rf /tmp/build` |
| `find` | Поиск файлов по условиям | `-name` (по имени); `-type` (f/d); `-mtime` (по возрасту) | `find /var -name "*.log" -mtime +7` |
| `grep` | Поиск по тексту | `-R` (recursive); `-n` (show line numbers); `-E` (extended regex) | `grep -R -n "ERROR" /var/log` |
| `awk` | Текстовая обработка колонок | (скрипт/выражение) — `{print $N}` | `awk '{print $5}' file` |
| `sed` | Потоковая замена/редактирование | `-i` (in-place — заменить в файле); `-E` (extended regex) | `sed -i 's/old/new/g' file` |
| `ps` / `top` / `htop` | Процессы/ресурсы | `ps aux` (все процессы, подробный вывод); `ps -ef` (альтернативный формат) | `ps aux | grep nginx` / `top` / `htop` |
| `systemctl` | Управление systemd сервисами | `status|start|stop|restart|enable|disable` | `systemctl restart nginx` |
| `journalctl` | Просмотр системного журнала | `-u <service>` (фильтр по юниту); `-b` (текущий boot); `-f` (follow) | `journalctl -u nginx -f` |
| `ssh` | Подключение к удалённым хостам | `-i <key>` (ключ); `-p <port>` (порт); `-o` (опции) | `ssh -i ~/.ssh/id_rsa -p 2222 user@host` |
| `scp` / `rsync` | Копирование файлов между хостами | `scp -i <key>`; `rsync -avz` (`-a` archive, `-v` verbose, `-z` compress); `--delete` (удалять лишние файлы на приёмнике) | `rsync -avz --delete ./dist user@host:/srv/` |
| `curl` / `wget` | HTTP‑запросы / скачивание | `-I` (HEAD запрос); `-s` (silent); `-L` (follow redirects); `-o` (output file) | `curl -I -s -L https://example.com` |
| `tar` | Архивация/распаковка | `-c` (create); `-x` (extract); `-z` (gzip); `-f` (file) | `tar czf backup.tgz /etc` / `tar xzf file.tgz` |
| `chmod` / `chown` | Права и владелец | `chmod u/g/o` или `chmod -R` (рекурсивно); `chown user:group -R` | `chown -R www:www /var/www` / `chmod 644 file` |
| `df` / `du` | Диск/занятость | `df -h` (human); `du -sh` (суммарный размер) | `df -h` / `du -sh /var/log` |
| `ip` / `ss` | Сеть: адреса, сокеты, маршруты | `ip addr` (показ адресов); `ip route` (маршруты); `ss -tuna` (`-t` TCP, `-u` UDP, `-n` numeric, `-a` all) | `ip addr show` / `ss -tuna | grep LISTEN` |
