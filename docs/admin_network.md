# Сеть и порты

## Коды ошибок HTTP

!!! note "Коды состояния HTTP"

    | Код | Категория | Примеры |
    | --- | --- | --- |
    | `1xx` | Информационные | 100 Continue, 101 Switching Protocols |
    | `2xx` | Успех | 200 OK, 201 Created, 204 No Content |
    | `3xx` | Перенаправление | 301 Moved, 302 Found, 304 Not Modified |
    | `4xx` | Ошибка клиента | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 405 Method Not Allowed, 408 Request Timeout |
    | `5xx` | Ошибка сервера | 500 Internal Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

    **Популярные ошибки:**

    | Код | Описание |
    | --- | --- |
    | `400` | Неверный запрос — синтаксическая ошибка |
    | `401` | Неавторизованный доступ — требуется аутентификация |
    | `403` | Доступ запрещён — нет прав |
    | `404` | Страница не найдена |
    | `405` | Метод не разрешён для данного URL |
    | `408` | Время ожидания истекло |
    | `500` | Внутренняя ошибка сервера |
    | `502` | Ошибка шлюза — некорректный ответ upstream |
    | `503` | Сервис недоступен — перегрузка или обслуживание |
    | `504` | Шлюз не отвечает — timeout upstream |

## Популярные порты

!!! note "Стандартные порты"

    | Порт | Протокол | Сервис | Описание |
    | --- | --- | --- | --- |
    | 20 | TCP | FTP-DATA | FTP передача данных |
    | 21 | TCP | FTP | FTP управление |
    | 22 | TCP | SSH | Secure Shell |
    | 23 | TCP | Telnet | Незащищённый удалённый доступ |
    | 25 | TCP | SMTP | Отправка почты |
    | 53 | UDP | DNS | Разрешение доменных имён |
    | 67-68 | UDP | DHCP | Автоматическая настройка сети |
    | 80 | TCP | HTTP | Веб-трафик |
    | 110 | TCP | POP3 | Получение почты |
    | 123 | UDP | NTP | Синхронизация времени |
    | 143 | TCP | IMAP | Получение почты (протокол) |
    | 443 | TCP | HTTPS | Защищённый веб-трафик |
    | 3306 | TCP | MySQL | База данных MySQL |
    | 5432 | TCP | PostgreSQL | База данных PostgreSQL |
    | 6379 | TCP | Redis | Кэш и очереди |
    | 8080 | TCP | HTTP-ALT | Альтернативный HTTP (dev/staging) |
    | 9090 | TCP | Prometheus | Мониторинг |

## Сетевые протоколы

!!! note "Описание протоколов"

    | Протокол | Порты | Особенности | Пример |
    | --- | --- | --- | --- |
    | TCP | 80, 443, 25 | Надёжная доставка, 3-way handshake | HTTP, HTTPS, SMTP |
    | UDP | 53, 67-68 | Без соединения, real-time | DNS, DHCP |
    | ICMP | — | Диагностика | ping, traceroute |
    | ARP | — | Преобразование IP → MAC | `arp -a` |
    | DHCP | 67-68 | Автоматическая настройка IP | Аренда адресов |
    | DNS | 53 | Разрешение доменных имён | `dig`, `nslookup` |

    **Сценарий: Проверить DNS**
    ```bash
    dig google.com
    # ;; ANSWER SECTION:
    # google.com.  300  IN  A  142.250.74.46
    ```

    ---

    **Сценарий: Проверить DHCP**
    ```bash
    dmesg | grep -i dhcp
    # dhclient[123]: DHCPACK from 192.168.1.1
    cat /var/lib/dhcp/dhclient.leases
    ```

    ---

    **Сценарий: Просмотр ARP-кэша**
    ```bash
    arp -a
    # ? (192.168.1.1) at aa:bb:cc:dd:ee:ff [ether] on eth0
    ip neigh show
    ```

## Troubleshooting сети

!!! note "Диагностика соединений"

    **ping** — проверка доступности
    ```bash
    ping -c 5 google.com              # 5 пакетов
    ping -i 2 google.com               # Интервал 2 сек
    ping -s 1024 google.com            # Размер пакета
    ```

    **Сценарий: Проверить связность**
    ```bash
    ping -c 4 8.8.8.8
    # 64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=14.2 ms
    ping -c 4 google.com               # Проверить DNS
    ```

    ---

    **traceroute/mtr** — трассировка маршрута
    ```bash
    traceroute google.com              # Linux
    tracert -d google.com              # Windows (без DNS)
    mtr google.com                     # Комбинированный (ping + traceroute)
    ```

    **Сценарий: Найти узкое место**
    ```bash
    mtr -n google.com
    # Host                Loss%  Snt   Last  Avg  Best  Wrst StDev
    # 1. 192.168.1.1       0.0%   10    1.2   1.5   0.8   2.1   0.4
    # 2. 10.0.0.1           0.0%   10    5.3   5.8   4.9   6.2   0.5
    ```

    ---

    **nslookup/dig** — DNS запросы
    ```bash
    nslookup google.com                # Информация о домене
    nslookup -type=MX google.com      # MX записи
    dig google.com                     # Подробный запрос
    dig +short google.com              # Только IP
    host google.com                    # Быстрый lookup
    ```

    **Сценарий: Проверить MX записи**
    ```bash
    dig MX google.com
    # ;; ANSWER SECTION:
    # google.com.  3600  IN  MX  10 smtp.google.com.
    ```

    ---

    **netstat/ss** — информация о соединениях
    ```bash
    netstat -tuln                      # Открытые порты (listening)
    netstat -anp | grep ESTABLISHED    # Установленные соединения
    ss -tuln                           # Аналог netstat (современный)
    ss -s                              # Статистика
    ```

    **Сценарий: Найти процесс на порту**
    ```bash
    ss -tuln | grep :80
    # State    Recv-Q   Send-Q   Local Address:Port   Peer Address:Port
    # LISTEN   0        128      0.0.0.0:80           0.0.0.0:*
    lsof -i :80                       # Процесс на порту 80
    ```

    ---

    **nmap** — сканирование портов
    ```bash
    nmap -sS 192.168.1.1              # SYN-сканирование
    nmap -p 80,443 google.com         # Конкретные порты
    nmap -O server.com                # Определение ОС
    nmap -sV server.com                # Версии сервисов
    ```

    **Сценарий: Сканирование сети**
    ```bash
    nmap -sS -p 1-1000 192.168.1.0/24
    # Host 192.168.1.1 is up
    # PORT    STATE  SERVICE
    # 22      open  ssh
    # 80      open  http
    # 443     open  https
    ```

    ---

    **telnet/curl/nc** — тестирование соединений
    ```bash
    telnet host 80                     # Проверка TCP
    curl -v http://site.com           # GET с деталями
    curl -I https://site.com          # HEAD запрос
    wget --spider site.com            # Проверка доступности
    nc -zv host 80                     # Netcat проверка
    ```

    **Сценарий: Проверить SSL-сертификат**
    ```bash
    echo | openssl s_client -connect google.com:443 -servername google.com 2>/dev/null | openssl x509 -noout -dates
    # notBefore=Jan  1 00:00:00 2024 GMT
    # notAfter=Jan  1 00:00:00 2025 GMT
    ```

## Модель OSI

!!! note "Уровни OSI"

    | Уровень | Название | Протоколы | Назначение |
    | --- | --- | --- | --- |
    | 7 | Прикладной | HTTP, HTTPS, DNS, FTP, SMTP | Взаимодействие приложений, формирование запросов |
    | 6 | Представления | TLS, SSL, JPEG, JSON, ASCII | Шифрование, сжатие, кодирование |
    | 5 | Сеансовый | TLS, RPC, NetBIOS | Управление сессиями, аутентификация |
    | 4 | Транспортный | TCP, UDP, QUIC, SCTP | Надёжная доставка, мультиплексирование портов |
    | 3 | Сетевой | IP, ICMP, IPsec, OSPF/BGP | Маршрутизация, логическая адресация |
    | 2 | Канальный | Ethernet, Wi-Fi, ARP, PPP | Физическая адресация (MAC), передача кадров |
    | 1 | Физический | Ethernet PHY, оптическое волокно, DSL | Передача битов как сигналов |

## Безопасные соединения

!!! note "SSH, SCP, файловые операции"

    **SSH**
    ```bash
    ssh user@host                     # Подключение
    ssh -p 2222 user@host            # Порт
    ssh -i key.pem user@host          # Приватный ключ
    ssh -L 8080:localhost:80 host   # Проброс порта (туннель)
    ssh -N -L 8080:localhost:80 host # Без выполнения команд
    ssh-keygen -t rsa                # Генерация ключей
    ssh-copy-id user@host            # Копирование ключа
    ```

    **Сценарий: Проброс порта**
    ```bash
    ssh -L 8080:localhost:80 user@host
    # Теперь localhost:8080 → host:80
    ```

    ---

    **scp/rsync** — копирование файлов
    ```bash
    scp file.txt user@host:/path      # Копировать файл
    scp -r dir/ user@host:/path       # Копировать директорию
    scp -P 2222 file user@host:/path  # Альтернативный порт
    rsync -avz dir/ user@host:/path   # Синхронизация
    ```

    **Сценарий: Синхронизация с удалённым сервером**
    ```bash
    rsync -avz --delete /local/dir/ user@host:/remote/dir/
    # building file list ... done
    # sent 12345 bytes  received 234 bytes
    ```

    ---

    **Удалённое выполнение**
    ```bash
    ssh user@host "command"          # Одна команда
    ssh user@host 'bash -s' < script.sh  # Скрипт
    ```

## Таблица сетевых команд

!!! note "Быстрый справочник"

    | Команда | Назначение | Пример |
    | --- | --- | --- |
    | `ping` | Проверка доступности | `ping -c 4 google.com` |
    | `traceroute` | Маршрут пакетов | `traceroute -d host` |
    | `nslookup` | Информация о DNS | `nslookup domain.com` |
    | `dig` | Детальный DNS-запрос | `dig +short domain.com` |
    | `netstat` | Состояние сети | `netstat -tuln` |
    | `ss` | Современный netstat | `ss -tuln` |
    | `ip` | Управление сетью | `ip addr show` |
    | `ifconfig` | Настройка интерфейсов | `ifconfig -a` |
    | `route` | Таблица маршрутизации | `route -n` |
    | `arp` | Кэш ARP | `arp -a` |
    | `nmap` | Сканирование сети | `nmap -sS host` |
    | `ssh` | Удалённый доступ | `ssh user@host` |
    | `scp` | Безопасное копирование | `scp file host:/path` |
    | `curl` | HTTP-клиент | `curl -I https://site.com` |
    | `nc` | Netcat | `nc -zv host 80` |
    | `mtr` | Комбинированная трассировка | `mtr google.com` |

    **Сценарий: Полный аудит сети**
    ```bash
    ss -tuln                          # Открытые порты
    # LISTEN   0   128   *:80        *:*
    nmap -sS -O localhost             # Сканирование localhost
    # PORT    STATE  SERVICE
    # 22      open  ssh
    # 80      open  http
    netstat -i                        # Сетевые интерфейсы
    ```

!!! warning "Сетевые проблемы"

    1. `ping -c 4 8.8.8.8` — проверить связность с интернетом
    2. `nslookup google.com` — проверить DNS
    3. `traceroute google.com` — найти проблемный узел
    4. `ss -tuln | grep :PORT` — проверить, слушает ли порт
    5. `dmesg | tail -20` — ошибки сетевого стека