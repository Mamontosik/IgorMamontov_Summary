# Краткая справка для прохождения assesment на DevOps роль

## OSI (Open Systems Interconnection) — модель взаимодействия устройств в компьютерных сетях

| Уровень | Название | Короткое описание | Применение | Используемые порты/интерфейсы | Ограничения |
|---:|---|---|---|---|---|
| 7 | Прикладной (Application) | Интерфейс приложений к сети; сетевые сервисы | Веб (HTTP/HTTPS), почта, FTP, DNS, API | TCP/UDP-порты приложений (напр. 80, 443, 25, 53, 21, 22) | Зависимость от безопасности и форматов; уязвимости приложений |
| 6 | Представления (Presentation) | Кодирование/декодирование, шифрование, сжатие | TLS/SSL, кодировки (UTF-8), сжатие | N/A (функция формата/шифрования) | Нагрузка на CPU; несовместимости форматов |
| 5 | Сеансовый (Session) | Управление сессиями/диалогами (установление/завершение) | RPC, управление сессиями приложений, чекпоинты | N/A | Stateful — сложность восстановления и масштабирования |
| 4 | Транспортный (Transport) | End-to-end связь: надежность, управление потоком/ошибками | TCP (надежно), UDP (ненадежно), мультиплексирование по портам | TCP/UDP номера портов (well-known/registered) | TCP: задержки, Head‑of‑Line; UDP: отсутствие гарантий доставки |
| 3 | Сетевой (Network) | Логическая адресация и маршрутизация (IP) | Маршрутизация IPv4/IPv6, ICMP, BGP/OSPF | N/A (используются IP‑адреса) | MTU/фрагментация; масштаб таблиц маршрутизации |
| 2 | Канальный (Data Link) | Передача кадров по сети, MAC‑адресация, VLAN | Ethernet, ARP, VLAN, коммутаторы, PPP | MAC‑адреса; физические/коммутаторные порты | Broadcast‑домен, MTU, уязвимости на уровне LAN (ARP spoofing) |
| 1 | Физический (Physical) | Передача битов: электрические/оптические/радиосигналы, коннекторы | Кабели (UTP, fiber), Wi‑Fi, разъёмы (RJ‑45, SFP) | Физические порты/разъёмы (RJ‑45, SFP и т.д.) | Полоса, дальность, помехи, аппаратные поломки |

## Краткая таблица протоколов
| Протокол | Расшифровка | Краткое описание | Поведение / особенности | Типичные порты / номера | Команды |
|---|---|---|---|---:|---|
| `TCP` | Transmission Control Protocol | Надёжный протокол с установкой соединения и гарантией доставки | Connection‑oriented, handshake, retransmit | любые TCP‑порты (напр. 22, 80, 443) | `ss -tna` / `tcpdump -n 'tcp port 22' -c 5` |
| `UDP` | User Datagram Protocol | Лёгкий протокол без гарантий доставки | Connectionless, низкие задержки | DNS 53 (UDP), DHCP 67/68, RTP/VoIP | `ss -una` / `dig @8.8.8.8 example.com` |
| `Broadcast` | Широковещательная доставка | Отправка пакета всем узлам подсети | L2/L3 broadcast, не маршрутизируется по умолчанию | MAC ff:ff:ff:ff:ff:ff; IP 255.255.255.255 | `tcpdump -n broadcast -c 20` / `arp -a` |
| `ICMP` | Internet Control Message Protocol | Диагностика и сообщения об ошибках (ping) | Используется для ping/traceroute, нет портов | нет портов (типы/коды ICMP) | `ping -c 3 8.8.8.8` / `traceroute -n 8.8.8.8` |
| `DHCP` | Dynamic Host Configuration Protocol | Автоматическое выдача IP и параметров сети | Клиент‑сервер, discovery через broadcast | UDP 67 (server), UDP 68 (client) | `tcpdump -n udp port 67 or udp port 68 -c 20` / `sudo dhclient -v` |
| `ARP` | Address Resolution Protocol | Разрешение IP → MAC в локальной сети | Broadcast запросы и кэширование | нет портов (L2) | `arp -a` / `ip neigh show` |

## Коды ошибок в рамках сетевого взаимодействия

| Код | Значение | Краткое описание | Как диагностировать | Быстрое исправление |
|---:|---|---|---|---|
| `200` | OK | Успешный запрос | Логи приложения, ответ body | Нет действия |
| `201` | Created | Ресурс создан | Логи API, Location header | Проверить запись в БД |
| `301` | Moved Permanently | Постоянный редирект | curl -I, Location | Исправить URL/конфиг |
| `302` | Found | Временный редирект | curl -I | Проверить логи редиректов |
| `400` | Bad Request | Неверный запрос | Логи приложения, validation errors | Исправить запрос/валидацию |
| `401` | Unauthorized | Нет/истёк токен | Заголовки Authorization | Проверить токен/аутентификацию |
| `403` | Forbidden | Доступ запрещён | ACL/permission logs | Исправить права/ACL |
| `404` | Not Found | Ресурс не найден | URL, роуты | Проверить роут/ресурс |
| `408` | Request Timeout | Таймаут запроса | nginx/timeout logs | Увеличить таймаут/оптимизировать |
| `429` | Too Many Requests | Rate limit | Заголовки rate-limit, метрики | Backoff, увеличить лимит |
| `500` | Internal Server Error | Ошибка приложения | Application logs, stacktrace | Исправить баг/деплой |
| `502` | Bad Gateway | Неверный ответ upstream | Nginx/HAProxy logs | Проверить/перезапустить upstream |
| `503` | Service Unavailable | Сервис недоступен | Health checks, metrics | Масштаб/maintenance |
| `504` | Gateway Timeout | Таймаут upstream | Nginx logs, slow backend | Оптимизировать backend/таймаут |

## GIT - команды для работы с репозиториями

| Команда | Результат / что делает команда | Доступные параметры (частые) | Пример команды |
|---|---|---|---|
| `git init` | Создаёт новый локальный репозиторий (инициализирует `.git`). | `--bare` (создаёт bare-репозиторий без рабочей копии) | `git init my-repo` |
| `git clone` | Клонирует удалённый репозиторий в новую папку. | `--depth <n>` (shallow clone — только последние n коммитов), `--branch <name>` (клонировать указанную ветку) | `git clone https://github.com/user/repo.git` |
| `git add` | Добавляет изменения в индекс (staging area) для следующего коммита. | `-A` (добавить/удалить все изменения), `.` (все изменения в текущей папке), `<path>` (конкретный путь/файл) | `git add .` / `git add src/main.c` |
| `git status` | Показывает текущее состояние рабочей директории и индекс. | `-s` (короткий/сводный вывод) | `git status` |
| `git diff` | Показ отличий: рабочая директория vs индекс; `--staged` — индекс vs последний коммит. | `--staged` (показать отличия, подготовленные к коммиту), `--name-only` (только имена изменённых файлов) | `git diff` / `git diff --staged` |
| `git commit` | Создаёт коммит из индекса. `--amend` исправляет последний коммит. | `-m <msg>` (сообщение коммита), `--amend` (изменить последний коммит), `-a` (автоматически добавить отслеживаемые файлы) | `git commit -m "fix bug"` / `git commit --amend` |
| `git log` | Показывает историю коммитов; `--oneline` — компактно, `--graph` — ASCII‑граф ветвления. | `--oneline` (одна строка на коммит), `--graph` (визуализация ветвления), `-n <N>` (показать N последних) | `git log --oneline --graph` |
| `git branch` | Работа с ветками: создать, перечислить, удалить. | `-a` (все локальные и remote ветки), `-d` (удалить ветку безопасно), `-D` (форсированное удаление), `<name>` (имя ветки) | `git branch feature/x` / `git branch -D old` |
| `git checkout` | Переключение веток или восстановление файлов; `-b` создаёт новую ветку и переключается на неё. | `-b <branch>` (создать и переключиться на ветку) | `git checkout -b feature/x` / `git checkout main` |
| `git switch` | Современная команда для переключения/создания веток (альтернатива `checkout` для веток). | `-c <branch>` (создать новую ветку и переключиться) | `git switch -c feature/x` / `git switch main` |
| `git merge` | Слияние указанной ветки в текущую; при конфликте нужно решать вручную. | `--no-ff` (всегда создать merge-коммит), `--squash` (объединить изменения без создания merge-коммита) | `git merge feature/x` |
| `git rebase` | Перенос (переписывание) коммитов на основе другой ветки; `-i` — интерактивное редактирование. | `-i` (интерактивный rebase: squash, fixup, редактирование) | `git rebase main` / `git rebase -i HEAD~3` |
| `git fetch` | Получает обновления с удалённого репозитория, не меняя рабочую ветку. | `--all` (все удалённые), `--prune` (удалить remote‑ссылки, которых больше нет на сервере) | `git fetch origin` / `git fetch --all --prune` |
| `git pull` | `fetch` + `merge` (по умолчанию). Часто используют `--rebase` для линейной истории. | `--rebase` (выполнить rebase вместо merge после fetch) | `git pull --rebase origin main` |
| `git push` | Отправляет локальные коммиты на удалённый репозиторий. | `--force` (форсировать перезапись истории — опасно), `--set-upstream` (установить upstream для ветки), `origin <branch>` (куда/какая ветка) | `git push origin feature/x` / `git push --set-upstream origin feature/x` |
| `git remote` | Управление удалёнными репозиториями (добавить/удалить/показать). | `add` (добавить remote), `remove` (удалить), `set-url` (сменить URL), `-v` (показать URL) | `git remote add origin git@github.com:user/repo.git` |
| `git tag` | Создаёт метки (теги) коммитов: lightweight или аннотированные. | `-a <name> -m <msg>` (аннотированный тег с сообщением), `<name>` (имя тега) | `git tag -a v1.0 -m "release 1.0"` / `git push origin v1.0` |
| `git stash` | Временно сохраняет незакоммиченные изменения и очищает рабочую директорию. | `push` (положить изменения в стек), `pop` (взять и удалить из стека), `list` (показать список), `apply` (применить без удаления) | `git stash push -m "WIP"` / `git stash pop` |
| `git reset` | Перемещает указатель ветки: `--soft` сохраняет индекс/файлы, `--hard` откатывает всё. | `--soft` (только переместить HEAD), `--mixed` (по умолчанию — сброс индекса), `--hard` (удалить изменения в рабочей директории — осторожно) | `git reset --soft HEAD~1` / `git reset --hard HEAD~1` |
| `git revert` | Создаёт новый коммит, отменяющий указанный коммит (без переписывания истории). | `-m <parent>` (для merge — указать родителя, который считать "основным") | `git revert <commit>` |
| `git cherry-pick` | Применяет выбранный коммит из другой ветки в текущую (копирует изменения). | `<commit>` (хеш коммита) | `git cherry-pick a1b2c3d` |
| `git rm` | Удаляет файл из индекса и рабочей копии; `--cached` — только из индекса. | `-r` (рекурсивно для папок), `--cached` (оставить локальный файл, удалить из индекса) | `git rm file.txt` / `git rm --cached file.txt` |
| `git mv` | Переименовывает/перемещает файлы и фиксирует это как одно действие. | — | `git mv oldname newname` |
| `git blame` | Показывает, кто и в каком коммите изменил каждую строку файла. | `-L <start>,<end>` (ограничить диапазон строк) | `git blame -L 10,20 file.c` |
| `git bisect` | Двоичный поиск проблемного коммита (поиск, где появился баг). | `start` (инициализация), `good` (отметить рабочую версию), `bad` (отметить проблемную) | `git bisect start; git bisect bad; git bisect good v1.0` |
| `git reflog` | Локальный журнал движений HEAD — помогает восстановить потерянные коммиты. | — | `git reflog` |
| `git config` | Настройки Git (имя, email, алиасы и т.д.). | `--global` (для пользователя), `--local` (для репозитория), `user.name` (параметр) | `git config --global user.name "Ivan"` |

### Популярные приложения/клиенты для работы с Git
!!! note ""

    - Git (CLI) — официальная командная строка.  
    - Visual Studio Code — встроенная поддержка Git + GUI.  
    - GitHub Desktop (Windows/macOS) — простой GUI для GitHub.  
    - Sourcetree (Atlassian) — бесплатный GUI с визуализацией веток.  
    - GitKraken — современный UI (есть бесплатные и платные версии).  
    - Tower — платный клиент для macOS/Windows.  
    - SmartGit — мультиплатформенный клиент.  
    - TortoiseGit — интеграция Git в проводник Windows.  
    - IntelliJ IDEA / PhpStorm / WebStorm — встроенная поддержка Git (JetBrains).  
    - Git Extensions — GUI для Windows.

## Docker - открытая платформа для разработки, доставки и запуска приложений в контейнерах.

| Команда | Описание | Дополнительные параметры (с пояснением) | Пример команды |
|---|---|---|---|
| `docker run` | Создать и запустить контейнер из образа | `-d` (detached), `--name <name>`, `-p host:container` (порт-маппинг), `-v host:container` (том), `--rm` (удалить после выхода), `-e KEY=VAL` (env) | `docker run -d --name web -p 8080:80 nginx` |
| `docker build` | Построить образ из Dockerfile | `-t name:tag` (имя/тег), `-f <Dockerfile>` (файл), `--build-arg KEY=VAL` | `docker build -t myapp:1.0 .` |
| `docker pull` | Скачать образ из реестра | `<image>:<tag>` (по умолчанию `latest`) | `docker pull redis:6.2` |
| `docker push` | Отправить образ в реестр | Требует `docker login` и корректного имени репозитория | `docker push myrepo/myapp:1.0` |
| `docker ps` | Показать запущенные контейнеры | `-a` (включая остановленные), `--format` (кастомный вывод) | `docker ps -a --format '{{.ID}}\t{{.Names}}\t{{.Status}}'` |
| `docker images` | Показать локальные образы | `--format`, `-a` (включая промежуточные) | `docker images --format '{{.Repository}}:{{.Tag}}\t{{.Size}}'` |
| `docker stop` | Остановить запущенный контейнер (graceful) | `-t <secs>` (таймаут перед SIGKILL) | `docker stop -t 10 web` |
| `docker start` | Запустить ранее созданный/остановленный контейнер | — | `docker start web` |
| `docker restart` | Перезапустить контейнер (stop + start) | `-t <secs>` | `docker restart -t 5 web` |
| `docker rm` | Удалить один или несколько контейнеров | `-f` (force), `-v` (удалить связанные тома) | `docker rm -f old_container` |
| `docker rmi` | Удалить локальный образ | `-f` (принудительно) | `docker rmi myapp:1.0` |
| `docker exec` | Выполнить команду внутри запущенного контейнера | `-it` (интерактивный терминал), `-u` (пользователь) | `docker exec -it web bash` |
| `docker logs` | Просмотреть логи контейнера | `-f` (follow), `--since`, `--tail <n>` | `docker logs -f --tail 100 web` |
| `docker inspect` | Полная JSON‑информация о контейнере/образе/сети | `--format '{{.Config}}'` для конкретных полей | `docker inspect --format '{{.NetworkSettings.IPAddress}}' web` |
| `docker stats` | Живые метрики (CPU, память) контейнеров | `--no-stream` (одноразовый снимок), указывать контейнеры | `docker stats --no-stream web db` |
| `docker network ls` | Показать сети Docker | `docker network inspect <network>` для деталей | `docker network ls` / `docker network inspect bridge` |
| `docker volume ls` | Показать тома Docker | `docker volume inspect <volume>` | `docker volume ls` |
| `docker-compose up` | (docker-compose) Поднять multi‑container приложение | `-d` (detached), `--build`, `--scale svc=n` | `docker-compose up -d --build` |
| `docker system prune` | Очистить неиспользуемые объекты | `-a` (удалить все неиспользуемые образы), `--volumes` | `docker system prune -a --volumes` |

