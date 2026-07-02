# Полезные команды для установки и настройки Kubernetes

## Работа с файлом Dockerfile при формировании контейнера

!!! tip "COPY вместо ADD"
    Если не нужно распаковывать архив — используйте `COPY`. Распаковка — явно в `RUN`.
    ``` yaml
    COPY app.tar.gz /tmp/
    RUN tar -xzf /tmp/app.tar.gz -C /app && rm /tmp/app.tar.gz
    ```

!!! warning "Не копируйте лишнее"
    Каждый `COPY`/`ADD` — новый слой образа. Используйте `.dockerignore`, чтобы исключить мусор:
    ```
    .git
    node_modules
    __pycache__
    *.log
    ```

!!! tip "Кэширование слоёв: сначала зависимости, потом код"
    | Порядок | Что копировать |
    |---------|---------------|
    | 1 | `requirements.txt` / `package.json` |
    | 2 | `RUN pip install` / `npm install` |
    | 3 | `COPY src/` / остальной код |

!!! warning "Не используйте ADD для скачивания из интернета"
    `ADD` по URL — недетерминировано, не кэшируется. Используйте `RUN curl` + `COPY`.
    ``` yaml
    RUN curl -L -o /tmp/file.tar.gz https://example.com/file.tar.gz
    COPY file.tar.gz /app/
    ```

## Использование и настройка дашборда Lens

!!! tip "Скачивание"
    Перейдите на [k8slens.dev/download](https://k8slens.dev/download) и скачайте дистрибутив под свою ОС.

!!! info "Документация"
    Шаги по настройке и управлению Lens: [docs.k8slens.dev/getting-started/activate-lens-desktop/](https://docs.k8slens.dev/getting-started/activate-lens-desktop/)

!!! note "Подключение кластера через kubeconfig"
    1. Добавить кластер: "Add Clusters from kubeconfig"
    2. Скопировать конфиг:
       ```bash
       cat ~/.kube/config
       ```
    3. Сохранить в `/kuber/config` в Lens

## Управление контекстами в рамках оркестрации

!!! info "Что такое контекст"
    Набор параметров, которые определяют, с каким кластером Kubernetes и пространством имён вы работаете.

!!! tip "Команды для работы с context"
    | Команда | Описание |
    |---------|----------|
    | `kubectl config get-contexts` | Список контекстов (активный помечен `*`) |
    | `kubectl config use-context <name>` | Переключиться на нужный контекст |
    | `kubectl config current-context` | Показать текущий контекст |

## Лучшие практики по настройке конфигураций в Kubernetes

### Общие советы

!!! tip "Используйте самую свежую стабильную версию API"
    Старые версии API устаревают и перестают работать. Проверяйте актуальную версию через:
    ```bash
    kubectl api-resources
    kubectl explain pod.spec.containers
    ```

!!! warning "Храните конфигурацию в системе контроля версий"
    Никогда не применяйте манифесты напрямую с локальной машины. Git — страховка для отката, сравнения и пересоздания кластера.

!!! info "Прочие советы по оформлению"
    | Практика | Рекомендация |
    |----------|--------------|
    | Формат | YAML вместо JSON. Булевы: только `true`/`false`; `yes`, `no` — в кавычки |
    | Минимализм | Не задавайте значения по умолчанию |
    | Группировка | Связанные объекты (Deployment + Service + ConfigMap) — в одном манифесте |
    | Аннотации | Добавляйте `kubernetes.io/description` — комментарий сохраняется в API |

### Рабочие нагрузки (поды, Deployment, Job)

!!! warning "Не создавайте «голые» поды (Naked Pods) в production"
    Поды без контроллера не перепланируются автоматически при падении узла. Всегда используйте контроллер:

    | Тип | Для чего | Особенности |
    |-----|----------|-------------|
    | **Deployment** | Постоянные сервисы | ReplicaSet + RollingUpdate + откат |
    | **Job** | Однократные задачи | Перезапуск при ошибке, отчёт об успехе |

### Service и сеть

!!! tip "Создавайте Service до рабочих нагрузок"
    Kubernetes прокидывает переменные окружения Service'ов в поды только для уже существующих Service'ов:
    ```
    FOO_SERVICE_HOST=<хост>
    FOO_SERVICE_PORT=<порт>
    ```

!!! warning "Избегайте hostPort и hostNetwork"
    Привязывают поды к узлам, усложняют планировщик и масштабирование.

    | Задача | Решение |
    |--------|---------|
    | Локальная отладка | `kubectl port-forward deployment/web 8080:80` |
    | Внешний доступ | Service типа NodePort |
    | DNS внутри кластера | `curl http://my-service.default.svc.cluster.local` |

!!! info "Headless Service"
    Для прямого обращения к подам без балансировки — `clusterIP: None`. DNS возвращает список IP всех подов напрямую.

### Лейблы

!!! note "Используйте осмысленные и стандартные лейблы"

    | Назначение | Команда |
    |------------|---------|
    | Поиск подов | `kubectl get pods -l app=myapp` |
    | Массовое удаление | `kubectl delete pod -l phase=test` |
    | Отладка (временно отключить под) | `kubectl label pod mypod app-` |

    ```yaml
    labels:
      app.kubernetes.io/name: myapp
      app.kubernetes.io/component: web
      tier: frontend
      phase: test
    ```


### Советы по kubectl

| Команда | Описание |
| --------- | ---------- |
| `kubectl apply -f configs/ --server-side` | Применить директорию целиком |
| `kubectl get pods -l app=myapp` | Получить поды по лейблу |
| `kubectl delete pod -l phase=test` | Массовое удаление подов |
| `kubectl scale deployment -l app=myapp` | Масштабировать по лейблу |
| `kubectl rollout undo deployment -l phase=test` | Откатить по лейблу |
| `kubectl logs -l app=myapp --all-containers -f` | Логи всех контейнеров |
| `kubectl create deployment webapp --image=nginx` | Быстрый тестовый Deployment |
| `kubectl expose deployment webapp --port=80` | Быстрый тестовый Service |
| `kubectl rollout restart deployment my-app` | Мягкий перезапуск |
| `kubectl scale deployment my-app --replicas=0` | Временная остановка подов |

!!! tip "Pod Disruption Budgets (PDB)"
    Используйте PDB, чтобы избежать прерываний работы при дрейне узлов.
