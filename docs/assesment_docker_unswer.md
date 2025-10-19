## Docker - открытая платформа для разработки, доставки и запуска приложений в контейнерах

### Список команд для работы с docker

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

### Практики по работе с Docker образами

| Решение | Описание | Преимущества | Команда |
|---|---|---:|---|
| Multi-stage build | Сборка артефакта в одном этапе, копирование только runtime-артефактов в финальный образ | Существенно меньше размер, нет инструментов сборки в финале | `# builder\nFROM golang:1.20 AS builder\nWORKDIR /app\n... \n# final\nFROM gcr.io/distroless/static\nCOPY --from=builder /app/bin/myapp /myapp` |
| Минимальные базовые образы (Alpine/Distroless/Scratch) | Использовать лёгкие или пустые образы вместо полноценных дистрибутивов | Меньше слоёв, меньший размер, меньше атакуемой поверхности | `FROM python:3.11-slim` или `FROM gcr.io/distroless/base` |
| Удаление build-зависимостей и кешей | После установки пакетов удалять списки apt/pip кеши и dev-пакеты | Уменьшение размера слоя после установки | `RUN apt-get install -y build-essential && make && apt-get purge -y build-essential && rm -rf /var/lib/apt/lists/*` |
| .dockerignore | Исключать ненужные файлы/папки из контекста сборки | Меньший контекст → быстрее билд, меньше случайных файлов в образе | `.dockerignore:\nnode_modules\n.git\nDockerfile` |
| Правильный порядок слоёв | Помещать часто меняющиеся файлы позже, зависимые от них слои invalidируются реже | Лучшая кэшируемость билдов → быстрее сборки в CI | Сначала `COPY package.json` / `RUN npm ci`, потом `COPY . .` |
| BuildKit и кеши слоёв | Включить BuildKit и использовать `--cache-from`/inline cache | Быстрые и повторяемые сборки в CI, уменьшение сетевого трафика | `DOCKER_BUILDKIT=1 docker build --cache-from myrepo/myimage:cache -t myrepo/myimage:latest .` |
| Минимизация количества слоёв | Объединять команды в одном RUN, использовать && и очистку в том же слое | Меньше слоёв → меньше метаданных и небольшая экономия места | `RUN apt-get update && apt-get install -y ... && rm -rf /var/lib/apt/lists/*` |
| Стриппинг и сжатие бинарей | Удаление символов отладки и сжатие (strip / upx) для собственных бинарей | Меньше размер исполняемого файла | `RUN strip /app/bin/myapp` или `RUN upx --best /app/bin/myapp` |
| Использование runtime-only образа | Перенос только runtime-артефактов (бинарий/зависимостей) в чистый образ | Минимальный финальный образ без инструментов сборки | См. пример multi-stage (copy только бинарей) |
| Кэширование зависимостей (CI) | Кешировать слои с зависимостями между сборками в CI | Быстрая сборка при неизменных зависимостях | Использовать `--cache-from` или кэш runner (GitLab/Actions) |
| Установка без рекомендаций | apt-get install --no-install-recommends / pip install --no-cache-dir | Уменьшает установку лишних пакетов/файлов | `RUN apt-get install -y --no-install-recommends curl` |
| Сканирование и оптимизация уязвимостей | Trivy/Grype/Clair для поиска уязвимостей и удаления ненужных пакетов | Безопасность + удаление ненужного софта | `trivy image myrepo/myapp:latest` |
| Сжатие слоёв / squash (опционально) | Объединение слоёв в один (редко нужно при BuildKit) | Меньше метаданных; иногда помогает при legacy workflow | `docker build --squash` (экспериментально) |
| Артефактная упаковка / distroless + шифрование | Использовать distroless и подписывать/сканировать образы | Минимизация векторов атаки и ЦСР/подписи образов | `FROM gcr.io/distroless/static` + cosign/podman sign |

### Команды и ключевые понятия docker compose

| Компонент / команда | Описание | Параметры | Пример |
|---|---|---|---|
| docker-compose.yml | Файл описания стека: services, networks, volumes, configs | version / services / volumes / networks / deploy и т.д. | `version: "3.8"\nservices:\n  web:\n    image: nginx` |
| docker compose up | Создать и запустить все сервисы из конфигурации | `-d` (detached), `--build`, `--remove-orphans` | `docker compose up -d --build` |
| docker compose down | Остановить и удалить контейнеры/сети/тома (опционально) | `--volumes`, `--rmi local|all` | `docker compose down --volumes` |
| docker compose build | Построить образы по настройкам в compose | `--no-cache`, `--pull` | `docker compose build --no-cache` |
| docker compose logs | Просмотреть логи сервисов | `-f` (follow), `--tail <n>` | `docker compose logs -f web` |
| docker compose ps | Показать состояние сервисов и контейнеров | `-a` (все) | `docker compose ps` |
| docker compose exec | Выполнить команду в запущенном контейнере | `-T` (без псевдотерминала), `-u` (user) | `docker compose exec web sh` |
| docker compose run | Создать и запустить одноразовый контейнер в контексте сервиса | `--rm`, `--no-deps`, `-e` | `docker compose run --rm web pytest` |
| docker compose pull / push | Скачать / отправить образы из/в registry | — | `docker compose pull` |
| docker compose config | Валидация и вывод итогового объединённого конфига | `--services`, `--volumes` | `docker compose config --services` |
| profiles / depends_on | Контроль порядка запуска и условные профили | `depends_on` — порядок; `profiles` — включение сервисов | В compose: `depends_on: - db` |
| volumes | Том для хранения данных между перезапусками | `driver`, `driver_opts`, `external` | `volumes:\n  dbdata:\n    driver: local` |
| networks | Настройка сетей между сервисами (bridge, overlay) | `driver`, `driver_opts`, `external` | `networks:\n  front:\n  back:` |
| restart policy | Политика рестарта контейнера | `no|always|on-failure|unless-stopped` | `restart: unless-stopped` |
| env_file / environment | Передача переменных окружения в сервис | `env_file: .env` или `environment:` | `environment:\n  - DB_HOST=db` |

### Что такое docker compose

!!! note ""

    Docker Compose — инструмент для определения и запуска многоконтейнерных Docker‑приложений через YAML‑файл (обычно docker-compose.yml). Описывает сервисы, сети и тома в декларативном виде; позволяет поднимать/остановливать стек одной командой.