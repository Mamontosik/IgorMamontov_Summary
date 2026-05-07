# Правила работы с bash

## Зачем нужен bash?

!!! note "Назначение bash"

    **Интерпретатор командной строки (оболочка)**
    <br>
    Bash (Bourne Again Shell) — это командная оболочка Unix, которая позволяет взаимодействовать с операционной системой через текстовые команды.

    **Автоматизация рутинных задач**
    <br>
    Скрипты bash позволяют автоматизировать бекапы, мониторинг, очистку логов и другие повторяющиеся операции.

    **Управление серверами без GUI**
    <br>
    Через SSH и bash-скрипты можно управлять удалёнными серверами, на которых не установлен графический интерфейс.

    **CI/CD пайплайны**
    <br>
    Bash используется в GitLab CI, GitHub Actions, Jenkins для написания сценариев сборки, тестирования и деплоя.

    **DevOps инструментарий**
    <br>
    Работа с Docker, Kubernetes, Ansible часто требует написания bash-скриптов для оркестрации процессов.

    **Обработка логов и данных**
    <br>
    Команды grep, awk, sed позволяют быстро анализировать большие объёмы текстовых данных.

    **Кроссплатформенность**
    <br>
    Bash доступен на Linux, macOS, и через WSL (Windows Subsystem for Linux) на Windows.

## Популярные команды

!!! note "Навигация и файлы"

    **cd, pwd, ls, find** — перемещение по файловой системе и поиск файлов
    ```bash
    cd /path/to/dir [Перейти в каталог]
    pwd [Показать текущий каталог]
    ls -la [Список файлов с деталями]
    find / -name "file.txt" [Поиск файла]
    ```

    **cp, mv, rm, mkdir, touch** — управление файлами и каталогами
    ```bash
    cp file1 file2 [Копирование]
    mv file1 file2 [Перемещение или переименование]
    rm file [Удаление файла]
    mkdir dir [Создание каталога]
    touch file [Создание пустого файла]
    ```

    **cat, head, tail, less** — просмотр содержимого файлов
    ```bash
    cat file [Вывод всего файла]
    head -n 10 file [Первые 10 строк]
    tail -n 10 file [Последние 10 строк]
    tail -f /var/log/syslog [Просмотр лога в реальном времени]
    ```

!!! note "Текст и данные"

    **grep, awk, sed** — поиск и обработка текста
    ```bash
    grep "error" file.log [Поиск строк с error]
    awk '{print $1}' file [Вывод первого поля]
    sed 's/old/new/g' file [Замена текста]
    ```

    **sort, uniq, wc, cut** — анализ данных
    ```bash
    sort file [Сортировка строк]
    uniq file [Удаление дубликатов]
    wc -l file [Подсчёт строк]
    cut -d: -f1 /etc/passwd [Извлечение первого поля]
    ```

!!! note "Права и пользователи"

    **chmod, chown, chgrp** — управление правами доступа
    ```bash
    chmod 755 script.sh [Установка прав на выполнение]
    chown user:group file [Смена владельца]
    ```

    **sudo, su** — переключение пользователей
    ```bash
    sudo command [Выполнение от имени root]
    su - username [Переключение пользователя]
    ```

!!! note "Сеть и соединения"

    **curl, wget** — HTTP-запросы
    ```bash
    curl -I https://example.com [Проверка заголовков]
    wget https://example.com/file.zip [Скачивание файла]
    ```

    **ssh, scp, rsync** — удалённый доступ и передача файлов
    ```bash
    ssh user@server [Подключение по SSH]
    scp file user@server:/path [Копирование файла]
    rsync -avz /local user@server:/remote [Синхронизация]
    ```

    **netstat, ss, ping, traceroute** — диагностика сети
    ```bash
    ss -tuln [Список слушающих портов]
    ping -c 4 google.com [Проверка доступности]
    traceroute google.com [Трассировка маршрута]
    ```

!!! note "Процессы и система"

    **ps, top, htop, kill** — управление процессами
    ```bash
    ps aux [Список всех процессов]
    top [Мониторинг в реальном времени]
    kill -9 PID [Принудительное завершение]
    ```

    **df, du, free, uname** — системная информация
    ```bash
    df -h [Место на дисках]
    du -sh /dir [Размер каталога]
    free -h [Использование памяти]
    uname -a [Информация о системе]
    ```

    **tar, gzip, zip** — архивация
    ```bash
    tar -czf archive.tar.gz /dir [Создание архива]
    gzip file [Сжатие файла]
    ```

!!! note "Перенаправление и цепочки"

    **>, >>, 2>, |** — перенаправление вывода
    ```bash
    command > file [Запись в файл (перезапись)]
    command >> file [Запись в файл (добавление)]
    command 2> error.log [Перенаправление ошибок]
    command1 | command2 [Передача вывода между командами]
    ```

    **&&, ||, ;** — цепочки команд
    ```bash
    cmd1 && cmd2 [cmd2 выполнится, если cmd1 успешна]
    cmd1 || cmd2 [cmd2 выполнится, если cmd1 завершилась с ошибкой]
    cmd1; cmd2 [Последовательное выполнение]
    ```

## Скрипты bash

???+ info "Основы написания скриптов"

    ```bash
    #!/bin/bash
    # Строка shebang — указывает системе, какой интерпретатор использовать

    # Комментарии начинаются с #
    # Переменные объявляются без пробелов: VAR=value
    # Обращение к переменной через $: $VAR или ${VAR}
    # Проверка кода возврата: $? (0 = успех, не 0 = ошибка)
    # Условия: if [ condition ]; then ... fi
    # Циклы: for var in list; do ... done
    # Функции: function_name() { ... }
    ```

!!! note "Скрипт #8: Мониторинг SSL-сертификатов"

    Проверяет срок действия SSL-сертификата и предупреждает за 30 дней до истечения.

    ```bash
    #!/bin/bash
    # Скрипт для мониторинга срока действия SSL-сертификата
    # Автор: DevOps Team
    # Дата: 2024

    # Объявляем переменные
    DOMAIN="example.com"              # Домен для проверки
    PORT=443                          # Порт HTTPS (стандартный 443)
    DAYS_THRESHOLD=30                 # Пороговое значение в днях для предупреждения
    ALERT_DAYS=7                     # Критический порог (осталось менее 7 дней)

    # Получаем дату истечения сертификата через openssl
    # s_client подключается к серверу, x509 извлекает данные сертификата
    # enddate показывает дату истечения, cut извлекает саму дату
    EXPIRY_DATE=$(echo | openssl s_client -servername "$DOMAIN" -connect "$DOMAIN:$PORT" 2>/dev/null \
        | openssl x509 -noout -enddate 2>/dev/null \
        | cut -d= -f2)

    # Проверяем, удалось ли получить дату сертификата
    if [ -z "$EXPIRY_DATE" ]; then
        echo "ОШИБКА: Не удалось получить данные сертификата для $DOMAIN"
        exit 1
    fi

    # Преобразуем дату истечения в формат секунд с начала эпохи (Unix timestamp)
    EXPIRY_EPOCH=$(date -d "$EXPIRY_DATE" +%s 2>/dev/null || date -j -f "%b %d %H:%M:%S %Y %Z" "$EXPIRY_DATE" +%s)
    CURRENT_EPOCH=$(date +%s)         # Текущее время в секундах
    SECONDS_IN_DAY=86400              # Количество секунд в сутках

    # Вычисляем количество дней до истечения сертификата
    DAYS_LEFT=$(( ($EXPIRY_EPOCH - $CURRENT_EPOCH) / $SECONDS_IN_DAY ))

    # Логируем информацию о сертификате
    echo "=========================================="
    echo "Мониторинг SSL-сертификата для $DOMAIN"
    echo "Дата истечения: $EXPIRY_DATE"
    echo "Осталось дней: $DAYS_LEFT"
    echo "=========================================="

    # Проверяем условия и выводим соответствующие сообщения
    if [ "$DAYS_LEFT" -le 0 ]; then
        # Сертификат уже истёк
        echo "КРИТИЧЕСКАЯ ОШИБКА: Сертификат для $DOMAIN УЖЕ ИСТЁК!"
        echo "Необходимо СРОЧНО обновить сертификат!"
        exit 2
    elif [ "$DAYS_LEFT" -le "$ALERT_DAYS" ]; then
        # Осталось менее 7 дней
        echo "ВНИМАНИЕ: Сертификат истекает через $DAYS_LEFT дней!"
        echo "Рекомендуется срочно продлить сертификат."
        exit 1
    elif [ "$DAYS_LEFT" -le "$DAYS_THRESHOLD" ]; then
        # Осталось менее 30 дней
        echo "ПРЕДУПРЕЖДЕНИЕ: Сертификат истекает через $DAYS_LEFT дней."
        echo "Не забудьте продлить сертификат."
        exit 0
    else
        # Сертификат в порядке
        echo "OK: Сертификат действителен ещё $DAYS_LEFT дней."
        exit 0
    fi
    ```

!!! note "Скрипт #9: Проверка доступности API"

    Проверяет доступность API эндпоинта и логирует результат.

    ```bash
    #!/bin/bash
    # Скрипт для проверки доступности API и логирования результатов
    # Использует curl для HTTP-запросов и проверяет коды ответов

    # Настройки API
    API_URL="https://api.example.com/health"  # URL для проверки
    METHOD="GET"                              # HTTP-метод (GET, POST, PUT, DELETE)
    TIMEOUT=10                                # Таймаут запроса в секундах
    RETRY_COUNT=3                             # Количество попыток при неудаче
    RETRY_DELAY=5                             # Задержка между попытками (секунды)

    # Настройки уведомлений
    LOG_FILE="/var/log/api_monitor.log"        # Файл для логирования
    SLACK_WEBHOOK=""                           # Webhook для Slack (опционально)
    EMAIL_ALERT=""                             # Email для уведомлений (опционально)

    # Функция для логирования с временной меткой
    log_message() {
        local LEVEL="$1"                       # Уровень: INFO, WARNING, ERROR
        local MESSAGE="$2"                     # Текст сообщения
        local TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
        echo "[$TIMESTAMP] [$LEVEL] $MESSAGE" | tee -a "$LOG_FILE"
    }

    # Функция для отправки уведомления в Slack
    send_slack_alert() {
        local MESSAGE="$1"
        if [ -n "$SLACK_WEBHOOK" ]; then
            curl -s -X POST -H 'Content-type: application/json' \
                --data "{\"text\":\"API Alert: $MESSAGE\"}" \
                "$SLACK_WEBHOOK" > /dev/null 2>&1
        fi
    }

    # Функция проверки API с повторными попытками
    check_api_with_retry() {
        local ATTEMPT=1                        # Счётчик попыток
        local SUCCESS=0                        # Флаг успешного выполнения

        while [ "$ATTEMPT" -le "$RETRY_COUNT" ]; do
            log_message "INFO" "Попытка $ATTEMPT/$RETRY_COUNT: Проверка $API_URL"

            # Выполняем HTTP-запрос с помощью curl
            # -s: тихий режим (не показывать прогресс)
            # -o: сохранить ответ в файл (или /dev/null)
            # -w: формат вывода (HTTP-код)
            # -X: HTTP-метод
            # --connect-timeout: таймаут соединения
            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
                -X "$METHOD" \
                --connect-timeout "$TIMEOUT" \
                "$API_URL" 2>/dev/null)

            # Проверяем код ответа
            if [ "$HTTP_CODE" = "200" ] || [ "$HTTP_CODE" = "201" ] || [ "$HTTP_CODE" = "204" ]; then
                log_message "INFO" "API доступен. HTTP-код: $HTTP_CODE"
                SUCCESS=1
                break
            else
                log_message "WARNING" "Попытка $ATTEMPT не удалась. HTTP-код: $HTTP_CODE"
                if [ "$ATTEMPT" -lt "$RETRY_COUNT" ]; then
                    log_message "INFO" "Ожидание $RETRY_DELAY секунд перед следующей попыткой..."
                    sleep "$RETRY_DELAY"
                fi
            fi

            ATTEMPT=$((ATTEMPT + 1))
        done

        # Если все попытки не удались
        if [ "$SUCCESS" -eq 0 ]; then
            log_message "ERROR" "API недоступен после $RETRY_COUNT попыток!"
            send_slack_alert "API $API_URL недоступен! HTTP-код: $HTTP_CODE"
            return 1
        fi

        return 0
    }

    # Основная логика скрипта
    log_message "INFO" "=========================================="
    log_message "INFO" "Запуск мониторинга API: $API_URL"
    log_message "INFO" "=========================================="

    # Проверяем доступность API
    if check_api_with_retry; then
        log_message "INFO" "Проверка завершена успешно."
        exit 0
    else
        log_message "ERROR" "Проверка завершена с ошибкой."
        exit 1
    fi
    ```

!!! note "Скрипт #11: Аудит открытых портов"

    Сканирует открытые порты и сравнивает с эталонным списком.

    ```bash
    #!/bin/bash
    # Скрипт для аудита открытых портов на сервере
    # Сравнивает текущие открытые порты с эталонным списком

    # Настройки
    REPORT_FILE="/var/log/port_audit_$(date +%Y%m%d_%H%M%S).log"  # Файл отчёта
    ETHALON_FILE="/etc/security/etalon_ports.txt"                  # Эталонный список портов
    ALLOWED_SERVICES="22:SSH,80:HTTP,443:HTTPS,3306:MySQL"        # Разрешённые сервисы (port:name)

    # Функция логирования
    log() {
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$REPORT_FILE"
    }

    # Функция получения списка открытых портов
    get_open_ports() {
        # Используем ss (более современная замена netstat)
        # -tuln: TCP, UDP, Listening, Numeric
        # awk извлекает номера портов
        ss -tuln | awk '/LISTEN/ {print $5}' | sed 's/.*://' | sort -n | uniq
    }

    # Функция проверки, разрешён ли порт
    is_port_allowed() {
        local PORT="$1"
        # Проверяем, есть ли порт в списке разрешённых сервисов
        echo "$ALLOWED_SERVICES" | grep -q "$PORT:"
        return $?
    }

    # Функция получения имени сервиса по порту
    get_service_name() {
        local PORT="$1"
        echo "$ALLOWED_SERVICES" | grep "$PORT:" | cut -d: -f2
    }

    # Начало аудита
    log "=========================================="
    log "АУДИТ ОТКРЫТЫХ ПОРТОВ"
    log "Хост: $(hostname)"
    log "Дата: $(date)"
    log "=========================================="

    # Получаем список текущих открытых портов
    log ""
    log "Текущие открытые порты:"
    OPEN_PORTS=$(get_open_ports)
    echo "$OPEN_PORTS" | while read -r PORT; do
        if [ -n "$PORT" ]; then
            SERVICE=$(get_service_name "$PORT")
            if [ -n "$SERVICE" ]; then
                log "  Порт $PORT ($SERVICE) — РАЗРЕШЁН"
            else
                log "  Порт $PORT — НЕИЗВЕСТНЫЙ СЕРВИС (возможно, требует проверки)"
            fi
        fi
    done

    # Сравнение с эталонным файлом, если он существует
    log ""
    log "Проверка по эталонному списку:"
    if [ -f "$ETHALON_FILE" ]; then
        log "  Эталонный файл: $ETHALON_FILE"
        # Читаем эталонный список и проверяем каждый порт
        while IFS= read -r ETALON_PORT; do
            if [ -n "$ETALON_PORT" ] && ! echo "$OPEN_PORTS" | grep -q "^$ETALON_PORT$"; then
                log "  ВНИМАНИЕ: Порт $ETALON_PORT из эталона НЕ открыт!"
            fi
        done < "$ETHALON_FILE"
        log "  Проверка по эталону завершена."
    else
        log "  Эталонный файл не найден: $ETHALON_FILE"
        log "  Создайте файл со списком портов (по одному на строку)."
    fi

    # Проверка на неожиданно открытые порты
    log ""
    log "Поиск неожиданно открытых портов:"
    UNEXPECTED=0
    echo "$OPEN_PORTS" | while read -r PORT; do
        if [ -n "$PORT" ] && ! echo "$ALLOWED_SERVICES" | grep -q "$PORT:"; then
            log "  ОБНАРУЖЕНО: Порт $PORT открыт, но не в списке разрешённых!"
            UNEXPECTED=1
        fi
    done

    if [ "$UNEXPECTED" -eq 0 ]; then
        log "  Неожиданных портов не обнаружено."
    fi

    log ""
    log "=========================================="
    log "Аудит завершён. Отчёт: $REPORT_FILE"
    log "=========================================="

    exit 0
    ```

!!! note "Скрипт #15: Очистка старых Docker образов"

    Удаляет неиспользуемые Docker образы, контейнеры и volumes.

    ```bash
    #!/bin/bash
    # Скрипт для очистки неиспользуемых Docker ресурсов
    # Удаляет: остановленные контейнеры, неиспользуемые образы, volumes, networks

    # Настройки
    DRY_RUN=false                           # true = только показать, что будет удалено
    KEEP_LATEST=3                           # Сколько последних образов каждого репозитория оставить
    LOG_FILE="/var/log/docker_cleanup.log"   # Файл лога
    ALERT_DISK_USAGE=80                     # Порог использования диска (%) для принудительной очистки

    # Функция логирования
    log() {
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
    }

    # Проверка, установлен ли Docker
    if ! command -v docker &> /dev/null; then
        log "ОШИБКА: Docker не установлен или недоступен!"
        exit 1
    fi

    # Проверка прав доступа к Docker
    if ! docker info > /dev/null 2>&1; then
        log "ОШИБКА: Нет прав доступа к Docker daemon (нужен sudo?)"
        exit 1
    fi

    # Функция проверки использования диска
    check_disk_usage() {
        # Получаем использование диска для корневого раздела
        DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
        log "Текущее использование диска: ${DISK_USAGE}%"
        if [ "$DISK_USAGE" -ge "$ALERT_DISK_USAGE" ]; then
            log "ВНИМАНИЕ: Диск заполнен более чем на ${ALERT_DISK_USAGE}%!"
            return 0  # Требуется очистка
        fi
        return 1
    }

    # Функция удаления остановленных контейнеров
    cleanup_containers() {
        log ""
        log "--- Очистка остановленных контейнеров ---"

        # Получаем список остановленных контейнеров
        STOPPED=$(docker ps -aq -f status=exited -f status=created 2>/dev/null)

        if [ -z "$STOPPED" ]; then
            log "Остановленных контейнеров не найдено."
            return 0
        fi

        log "Найдено остановленных контейнеров: $(echo "$STOPPED" | wc -l)"

        if [ "$DRY_RUN" = true ]; then
            log "[DRY RUN] Будут удалены контейнеры:"
            docker ps -a -f status=exited -f status=created --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}"
        else
            log "Удаление остановленных контейнеров..."
            docker rm $STOPPED 2>/dev/null && log "Контейнеры удалены." || log "Ошибка при удалении контейнеров."
        fi
    }

    # Функция очистки неиспользуемых образов
    cleanup_images() {
        log ""
        log "--- Очистка неиспользуемых образов ---"

        # Получаем список образов без тегов (dangling)
        DANGLING=$(docker images -f "dangling=true" -q 2>/dev/null)

        if [ -n "$DANGLING" ]; then
            log "Найдено неиспользуемых (dangling) образов: $(echo "$DANGLING" | wc -l)"
            if [ "$DRY_RUN" = true ]; then
                log "[DRY RUN] Будут удалены dangling образы"
                docker images -f "dangling=true"
            else
                docker rmi $DANGLING 2>/dev/null && log "Dangling образы удалены." || log "Ошибка при удалении dangling образов."
            fi
        else
            log "Dangling образы не найдены."
        fi

        # Удаление старых образов (оставляем только KEEP_LATEST последних версий каждого репозитория)
        log ""
        log "Проверка образов для очистки (оставляем $KEEP_LATEST последних)..."

        # Получаем список уникальных репозиториев
        REPOS=$(docker images --format "{{.Repository}}" | sort -u | grep -v "^$")

        echo "$REPOS" | while read -r REPO; do
            if [ -n "$REPO" ]; then
                log "Обработка репозитория: $REPO"
                # Получаем образы этого репозитория, отсортированные по дате создания (новые сверху)
                IMAGES=$(docker images "$REPO" --format "{{.ID}}" --sort=created)

                # Пропускаем первые KEEP_LATEST образов
                COUNTER=0
                echo "$IMAGES" | while read -r IMAGE_ID; do
                    COUNTER=$((COUNTER + 1))
                    if [ "$COUNTER" -gt "$KEEP_LATEST" ]; then
                        # Проверяем, используется ли образ каким-либо контейнером
                        IN_USE=$(docker ps -a --filter "ancestor=$IMAGE_ID" -q)
                        if [ -z "$IN_USE" ]; then
                            if [ "$DRY_RUN" = true ]; then
                                log "[DRY RUN] Будет удалён образ: $IMAGE_ID"
                            else
                                docker rmi "$IMAGE_ID" >/dev/null 2>&1 && log "Удалён образ: $IMAGE_ID" || true
                            fi
                        fi
                    fi
                done
            fi
        done
    }

    # Функция очистки volumes
    cleanup_volumes() {
        log ""
        log "--- Очистка неиспользуемых volumes ---"

        # Получаем список volumes, не используемых ни одним контейнером
        UNUSED_VOLUMES=$(docker volume ls -q -f "dangling=true" 2>/dev/null)

        if [ -z "$UNUSED_VOLUMES" ]; then
            log "Неиспользуемых volumes не найдено."
            return 0
        fi

        log "Найдено неиспользуемых volumes: $(echo "$UNUSED_VOLUMES" | wc -l)"

        if [ "$DRY_RUN" = true ]; then
            log "[DRY RUN] Будут удалены volumes:"
            docker volume ls -f "dangling=true"
        else
            docker volume rm $UNUSED_VOLUMES 2>/dev/null && log "Volumes удалены." || log "Ошибка при удалении volumes."
        fi
    }

    # Функция очистки networks
    cleanup_networks() {
        log ""
        log "--- Очистка неиспользуемых networks ---"

        # Удаляем неиспользуемые networks (кроме стандартных)
        if [ "$DRY_RUN" = true ]; then
            log "[DRY RUN] Будет выполнено: docker network prune -f"
        else
            docker network prune -f >/dev/null 2>&1 && log "Неиспользуемые networks удалены." || log "Ошибка при очистке networks."
        fi
    }

    # Функция вывода статистики Docker
    show_docker_stats() {
        log ""
        log "--- Статистика Docker до очистки ---"
        log "Образов: $(docker images -q | wc -l)"
        log "Контейнеров (всего): $(docker ps -aq | wc -l)"
        log "Контейнеров (запущено): $(docker ps -q | wc -l)"
        log "Volumes: $(docker volume ls -q | wc -l)"
        log "Networks: $(docker network ls -q | wc -l)"

        # Использование диска Docker
        log ""
        log "Использование диска Docker:"
        docker system df 2>/dev/null | while read -r LINE; do
            log "  $LINE"
        done
    }

    # Главная функция
    main() {
        log "=========================================="
        log "ЗАПУСК ОЧИСТКИ DOCKER"
        log "Хост: $(hostname)"
        log "Дата: $(date)"
        if [ "$DRY_RUN" = true ]; then
            log "Режим: DRY RUN (изменения не будут применены)"
        fi
        log "=========================================="

        # Показываем статистику до очистки
        show_docker_stats

        # Проверяем, нужна ли очистка
        if check_disk_usage || [ "$DRY_RUN" = false ]; then
            # Выполняем очистку
            cleanup_containers
            cleanup_images
            cleanup_volumes
            cleanup_networks

            log ""
            log "--- Статистика после очистки ---"
            show_docker_stats
        else
            log "Очистка не требуется (диск заполнен менее чем на ${ALERT_DISK_USAGE}%)."
        fi

        log ""
        log "=========================================="
        log "ОЧИСТКА ЗАВЕРШЕНА"
        log "=========================================="
    }

    # Обработка аргументов командной строки
    while [[ $# -gt 0 ]]; do
        case $1 in
            --dry-run)
                DRY_RUN=true
                shift
                ;;
            --keep-latest=*)
                KEEP_LATEST="${1#*=}"
                shift
                ;;
            --help|-h)
                echo "Использование: $0 [--dry-run] [--keep-latest=N]"
                echo "  --dry-run          Показать, что будет удалено, без фактического удаления"
                echo "  --keep-latest=N    Оставить N последних образов каждого репозитория (по умолчанию: 3)"
                exit 0
                ;;
            *)
                log "Неизвестный аргумент: $1"
                exit 1
                ;;
        esac
    done

    # Запуск
    main
    ```

!!! note "Скрипт #19: Мониторинг задержек сети"

    Измеряет задержки до указанных хостов и логирует результаты.

    ```bash
    #!/bin/bash
    # Скрипт для мониторинга задержек сети (latency) до указанных хостов
    # Использует ping для измерения времени отклика

    # Настройки
    HOSTS=("8.8.8.8" "1.1.1.1" "google.com" "youtube.com")  # Хосты для проверки
    PING_COUNT=5                                             # Количество ping-запросов на хост
    TIMEOUT=5                                                # Таймаут на каждый ping (секунды)
    LATENCY_THRESHOLD=100                                     # Порог задержки в мс (предупреждение)
    CRITICAL_LATENCY=200                                      # Критическая задержка в мс
    LOG_FILE="/var/log/network_latency.log"                   # Файл лога
    REPORT_INTERVAL=60                                        # Интервал между проверками (секунды)

    # Функция логирования
    log() {
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
    }

    # Функция проверки задержки до одного хоста
    check_latency() {
        local HOST="$1"
        local RESULT=""

        # Выполняем ping и извлекаем статистику
        # -c: количество пакетов
        # -W: таймаут ожидания ответа
        # Вывод grep ищет строку с rtt (round-trip time)
        PING_OUTPUT=$(ping -c "$PING_COUNT" -W "$TIMEOUT" "$HOST" 2>/dev/null)

        # Проверяем, успешен ли ping
        if [ $? -ne 0 ]; then
            echo "ERROR: Нет ответа от $HOST"
            return 1
        fi

        # Извлекаем среднее время задержки (rtt)
        # Формат вывода: rtt min/avg/max/mdev = 10.123/15.456/20.789/2.345
        AVG_LATENCY=$(echo "$PING_OUTPUT" | grep 'rtt' | awk -F'/' '{print $5}' | awk '{print $1}')

        if [ -z "$AVG_LATENCY" ]; then
            echo "ERROR: Не удалось получить задержку для $HOST"
            return 1
        fi

        # Возвращаем задержку
        echo "$AVG_LATENCY"
        return 0
    }

    # Функция анализа задержки и вывода статуса
    analyze_latency() {
        local HOST="$1"
        local LATENCY="$2"

        # Проверяем, является ли задержка числом
        if ! [[ "$LATENCY" =~ ^[0-9.]+$ ]]; then
            log "ERROR: Нет ответа от $HOST"
            return 1
        fi

        # Сравниваем с пороговыми значениями
        # Используем bc для сравнения чисел с плавающей точкой
        IS_CRITICAL=$(echo "$LATENCY > $CRITICAL_LATENCY" | bc -l 2>/dev/null)
        IS_HIGH=$(echo "$LATENCY > $LATENCY_THRESHOLD" | bc -l 2>/dev/null)

        if [ "$IS_CRITICAL" -eq 1 ]; then
            log "КРИТИЧНО: $HOST — задержка ${LATENCY}ms (превышает $CRITICAL_LATENCY ms)!"
            return 2
        elif [ "$IS_HIGH" -eq 1 ]; then
            log "ВНИМАНИЕ: $HOST — задержка ${LATENCY}ms (превышает $LATENCY_THRESHOLD ms)"
            return 1
        else
            log "OK: $HOST — задержка ${LATENCY}ms"
            return 0
        fi
    }

    # Функция генерации отчёта
    generate_report() {
        log "=========================================="
        log "МОНИТОРИНГ СЕТЕВЫХ ЗАДЕРЖЕК"
        log "Хост: $(hostname)"
        log "Дата: $(date)"
        log "=========================================="

        local TOTAL_HOSTS=${#HOSTS[@]}
        local SUCCESS_COUNT=0
        local WARNING_COUNT=0
        local ERROR_COUNT=0

        # Проверяем каждый хост
        for HOST in "${HOSTS[@]}"; do
            log ""
            log "Проверка хоста: $HOST"

            LATENCY=$(check_latency "$HOST")
            STATUS=$?

            if [ $STATUS -eq 0 ]; then
                # Успешная проверка
                analyze_latency "$HOST" "$LATENCY"
                STATUS=$?
                if [ $STATUS -eq 0 ]; then
                    SUCCESS_COUNT=$((SUCCESS_COUNT + 1))
                elif [ $STATUS -eq 1 ]; then
                    WARNING_COUNT=$((WARNING_COUNT + 1))
                fi
            else
                ERROR_COUNT=$((ERROR_COUNT + 1))
            fi
        done

        # Итоговая статистика
        log ""
        log "=========================================="
        log "ИТОГИ:"
        log "  Всего хостов: $TOTAL_HOSTS"
        log "  Успешно: $SUCCESS_COUNT"
        log "  Предупреждений: $WARNING_COUNT"
        log "  Ошибок: $ERROR_COUNT"
        log "=========================================="

        # Возвращаем код в зависимости от результатов
        if [ "$ERROR_COUNT" -gt 0 ]; then
            return 2
        elif [ "$WARNING_COUNT" -gt 0 ]; then
            return 1
        fi
        return 0
    }

    # Главная функция (с возможностью непрерывного мониторинга)
    main() {
        case "$1" in
            once)
                # Однократная проверка
                generate_report
                ;;
            watch)
                # Непрерывный мониторинг
                log "Запуск непрерывного мониторинга (интервал: $REPORT_INTERVAL сек)"
                while true; do
                    generate_report
                    sleep "$REPORT_INTERVAL"
                done
                ;;
            *)
                # По умолчанию — однократная проверка
                generate_report
                ;;
        esac
    }

    # Запуск
    main "$@"
    ```

!!! note "Скрипт #21: Деплой с откатом"

    Скрипт деплоя приложения с возможностью автоматического отката при ошибке.

    ```bash
    #!/bin/bash
    # Скрипт деплоя приложения с поддержкой отката (rollback)
    # В случае ошибки автоматически возвращает предыдущую версию

    # Настройки приложения
    APP_NAME="myapp"                                     # Имя приложения
    DEPLOY_DIR="/opt/${APP_NAME}"                        # Директория деплоя
    BACKUP_DIR="/opt/${APP_NAME}/backups"                # Директория бэкапов
    CURRENT_LINK="${DEPLOY_DIR}/current"                 # Симлинк на текущую версию
    NEW_VERSION="$1"                                     # Новая версия (передаётся аргументом)

    # Настройки сервиса
    SERVICE_MANAGER="systemd"                            # systemd или supervisor
    SERVICE_NAME="${APP_NAME}"                           # Имя сервиса

    # Настройки уведомлений
    SLACK_WEBHOOK=""                                     # Webhook для Slack (опционально)
    EMAIL_ALERT=""                                       # Email для уведомлений

    # Настройки проверки здоровья (health check)
    HEALTH_CHECK_URL="http://localhost:8080/health"      # URL для проверки
    HEALTH_CHECK_TIMEOUT=60                              # Таймаут ожидания готовности (сек)
    HEALTH_CHECK_INTERVAL=5                              # Интервал между проверками

    # Функция логирования
    log() {
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] [DEPLOY] $1"
    }

    # Функция отправки уведомлений
    send_notification() {
        local MESSAGE="$1"
        local STATUS="$2"  # SUCCESS или FAILURE

        log "Уведомление: $MESSAGE"

        if [ -n "$SLACK_WEBHOOK" ]; then
            curl -s -X POST -H 'Content-type: application/json' \
                --data "{\"text\":\"Deploy $STATUS: $APP_NAME - $MESSAGE\"}" \
                "$SLACK_WEBHOOK" > /dev/null 2>&1
        fi
    }

    # Функция создания бэкапа текущей версии
    create_backup() {
        if [ -L "$CURRENT_LINK" ] && [ -d "$(readlink -f "$CURRENT_LINK")" ]; then
            CURRENT_VERSION=$(readlink "$CURRENT_LINK" | xargs basename)
            BACKUP_PATH="${BACKUP_DIR}/${CURRENT_VERSION}_$(date +%Y%m%d_%H%M%S)"

            log "Создание бэкапа текущей версии: $CURRENT_VERSION"
            cp -a "$(readlink -f "$CURRENT_LINK")" "$BACKUP_PATH" 2>/dev/null

            if [ $? -eq 0 ]; then
                log "Бэкап создан: $BACKUP_PATH"
                echo "$BACKUP_PATH"  # Возвращаем путь к бэкапу
            else
                log "ОШИБКА: Не удалось создать бэкап!"
                return 1
            fi
        else
            log "Текущая версия не найдена, бэкап не требуется."
            echo ""
        fi
    }

    # Функция деплоя новой версии
    deploy_new_version() {
        local VERSION="$1"
        local VERSION_DIR="${DEPLOY_DIR}/${VERSION}"

        log "=========================================="
        log "НАЧАЛО ДЕПЛОЯ: $APP_NAME v.$VERSION"
        log "=========================================="

        # Проверяем, существует ли директория с новой версией
        if [ ! -d "$VERSION_DIR" ]; then
            log "ОШИБКА: Директория версии не найдена: $VERSION_DIR"
            return 1
        fi

        # Создаём бэкап текущей версии
        BACKUP_PATH=$(create_backup)

        # Останавливаем сервис
        log "Остановка сервиса $SERVICE_NAME..."
        if [ "$SERVICE_MANAGER" = "systemd" ]; then
            systemctl stop "$SERVICE_NAME" 2>/dev/null
        else
            supervisorctl stop "$SERVICE_NAME" 2>/dev/null
        fi

        # Ждём остановки сервиса
        sleep 5

        # Переключаем симлинк на новую версию
        log "Переключение на новую версию: $VERSION"
        ln -sfn "$VERSION_DIR" "$CURRENT_LINK"

        # Запускаем сервис
        log "Запуск сервиса $SERVICE_NAME..."
        if [ "$SERVICE_MANAGER" = "systemd" ]; then
            systemctl start "$SERVICE_NAME" 2>/dev/null
        else
            supervisorctl start "$SERVICE_NAME" 2>/dev/null
        fi

        # Проверяем статус сервиса
        sleep 3
        if [ "$SERVICE_MANAGER" = "systemd" ]; then
            systemctl is-active --quiet "$SERVICE_NAME"
        else
            supervisorctl status "$SERVICE_NAME" | grep -q "RUNNING"
        fi

        if [ $? -ne 0 ]; then
            log "ОШИБКА: Сервис не запустился!"
            return 1
        fi

        log "Сервис успешно запущен."
        return 0
    }

    # Функция проверки здоровья приложения
    health_check() {
        log "Проверка доступности приложения (health check)..."

        local ELAPSED=0
        while [ "$ELAPSED" -lt "$HEALTH_CHECK_TIMEOUT" ]; do
            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$HEALTH_CHECK_URL" 2>/dev/null)

            if [ "$HTTP_CODE" = "200" ]; then
                log "Health check пройден успешно!"
                return 0
            fi

            log "Ожидание готовности... (прошло ${ELAPSED}s из ${HEALTH_CHECK_TIMEOUT}s)"
            sleep "$HEALTH_CHECK_INTERVAL"
            ELAPSED=$((ELAPSED + HEALTH_CHECK_INTERVAL))
        done

        log "ОШИБКА: Health check не пройден за ${HEALTH_CHECK_TIMEOUT} секунд!"
        return 1
    }

    # Функция отката к предыдущей версии
    rollback() {
        local BACKUP_PATH="$1"

        if [ -z "$BACKUP_PATH" ] || [ ! -d "$BACKUP_PATH" ]; then
            log "ОШИБКА: Бэкап для отката не найден!"
            return 1
        fi

        log "=========================================="
        log "ВЫПОЛНЯЕТСЯ ОТКАТ К ПРЕДЫДУЩЕЙ ВЕРСИИ"
        log "=========================================="

        # Останавливаем сервис
        log "Остановка сервиса..."
        if [ "$SERVICE_MANAGER" = "systemd" ]; then
            systemctl stop "$SERVICE_NAME" 2>/dev/null
        else
            supervisorctl stop "$SERVICE_NAME" 2>/dev/null
        fi

        sleep 3

        # Переключаем симлинк на бэкап
        log "Переключение на бэкап: $(basename "$BACKUP_PATH")"
        ln -sfn "$BACKUP_PATH" "$CURRENT_LINK"

        # Запускаем сервис
        log "Запуск сервиса..."
        if [ "$SERVICE_MANAGER" = "systemd" ]; then
            systemctl start "$SERVICE_NAME" 2>/dev/null
        else
            supervisorctl start "$SERVICE_NAME" 2>/dev/null
        fi

        sleep 3

        # Проверяем статус после отката
        if [ "$SERVICE_MANAGER" = "systemd" ]; then
            systemctl is-active --quiet "$SERVICE_NAME"
        else
            supervisorctl status "$SERVICE_NAME" | grep -q "RUNNING"
        fi

        if [ $? -eq 0 ]; then
            log "Откат выполнен успешно!"
            send_notification "Откат к предыдущей версии выполнен успешно" "ROLLBACK_SUCCESS"
            return 0
        else
            log "КРИТИЧЕСКАЯ ОШИБКА: Откат не удался!"
            send_notification "КРИТИЧНО: Откат не удался!" "ROLLBACK_FAILURE"
            return 1
        fi
    }

    # Главная логика
    main() {
        # Проверяем, передана ли версия
        if [ -z "$NEW_VERSION" ]; then
            log "Использование: $0 <version>"
            log "Пример: $0 v1.2.3"
            exit 1
        fi

        # Создаём необходимые директории
        mkdir -p "$BACKUP_DIR"

        # Создаём бэкап ДО деплоя
        BACKUP_PATH=$(create_backup)

        # Выполняем деплой
        if deploy_new_version "$NEW_VERSION"; then
            # Деплой успешен, проверяем здоровье
            if health_check; then
                log "=========================================="
                log "ДЕПЛОЙ УСПЕШНО ЗАВЕРШЁН: $APP_NAME v.$NEW_VERSION"
                log "=========================================="
                send_notification "Деплой v.$NEW_VERSION успешно завершён" "SUCCESS"
                exit 0
            else
                log "Health check не пройден, выполняется откат..."
                rollback "$BACKUP_PATH"
                exit 1
            fi
        else
            log "Ошибка при деплое, выполняется откат..."
            rollback "$BACKUP_PATH"
            exit 1
        fi
    }

    # Запуск
    main
    ```

!!! note "Скрипт #23: Ребут при нехватке памяти"

    Перезагружает сервер при критической нехватке оперативной памяти.

    ```bash
    #!/bin/bash
    # Скрипт мониторинга оперативной памяти
    # При критической нехватке памяти выполняет graceful reboot сервера

    # Настройки пороговых значений
    MEMORY_THRESHOLD=95                 # Порог использования памяти (%) для предупреждения
    CRITICAL_THRESHOLD=98               # Критический порог (%) для перезагрузки
    CHECK_INTERVAL=60                   # Интервал проверки (секунды)
    REBOOT_DELAY=30                     # Задержка перед перезагрузкой (секунды)

    # Настройки логирования
    LOG_FILE="/var/log/memory_monitor.log"
    MAX_LOG_SIZE=10485760               # Максимальный размер лога (10MB)

    # Настройки уведомлений
    SLACK_WEBHOOK=""                    # Webhook для Slack
    EMAIL_ALERT=""                      # Email для алертов

    # Функция логирования с ротацией лога
    log() {
        # Проверяем размер лога и выполняем ротацию при необходимости
        if [ -f "$LOG_FILE" ]; then
            LOG_SIZE=$(stat -f%z "$LOG_FILE" 2>/dev/null || stat -c%s "$LOG_FILE" 2>/dev/null)
            if [ "$LOG_SIZE" -gt "$MAX_LOG_SIZE" ]; then
                mv "$LOG_FILE" "${LOG_FILE}.old"
                touch "$LOG_FILE"
            fi
        fi
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
    }

    # Функция отправки уведомлений
    send_alert() {
        local MESSAGE="$1"
        local SEVERITY="$2"  # INFO, WARNING, CRITICAL

        log "[ALERT] [$SEVERITY] $MESSAGE"

        if [ -n "$SLACK_WEBHOOK" ]; then
            curl -s -X POST -H 'Content-type: application/json' \
                --data "{\"text\":\"Memory Alert [$SEVERITY]: $MESSAGE\"}" \
                "$SLACK_WEBHOOK" > /dev/null 2>&1
        fi
    }

    # Функция получения использования памяти в процентах
    get_memory_usage() {
        # Читаем данные из /proc/meminfo
        # MemTotal: общая память
        # MemAvailable: доступная память (с учётом кэшей, которые можно освободить)
        MEM_TOTAL=$(grep MemTotal /proc/meminfo | awk '{print $2}')      # В килобайтах
        MEM_AVAILABLE=$(grep MemAvailable /proc/meminfo | awk '{print $2}') # В килобайтах

        if [ -z "$MEM_TOTAL" ] || [ -z "$MEM_AVAILABLE" ] || [ "$MEM_TOTAL" -eq 0 ]; then
            echo "ERROR"
            return 1
        fi

        # Вычисляем использованную память
        MEM_USED=$((MEM_TOTAL - MEM_AVAILABLE))
        # Вычисляем процент использования
        USAGE_PERCENT=$(( (MEM_USED * 100) / MEM_TOTAL ))

        echo "$USAGE_PERCENT"
        return 0
    }

    # Функция получения детальной информации о памяти
    get_memory_info() {
        echo "--- Информация о памяти ---"
        echo "Общая память: $(grep MemTotal /proc/meminfo | awk '{print $2/1024 " MB"}')"
        echo "Доступная: $(grep MemAvailable /proc/meminfo | awk '{print $2/1024 " MB"}')"
        echo "Использовано: $(get_memory_usage)%"
        echo ""
        echo "Топ-5 процессов по использованию памяти:"
        ps aux --sort=-%mem | head -6 | tail -5
    }

    # Функция graceful reboot
    perform_reboot() {
        send_alert "КРИТИЧЕСКАЯ НЕХВАТКА ПАМЯТИ! Сервер будет перезагружен через $REBOOT_DELAY секунд!" "CRITICAL"

        log "=========================================="
        log "ВЫПОЛНЯЕТСЯ ПЕРЕЗАГРУЗКА СЕРВЕРА"
        log "Причина: Использование памяти превысило ${CRITICAL_THRESHOLD}%"
        log "=========================================="

        # Логируем информацию о памяти перед перезагрузкой
        get_memory_info >> "$LOG_FILE" 2>&1

        # Уведомляем о перезагрузке
        wall "ВНИМАНИЕ: Сервер будет перезагружен через $REBOOT_DELAY секунд из-за нехватки памяти!"

        log "Ожидание $REBOOT_DELAY секунд перед перезагрузкой..."
        sleep "$REBOOT_DELAY"

        # Выполняем reboot
        log "Перезагрузка..."
        /sbin/reboot
    }

    # Функция очистки кэша памяти (попытка избежать перезагрузки)
    cleanup_memory() {
        log "Попытка очистки кэша памяти..."

        # Синхронизируем данные на диск
        sync

        # Очищаем page cache, dentries и inodes (требует прав root)
        if [ -w /proc/sys/vm/drop_caches ]; then
            echo 3 > /proc/sys/vm/drop_caches
            log "Кэш памяти очищен."
        else
            log "Нет прав для очистки кэша (нужен root)."
        fi

        # Убиваем процессы с наибольшим потреблением памяти (опционально)
        # ВНИМАНИЕ: Это может прервать работу критических сервисов!
        # Раскомментируйте только если понимаете последствия:
        # ps aux --sort=-%mem | awk 'NR>1 && $4 > 10.0 {print $2}' | head -3 | xargs kill -15
    }

    # Главная функция мониторинга
    main() {
        log "=========================================="
        log "ЗАПУСК МОНИТОРИНГА ПАМЯТИ"
        log "Хост: $(hostname)"
        log "Порог предупреждения: ${MEMORY_THRESHOLD}%"
        log "Критический порог: ${CRITICAL_THRESHOLD}%"
        log "=========================================="

        # Проверяем, запущен ли уже мониторинг
        if [ -f "/var/run/memory_monitor.pid" ]; then
            OLD_PID=$(cat /var/run/memory_monitor.pid)
            if ps -p "$OLD_PID" > /dev/null 2>&1; then
                log "Мониторинг уже запущен (PID: $OLD_PID). Выход."
                exit 0
            fi
        fi

        # Записываем свой PID
        echo $$ > /var/run/memory_monitor.pid

        # Основной цикл мониторинга
        while true; do
            MEMORY_USAGE=$(get_memory_usage)

            if [ "$MEMORY_USAGE" = "ERROR" ]; then
                log "ОШИБКА: Не удалось получить данные о памяти!"
                sleep "$CHECK_INTERVAL"
                continue
            fi

            log "Использование памяти: ${MEMORY_USAGE}%"

            # Проверяем пороговые значения
            if [ "$MEMORY_USAGE" -ge "$CRITICAL_THRESHOLD" ]; then
                send_alert "Критическая нехватка памяти: ${MEMORY_USAGE}%!" "CRITICAL"
                get_memory_info >> "$LOG_FILE" 2>&1
                perform_reboot
                break
            elif [ "$MEMORY_USAGE" -ge "$MEMORY_THRESHOLD" ]; then
                send_alert "Высокое использование памяти: ${MEMORY_USAGE}%!" "WARNING"
                # Пытаемся очистить кэш
                cleanup_memory
            fi

            sleep "$CHECK_INTERVAL"
        done

        # Удаляем PID-файл при выходе
        rm -f /var/run/memory_monitor.pid
    }

    # Обработка сигналов для корректного завершения
    trap "log 'Получен сигнал завершения. Остановка мониторинга...'; rm -f /var/run/memory_monitor.pid; exit 0" SIGTERM SIGINT

    # Запуск
    main "$@"
    ```

## Ограничения bash

!!! note "Когда bash не подходит"

    **Сложная логика и большие программы**
    <br>
    Bash не предназначен для написания сложных алгоритмов. Для этого лучше использовать Python, Go или другие языки программирования.

    **Работа с JSON, XML, сложными структурами данных**
    <br>
    Bash плохо работает с вложенными структурами данных. Для JSON лучше использовать jq, Python или специализированные инструменты.

    **Математические вычисления**
    <br>
    Bash поддерживает только целочисленную арифметику. Для сложных вычислений используйте bc, awk или Python.

    **Параллельное выполнение и многопоточность**
    <br>
    Bash выполняет команды последовательно. Параллельность возможна через фоновые процессы, но управление ими затруднено.

    **Переносимость между ОС**
    <br>
    Скрипты, написанные под Linux, могут не работать в macOS или BSD без модификаций (различия в утилитах date, sed, awk и др.).

    **Обработка ошибок**
    <br>
    В bash сложно писать надёжный код с обработкой исключений. Нет структурированной обработки ошибок, как в современных языках.

    **Работа с базами данных**
    <br>
    Хотя можно вызывать CLI-клиенты (mysql, psql), bash не является лучшим выбором для сложной работы с БД.

    **Безопасность**
    <br>
    Bash-скрипты уязвимы к инъекциям, если неправильно обрабатываются входные данные. Требуют внимательного подхода к экранированию спецсимволов.
