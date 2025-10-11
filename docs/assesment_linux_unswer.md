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

#### Визуализация (Mermaid)

```mermaid
flowchart LR
  POST --> Bootloader
  Bootloader --> Kernel
  Kernel --> initramfs
  initramfs --> switch_root
  switch_root --> systemd
  systemd --> Services
  Services --> UserLogin
```

> Примечание: для рендеринга Mermaid включите поддержку в `mkdocs.yml` (например, mkdocs-mermaid2-plugin).

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