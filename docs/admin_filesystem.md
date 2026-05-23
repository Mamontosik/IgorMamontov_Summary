# Файловая система и мониторинг

## Проверка места на дисках

!!! note "Команды для анализа дискового пространства"

    **df** — использование дисков
    ```bash
    df -h          # Размеры в человекочитаемом формате
    # Filesystem      Size  Used Avail Use% Mounted on
    # /dev/sda1       100G   85G   15G  85% /
    df -i          # Количество инодов
    df -T          # Типы файловых систем
    ```

    **du** — анализ размера директорий
    ```bash
    du -sh /path                    # Размер конкретной директории
    du -sh *                        # Размер всех поддиректорий
    du -h --max-depth=1             # Иерархия с глубиной 1
    ```

    **Сценарий 1: Найти, что занимает место**
    ```bash
    du -sh /* | sort -hr | head -10
    # 50G     /var
    # 20G     /home
    # 10G     /opt
    #  5G     /usr
    #  2G     /tmp
    ```

    ---

    **Сценарий 2: Найти большие файлы**
    ```bash
    find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | head -10
    # -rwxr-xr-x 1 root root 500M /var/lib/docker/overlay2/...
    # -rw-r--r-- 1 root root 200M /var/log/nginx/access.log
    # -rwxr-xr-x 1 root root 150M /opt/java/java8.jar
    ```

    ---

    **smartctl** — SMART-информация дисков
    ```bash
    sudo smartctl -a /dev/sda   # Полная SMART-информация
    sudo smartctl -H /dev/sda   # Быстрая проверка здоровья
    ```

    **Сценарий 1: Проверить SMART-здоровье диска**
    ```bash
    sudo smartctl -H /dev/sda
    # === START OF READ SMART DATA SECTION ===
    # SMART overall-health self-assessment test result: PASSED
    sudo smartctl -a /dev/sda | grep -E "Power_On_Hours|Temperature"
    # Power_On_Hours          0x0032   096   096   000    Old_age   -       12345
    # Temperature_Celsius     0x0022   045   045   000    Old_age   -       35
    ```

    ---

    **Состояние RAID-массива**
    ```bash
    cat /proc/mdstat
    # Personalities : [raid1]
    # md0 : active raid1 sda1[1] sdb1[0]
    #       488286464 blocks super 1.2 [2/2] [UU]
    ```

!!! warning "Диск заполнен"

    1. `df -h` — проверить Use% и найти заполненный раздел
    2. `du -sh /* | sort -hr | head -10` — найти виновника
    3. `journalctl --vacuum-size=100M` — очистить systemd-логи
    4. `docker system prune -a` — очистить неиспользуемые образы и контейнеры

## Мониторинг ресурсов

!!! note "Проверка процессора, памяти и дисков"

    **top** — список процессов и системная информация
    ```bash
    top                       # Обновление в реальном времени
    top -u username           # Фильтр по пользователю
    top -p PID                # Мониторинг конкретного PID
    ```

    **Сценарий 1: Мониторинг в реальном времени (pipeline)**
    ```bash
    watch -n 1 'df -h | grep -E "^/dev"; echo "---"; ps aux --sort=-%cpu | head -5'
    # Каждую секунду обновляется вывод: использование дисков + топ-5 процессов по CPU
    ```

    ---

    **Горячие клавиши top:**
    | Клавиша | Действие |
    | --- | --- |
    | `Пробел` | Обновить вывод |
    | `M` | Сортировка по памяти |
    | `P` | Сортировка по CPU (по умолчанию) |
    | `T` | Сортировка по времени |
    | `A` | Показать max потребление ресурсов |
    | `u` | Фильтр по пользователю |
    | `k` | Завершить процесс (ввести PID) |
    | `c` | Показать полный путь процесса |
    | `n` | Изменить количество процессов |
    | `q` | Выход |

    **iotop** — мониторинг дискового I/O по процессам
    ```bash
    sudo iotop                # Интерактивный мониторинг
    sudo iotop -b             # Пакетный режим (для логов)
    sudo iotop -p PID          # Мониторинг конкретного процесса
    ```

    **Сценарий 1: Кто грузит диски**
    ```bash
    sudo iotop -b -o -d 1 | head -20
    # Total DISK READ:       0.00 B/s | Total DISK WRITE:       0.00 B/s
    # Current DISK READ:       0.00 B/s | Current DISK WRITE:     523.00 K/s
    #   TID  PRIO  USER  DISK READ  DISK WRITE  COMMAND
    #    123 be/3 root        0.00 B    523.00 K/s rsyslogd
    ```

    ---

    **iostat** — мониторинг CPU и дискового I/O
    ```bash
    iostat -x 1 5              # Расширенная статистика, обновление каждые 1 сек, 5 раз
    iostat -dx /dev/sda        # Статистика конкретного устройства
    ```

    **Сценарий 1: Анализ I/O через iostat**
    ```bash
    iostat -x 1 10 | grep -E "Device|%"
    # Device  r/s    w/s    rkB/s   wkB/s  await  %util
    # sda     2.50  10.30  128.00  528.00  2.35  15.20
    # sdb     0.00   0.00    0.00    0.00  0.00  0.00
    ```

    ---

    **htop** — улучшенный top (нужно установить)
    ```bash
    sudo apt install htop     # Установка в Debian/Ubuntu
    htop                      # Интерактивный мониторинг
    ```

!!! warning "Процесс грузит CPU"

    1. `top` → найти PID с высоким %CPU
    2. `ps -ef | grep PID` — что запущено и родительский процесс
    3. `strace -p PID` — посмотреть системные вызовы (耗资源, использовать кратко)
    4. `kill -STOP PID` — приостановить для анализа
    5. `cpulimit -p PID -l 50` — ограничить CPU до 50%

## Работа с файлами

!!! note "Управление файлами"

    **find** — поиск файлов
    ```bash
    find / -type f -size +100M         # Файлы больше 100M
    find / -name "*.log" -mtime +30    # Логи старше 30 дней
    find / -type f -perm 777            # Файлы с правами 777
    ```

    **Сценарий 1: Найти и очистить старые логи**
    ```bash
    # Найти логи старше 30 дней
    find /var/log -type f -name "*.log" -mtime +30 -ls
    # 12345  -rw-r--r--  1 root 100M /var/log/nginx/access.log.3
    # Проверить перед удалением, затем очистить:
    find /var/log -type f -name "*.log" -mtime +30 -exec truncate -s 0 {} \;
    ```

    ---

    **Сценарий 2: Массовое изменение прав**
    ```bash
    find /var/www -type f -exec chmod 644 {} \;    # Файлы: rw-r--r--
    find /var/www -type d -exec chmod 755 {} \;    # Директории: rwxr-xr-x
    chown -R www-data:www-data /var/www          # Сменить владельца
    # Проверить:
    ls -la /var/www
    # drwxr-xr-x  www-data www-data 4096 Jan 15 10:00 /var/www/html
    ```

    ---

    **Сценарий 3: Найти дубликаты по MD5**
    ```bash
    find /path -type f -exec md5sum {} \; | sort | uniq -d -w32
    # d41d8cd98f00b204e9800998ecf8427e  ./file1.txt
    # d41d8cd98f00b204e9800998ecf8427e  ./file2.txt
    ```

    ---

    **grep** — поиск в файлах
    ```bash
    grep "pattern" file              # Найти строку
    grep -r "pattern" dir/            # Рекурсивный поиск
    grep -n "pattern" file            # Показать номера строк
    grep -v "pattern" file            # Инвертированный поиск
    ```

    **Сценарий 1: Рекурсивный поиск в логах**
    ```bash
    grep -rn "ERROR" /var/log/ --include="*.log" | tail -20
    # /var/log/nginx/error.log:145:2024-01-15 ERROR: Connection timeout
    # /var/log/nginx/error.log:156:2024-01-15 ERROR: 502 Bad Gateway
    grep -rn "ERROR\|WARN" /var/log/ --include="*.log" | awk -F: '{print $1,$2}' | sort | uniq -c
    # 15 /var/log/nginx/error.log:145
    # 3 /var/log/syslog:234
    ```

    ---

    **Навигация по каталогам**
    ```bash
    pwd                       # Текущий каталог
    cd dir                    # Перейти в dir
    cd                        # Домашний каталог
    cd ..                     # На уровень выше
    cd -                      # Предыдущий каталог
    ```

    **Просмотр содержимого**
    ```bash
    ls                        # Список файлов
    ls -la                    # Форматированный список со скрытыми
    ls -lh                    # Размеры в человекочитаемом формате
    ls -lt                    # Сортировка по времени
    ```

    **Создание и удаление**
    ```bash
    mkdir dir                 # Создать директорию
    mkdir -p path/to/dir     # Создать вложенные директории
    touch file               # Создать пустой файл
    rm file                   # Удалить файл
    rm -rf dir                # Удалить директорию рекурсивно
    ```

    **Копирование и перемещение**
    ```bash
    cp file1 file2            # Копировать файл
    cp -r dir1 dir2           # Копировать директорию
    mv file1 file2            # Переместить или переименовать
    ```

    **Символические ссылки**
    ```bash
    ln -s file link           # Создать симлинк
    ls -la | grep "^l"        # Найти все симлинки
    ```

    **Просмотр содержимого файлов**
    ```bash
    cat file                  # Весь файл
    head -n 20 file           # Первые 20 строк
    tail -n 20 file          # Последние 20 строк
    tail -f file             # Следить за изменениями
    more file                 # Постраничный просмотр
    less file                 # Интерактивный просмотр (q — выход)
    ```

    **Запись в файл**
    ```bash
    cat > file                # Запись со stdin
    echo "text" > file       # Перезаписать файл
    echo "text" >> file       # Добавить в конец
    ```

!!! warning "Удалили не тот файл"

    1. `ls -la /proc/$(lsof +D / | grep filename | awk '{print $2}')/fd` — найти дескриптор
    2. `cp /proc/PID/fd/N /restore/path` — восстановить из дескриптора процесса
    3. `debugfs -R "ls -d /path" /dev/sda1` — поиск в ext4 (если файловая ext4)

## Управление процессами

!!! note "Мониторинг и управление процессами"

    **ps** — просмотр процессов
    ```bash
    ps aux                    # Все процессы
    ps -ef                    # Подробный формат
    ps aux | grep nginx       # Найти процесс
    pgrep -f nginx            # PID по имени
    pstree                    # Дерево процессов
    ```

    **Сценарий 1: Найти и убить зависший процесс**
    ```bash
    ps aux | grep -E "nginx|python" | grep -v grep
    # root      1234  0.5  0.2  45678 12345 ?        S    10:00   0:00 nginx: worker
    # user     5678  2.1  1.5  98765 45678 ?        S    10:00   5:23 python app.py
    pgrep -f "nginx worker"              # Получить PID
    kill -9 $(pgrep -f "nginx worker")   # Принудительно завершить
    ```

    ---

    **kill** — управление процессами
    ```bash
    kill PID                  # Завершить процесс
    kill -9 PID               # Принудительное завершение
    kill -STOP PID            # Приостановить
    kill -CONT PID            # Продолжить
    pkill -f nginx            # Завершить по имени
    ```

    **Сценарий 1: Безопасное завершение процесса**
    ```bash
    # Сначала попробовать SIGTERM (мягкое завершение)
    kill PID
    # Подождать и проверить
    sleep 5 && ps -p PID
    # Если не завершился — SIGKILL (принудительное)
    kill -9 PID
    ```

    ---

    **nice/renice** — приоритеты процессов
    ```bash
    nice -n 10 command        # Запустить с пониженным приоритетом
    renice -n 5 -p PID        # Изменить приоритет работающего
    top                       # Интерактивное управление
    ```

    **Сценарий 1: Ограничить CPU процесса**
    ```bash
    cpulimit -p PID -l 50                # Ограничить до 50%
    # или через nice
    renice +10 -p PID                    # Понизить приоритет (nice: -20...+20)
    ps -o pid,nice,comm -p PID
    #   PID  NI COMMAND
    #  12345  10 python
    ```

    ---

    **lsof** — мониторинг открытых файлов
    ```bash
    lsof                      # Все открытые файлы
    lsof -i                    # Открытые сетевые соединения
    lsof -p PID               # Файлы конкретного процесса
    ```

    **Сценарий 1: Что держит файл**
    ```bash
    lsof /var/log/syslog
    # COMMAND  PID   USER  FD  TYPE DEVICE SIZE/OFF NODE NAME
    # rsyslogd  123   root  3w  reg  8,1    1024    456 /var/log/syslog
    lsof -i :80                         # Процессы на порту 80
    # COMMAND   PID   USER  FD  TYPE DEVICE SIZE/OFF NODE NAME
    # nginx     123   root  6u  IPv4  7890      0t0   TCP *:http (LISTEN)
    ```

    ---

    **Фоновые процессы**
    ```bash
    command &                 # Запустить в фоне
    nohup command &           # Игнорировать сигнал HUP
    jobs                      # Список фоновых задач
    fg                        # Перевести в foreground
    bg                        # Продолжить в фоне
    ```

!!! warning "Процесс не завершается (zombie)"

    1. `ps aux | grep -w Z` — найти zombie-процессы
      `USER       PID  STAT COMMAND`
      `root      5678  Z   [defunct]`
    2. Найти родителя: `ps -o ppid= -p 5678`
    3. Перезапустить родительский процесс: `kill -HUP родитель`
    4. Если не помогает — reboot (zombie нельзя kill -9)

## Диски и файловые системы

!!! note "Работа с дисками"

    **lsblk** — список блочных устройств
    ```bash
    lsblk                     # Список блочных устройств
    # NAME        SIZE  TYPE  MOUNTPOINT
    # sda          100G  disk
    # sda1         100G  part  /
    # sdb           50G  disk
    ```

    **Сценарий 1: Найти новый диск**
    ```bash
    lsblk
    # NAME  SIZE  TYPE  MOUNTPOINT
    # sda   100G  disk
    # sdb    50G  disk     # <-- новый, неразмеченный
    fdisk -l /dev/sdb      # Проверить таблицу разделов
    ```

    ---

    **mount** — монтирование
    ```bash
    mount /dev/sdb1 /mnt      # Монтировать
    umount /mnt               # Размонтировать
    df -T                     # Типы файловых систем
    findmnt                   # Текущие монтирования
    ```

    **Сценарий 1: Создать и смонтировать новый диск**
    ```bash
    lsblk                     # Найти новый диск (sdb)
    # NAME  SIZE  TYPE  MOUNTPOINT
    # sda   100G  disk
    # sdb    50G  disk
    mkfs.ext4 /dev/sdb       # Создать файловую систему ext4
    mount /dev/sdb /mnt/data # Примонтировать
    echo "/dev/sdb /mnt/data ext4 defaults 0 2" >> /etc/fstab  # Автомонтирование
    mount -a                  # Проверить fstab
    ```

    ---

    **LVM** — логические тома
    ```bash
    pvcreate /dev/sdb         # Создать Physical Volume
    vgextend vg_name /dev/sdb # Расширить Volume Group
    lvextend -L +10G /dev/vg_name/lv_root  # Расширить Logical Volume
    resize2fs /dev/vg_name/lv_root         # Применить изменения
    ```

    **Сценарий 1: Расширить раздел (LVM)**
    ```bash
    lsblk                     # Найти PV
    # NAME        SIZE  TYPE MOUNTPOINT
    # sdb          10G   disk
    pvcreate /dev/sdb         # Создать Physical Volume
    vgdisplay                 # Найти VG name
    vgextend vg_name /dev/sdb # Расширить Volume Group
    lvextend -L +10G /dev/vg_name/lv_root  # Расширить Logical Volume
    resize2fs /dev/vg_name/lv_root         # Применить изменения
    df -h                     # Проверить
    ```

    ---

    **mkfs** — создание файловых систем
    ```bash
    mkfs.ext4 /dev/sdb1       # Создать ext4
    mkfs.xfs /dev/sdb1        # Создать xfs
    mkfs -t ext4 /dev/sdb1    # Аналогично
    ```

    **Сценарий 1: Создать XFS с меткой**
    ```bash
    mkfs.xfs -L data /dev/sdb1
    # meta-data=/dev/sdb1              isize=512    agcount=4
    # data      =                     bsize=4096   blocks=524288
    echo "LABEL=data /mnt/data xfs defaults 0 0" >> /etc/fstab
    ```

    ---

    **df -i** — проверка inodes
    ```bash
    df -i                     # Проверить использование inodes
    # Filesystem  Inodes  IUsed  IFree  IUse%  Mounted on
    # /dev/sda1   65536  65536      0  100%   /
    ```

    **Сценарий 1: Очистить inodes (если закончились)**
    ```bash
    df -i                     # Проверить использование inodes
    # Filesystem  Inodes  IUsed  IFree  IUse%  Mounted on
    # /dev/sda1   65536  65536      0  100%   /
    find /var/spool -type d | while read d; do echo "$(ls -A $d | wc -l) $d"; done | sort -rn | head -10
    # Очистка:
    find /var/spool/mail -type f -mtime +30 -delete
    ```

    ---

    **fdisk/parted** — работа с разделами
    ```bash
    fdisk -l                  # Таблица разделов
    parted -l                 # Подробная информация о разделах
    blkid                     # UUID и типы файловых систем
    ```

    **fsck** — проверка и восстановление
    ```bash
    fsck /dev/sdb1            # Проверка
    fsck -a /dev/sdb1         # Автоматическое восстановление
    e2fsck -f /dev/sdb1       # Принудительная проверка
    ```

!!! warning "Диск в readonly"


    1. `dmesg | grep -E "sd[a-z]|error"` — посмотреть ошибки ядра
    2. `mount | grep /dev/sda` — проверить флаги монтирования (ro/rw)
    3. `smartctl -H /dev/sda` — SMART-проверка (диск может умирать)
    4. `dd if=/dev/sda of=/dev/null bs=1M count=100` — проверить чтение
    5. Если диск сыпется — бекап и замена