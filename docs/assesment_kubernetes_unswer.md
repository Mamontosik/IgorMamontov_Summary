## Kubernetes (K8s) — автоматизация развёртывания и управления контейнеризированными приложениями

### Основные компоненты Kubernetes

| Компонент | Описание | Полезные советы |
|---|---|---|
| Control Plane (master) | Набор сервисов: kube-apiserver, etcd, controller-manager, scheduler | HA для control plane, бэкапы etcd, ограничить доступ к API |
| kube-apiserver | Входная точка API кластера | За LB, включить аутентификацию/авторизацию, логирование |
| etcd | Хранилище метаданных кластера | Отдельные диски/узлы, регулярные бэкапы, мониторинг I/O |
| kube-scheduler | Назначает Pod на Node | Использовать affinity/taints/tolerations, тестировать policy |
| kube-controller-manager | Контроллеры состояния (Replica, Node и т.д.) | Мониторить failed controllers, выделить ресурсы |
| cloud-controller-manager | Интеграция с облачными провайдерами (если есть) | Ограничивать права, использовать минимальные IAM роли |
| kubelet | Агент на Node, запускает контейнеры | Минимизировать привилегии, следить за ресурсами node |
| kube-proxy | Реализует Service через iptables/ipvs | Предпочтительно ipvs для высокой нагрузки |
| Container Runtime | containerd/CRI-O/другой runtime | Использовать поддерживаемый runtime, безопасные конфиги |
| CNI (сеть) | Сетевая подсистема (Calico/Flannel/Weave) | Выбрать CNI по требованиям (policy/mtu/perf), тестировать обновления |
| Ingress Controller | Внешний HTTP(S) доступ (NGINX/Traefik) | TLS на Ingress, healthchecks, rate limiting |
| scheduler extender / custom controllers | Доп. логика планирования/операторы | Вынести сложную логику в отдельные контроллеры/CRD |

#### Сеть: поведение Pod/Container и потоки трафика (по namespace)

| Направление трафика | Как это работает (сеть) | Ключевые объекты |
|---|---|---|
| Pod → Pod (внутри namespace) | Прямой обмен по Pod IP внутри cluster network; пакеты идут через veth → CNI | veth pair, CNI routing/bridge/overlay; нет NAT между Pod |
| Pod → Pod (между namespace) | Аналогично, маршрутизация по Pod IP; DNS/FQDN зависит от namespace | NetworkPolicy может блокировать/разрешать трафик по namespace/label |
| Pod → Service (ClusterIP, внутри namespace) | Запрос на ClusterIP DNAT'ится kube-proxy в один из целевых Pod | Service (ClusterIP), kube-proxy (iptables/ipvs), DNS резолвинг |
| Pod → Service (в другой namespace) | Используется FQDN svc.namespace.svc.cluster.local или указание namespace | Убедиться в правильном DNS запросе и разрешениях NetworkPolicy |
| Ingress → Service → Pod (входящий HTTP(S)) | Ingress controller принимает внешний трафик, делает TLS termination и форвардит на Service → Pod | Ingress controller, TLS certs, healthchecks; обычно reverse‑proxy |
| External → NodePort / LoadBalancer → Pod | Внешний трафик приходит на NodePort или облачный LB → Node → kube-proxy DNAT → Pod | NodePort открывает порт на Node; LB направляет на Node/целевые группы |
| Pod → External (egress) | Под отправляет пакет наружу; обычно SNAT выполняется на Node или egress gateway | Egress контролируется NAT, firewall или egress gateway |
| DNS внутри кластера | CoreDNS (ClusterIP) резолвит service/pod FQDN | CoreDNS критичен для discovery; следить за его ресурсами/health |
| NetworkPolicy (ingress/egress) | Политики ограничивают потоки на уровне Pod селекторов | По умолчанию отсутствие политики = открыто; политики по умолчанию блокируют |
| HostNetwork / HostPort | Pod использует сетевой стек ноды или занимает порт хоста | Обходит kube-proxy; возможны конфликты портов и повышенные риски безопасности |
| Overlay / MTU | Меж‑Node трафик инкапсулируется (vxlan/ipip); MTU влияет на производительность | Неправильный MTU → фрагментация/потеря пакетов; тестировать MTU при развертывании |
| Multicast / Broadcast | Обычно не проходит через overlay между Node | Discovery, зависящий от broadcast, может не работать без L2‑поддержки |
| Egress Gateway / NAT | Управление исходящим трафиком через выделенный gateway | Позволяет контролировать исходящие IP и политики безопасности |

### Список команды kubectl

| Команда | Описание | Параметры (описание) | Пример |
|---|---|---|---|
| `kubectl get` | Список ресурсов | `-n <ns>` — указать namespace; `-o wide` — показать дополнительные колонки (NODE, IP); `-o yaml/json` — вывести в структурированном формате | `kubectl get pods -n prod -o wide` |
| `kubectl describe` | Детали ресурса и события | указывать тип и имя ресурса `pod|svc|node <name>`; выводит события и состояние | `kubectl describe pod myapp -n staging` |
| `kubectl logs` | Логи контейнера | `-f` — follow (подписка); `-c <container>` — выбрать контейнер в Pod; `--since=<duration>` — показать логи за период | `kubectl logs -f pod/myapp -c web` |
| `kubectl exec` | Выполнить команду в контейнере | `-it` — интерактивно + TTY; указывайте `pod -- <cmd>` | `kubectl exec -it pod/myapp -- /bin/sh` |
| `kubectl apply` | Применить манифест (idempotent) | `-f <file|dir>` — файл или директория манифестов; `--prune` — удалять ресурсы не описанные (в сочетании с селектором) | `kubectl apply -f k8s/` |
| `kubectl delete` | Удалить ресурс | `-f` — файл/директория; `--grace-period=<s>` — время завершения; `--force` — принудительное удаление | `kubectl delete -f deploy.yaml` |
| `kubectl rollout` | Статус/undo/restart deployment | подкоманды: `status` — проверить rollout; `undo` — откат; `restart` — перезапуск rollout | `kubectl rollout status deploy/myapp` |
| `kubectl scale` | Масштабирование | `--replicas=<n>` — задать количество реплик | `kubectl scale deploy myapp --replicas=3` |
| `kubectl top` | Метрики (metrics-server) | `pods|nodes` — выбрать уровень, `-n <ns>` — namespace | `kubectl top pod -n prod` |
| `kubectl port-forward` | Проброс порта локально | `local:remote` — формат проброса; можно указывать `pod/<name>` или `svc/<name>` | `kubectl port-forward svc/my-svc 8080:80` |
| `kubectl get events` | События в namespace | `-n <ns>` — namespace; `--sort-by=.metadata.creationTimestamp` — сортировать по времени | `kubectl get events -n default` |
| `kubectl config` | Управление kubeconfig/контекстами | `use-context <name>` — переключить контекст; `view` — показать конфигурацию; `set-context` — создать/изменить контекст | `kubectl config use-context prod` |
| `kubectl cp` | Копировать файлы в/из контейнера | синтаксис `<src> <dest>`; `pod:/path` и локальные пути; поддерживает папки | `kubectl cp pod/myapp:/app/log.txt ./log.txt` |
| `kubectl auth` | Проверка RBAC (can-i) | `can-i <verb> <resource> [--as=<user>]` — проверить права для субъекта | `kubectl auth can-i create pods --as system:serviceaccount:default:sa` |
| `kubectl apply --server-side` | Server-side apply | `-f` — файл манифеста; делает server-side конфигурационное слияние и ownership полей | `kubectl apply --server-side -f deploy.yaml` |
| `kubectl explain` | Описание полей ресурса | указывать путь полей `kubectl explain deployment.spec` — показать описание и типы полей | `kubectl explain pod.spec.containers` |
| `kubectl wait` | Ожидание условия | `--for=condition=<cond>` — условие (например available); `--timeout=<duration>` — таймаут ожидания | `kubectl wait --for=condition=available deployment/myapp --timeout=120s` |
| `kubectl label` / `kubectl annotate` | Управление метками/аннотациями | `-n <ns>` — namespace; `key=value` — формат метки; `--overwrite` для перезаписи | `kubectl label pod mypod env=prod` |
| `kubectl patch` | Частичное изменение ресурса | `--type=json/merge -p '<patch>'` — указать тип патча и содержимое; поддерживает JSON/merge/strategic | `kubectl patch deploy myapp -p '{"spec":{"replicas":3}}'` |

### Список ошибок и рекомендации по исправлению

| Ошибки | Причины | Диагностика | Рекомендации по исправлению |
|---|---:|---|---|
| CrashLoopBackOff | Ошибка запуска приложения | `kubectl describe pod`, `kubectl logs -p` | Проверить логи, исправить образ/entrypoint, добавить startupProbe |
| ImagePullBackOff / ErrImagePull | Неверный image/tag или auth | `kubectl describe pod` (events) | Исправить image, добавить imagePullSecrets, проверить registry |
| Pending | Нет доступных Node (ресурсы/taints) | `kubectl describe pod` (events), `kubectl get nodes` | Увеличить capacity, снять taints, изменить requests |
| OOMKilled / Exit 137 | Превышен лимит памяти или OOM killer | `kubectl describe pod`, `kubectl top pod`, `dmesg` | Увеличить limits/requests, оптимизировать память, проверить node OOM |
| NodeNotReady | Проблемы с kubelet/сетью/диском | `kubectl get nodes`, `kubectl describe node <name>`, `journalctl -u kubelet` | Проверить kubelet, сеть, диск; drain и ремонт node |
| DNS Failures | CoreDNS упал/перегружен или NetworkPolicy блокирует | `kubectl get pods -n kube-system`, `kubectl logs -n kube-system coredns` | Перезапустить CoreDNS, увеличить resources, проверить NetworkPolicy |
| Probe failures | Неправильные readiness/liveness настройки | `kubectl describe pod`, `kubectl logs` | Исправить путь/порт/таймауты probe или увеличить initialDelaySeconds |
| High restarts | Нестабильность приложения/циклические ошибки | `kubectl get pods --sort-by=.status.restartCount`, `kubectl logs -p` | Анализ логов, изменить retry/backoff, проверить зависимости |
| Evicted | Давление на ресурсы (disk/memory) | `kubectl describe pod` (reason Evicted), `kubectl describe node` | Очистить диск, добавить ресурсы, настроить eviction thresholds |
| API server not reachable | LB/network/auth проблемы | `kubectl cluster-info`, `curl -k https://<apiserver>` | Проверить LB, firewall, certs; посмотреть логи apiserver |
| RBAC / auth errors | Неправильные роли/контексты | `kubectl auth can-i ...`, `kubectl config view` | Исправить роли/rolebindings, обновить kubeconfig |
| Certificate / TLS errors | Просроченные или неверные сертификаты | `kubectl get csr`, `openssl s_client -connect apiserver:6443` | Обновить certs, пересоздать kubeconfig, проверить time sync |

### Типы ресурсов Kubernetes

| Ресурс | Описание | Виды применения |
|---|---|---|
| Pod | Минимальная единица деплоя — один/несколько контейнеров, общий сетевой namespace и тома | Запуск приложений, sidecar, кратковременные задачи |
| Deployment | Декларативное управление ReplicaSet, обеспечивает rollout/rollback | Бесстейтные сервисы, rolling updates, CI/CD |
| ReplicaSet | Поддерживает заданное число реплик Pod | Обеспечение необходимого количества экземпляров |
| StatefulSet | Управляет stateful-приложениями с устойчивой идентичностью и PVC | БД, очереди, stateful сервисы (Postgres, Kafka) |
| DaemonSet | Запускает Pod на каждой (или выбранных) ноде | Лог‑агенты, мониторинг, сетевые плагины |
| Job | Одноразовая задача с гарантией завершения | Batch‑задания, миграции, миграционные скрипты |
| CronJob | Планирование Job по расписанию | Регулярные бэкапы, отчёты, периодические задачи |
| Service | Абстракция доступа к набору Pod (ClusterIP/NodePort/LoadBalancer) | Внутренний/внешний доступ к приложениям |
| Ingress | Правила внешнего HTTP(S) доступа, TLS termination | Маршрутизация внешнего веб‑трафика |
| ConfigMap | Хранение неконфиденциальных конфигураций | Передача конфигураций в Pod без пересборки образа |
| Secret | Хранение конфиденциальных данных (пароли, ключи) | Передача секретов в Pod/CSI секреты |
| Namespace | Логическая изоляция ресурсов и политик | Мульти‑тенантность, разделение окружений |
| PersistentVolume (PV) | Физический ресурс хранилища в кластере | Долговременное хранение для Pod |
| PersistentVolumeClaim (PVC) | Запрос тома от Pod (связь с PV) | Динамическое/статическое выделение хранилища |
| StorageClass | Политика динамического provision‑а томов | Настройка типов дисков и параметров provisioner |
| Node | Физическая или виртуальная нода в кластере | Размещение Pod, capacity planning |
| ServiceAccount | Идентификатор для Pod при обращении к API | RBAC для приложений, автоматические токены |
| Role / ClusterRole | Набор прав (namespaced / cluster) | Ограничение доступа к API |
| RoleBinding / ClusterRoleBinding | Привязка ролей к субъектам | Назначение прав пользователям/сервисам |
| CRD (CustomResourceDefinition) | Расширение API пользовательскими ресурсами | Операторы, кастомные контроллеры |
| HPA (HorizontalPodAutoscaler) | Автомасштабирование Pod по метрикам | Автоскейлинг по CPU/метрикам/custom |
| VPA (VerticalPodAutoscaler) | Автоматическая корректировка ресурсов контейнера | Подходит для stateful/legacy приложений |
| PodDisruptionBudget (PDB) | Ограничение одновременных voluntary disruptions | Поддержание доступности при обновлениях/maintenance |

## Механизм управления трафиком в K8s

| Задача | Что делает Kubernetes | Фрагмент манифеста | Команды для проверки |
|---|---|---|---|
| Балансировка трафика внутри кластера | Service (ClusterIP) + kube-proxy распределяет входящие запросы на Pod | Service: `kind: Service\nspec:\n  type: ClusterIP\n  selector: {app: myapp}\n  ports: - port: 80 targetPort: 8080` | `kubectl get svc`, `kubectl describe svc mysvc`, `ss -tuna` на Node |
| Внешний доступ / маршрутизация HTTP(S) | Ingress (IngressController) роутит внешний HTTP(S) → Service; TLS termination | Ingress: `kind: Ingress\nmetadata:\n  annotations: {nginx.ingress.kubernetes.io/rewrite-target: /}\nspec:\n  rules: - host: example.com` | `kubectl get ingress`, `kubectl describe ingress`, `kubectl logs -n ingress-nginx pod/...` |
| NodePort / LoadBalancer | Экспонирование сервиса наружу через порт на Node или облачный LB | Service типа `NodePort` или `LoadBalancer` | `kubectl get svc -o wide`, проверка облачного LB в провайдере |
| Сессии / сохранение состояния (sticky) | Service `sessionAffinity: ClientIP` или ingress annotations | `spec:\n  sessionAffinity: ClientIP` | `kubectl describe svc mysvc` |
| Контроль готовности трафика (avoid sending to unhealthy Pods) | readinessProbe предотвращает отправку трафика в неполадки | `readinessProbe:\n  httpGet: {path: /health, port: 8080}\n  initialDelaySeconds: 10` | `kubectl get pods -o wide`, `kubectl describe pod` (поля Readiness) |
| Отказоустойчивость и масштабирование | Deployment + ReplicaSet + HPA масштабируют количество Pod по метрикам | HPA: `apiVersion: autoscaling/v2\nkind: HorizontalPodAutoscaler\nspec:\n  scaleTargetRef: {kind: Deployment,name: myapp}\n  minReplicas: 2 maxReplicas: 10 metrics: ...` | `kubectl get hpa`, `kubectl top pods`, `kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=70` |
| Стабильность обновлений (rolling / canary) | Deployment strategy: RollingUpdate / Canary через additional controllers | `strategy:\n  type: RollingUpdate\n  rollingUpdate: {maxUnavailable: 1, maxSurge: 1}` | `kubectl rollout status deploy/myapp`, `kubectl rollout undo deploy/myapp` |
| Управление эвакуацией нод и drain | `kubectl cordon` / `kubectl drain` перемещают Pod для обслуживания Node | N/A (команды) | `kubectl cordon nodename`, `kubectl drain nodename --ignore-daemonsets --delete-local-data` |
| Контроль допустимого прерывания Pod | PodDisruptionBudget ограничивает количество одновременных voluntary disruptions | `apiVersion: policy/v1\nkind: PodDisruptionBudget\nspec:\n  minAvailable: 50%\n  selector: {matchLabels:{app: myapp}}` | `kubectl get pdb`, `kubectl describe pdb` |
| Распределение по топологии / отказоустойчивость | PodTopologySpread / affinity/anti-affinity распределяют Pod по зонам/нодам | `topologySpreadConstraints` или `affinity: podAntiAffinity` | `kubectl describe deployment`, `kubectl get pods -o wide` |
| Управление ресурсами и QoS | requests/limits + QoS классы дают планирование и поведение при давлении | `resources:\n  requests: {cpu: 100m, memory: 128Mi}\n  limits: {cpu: 500m, memory: 512Mi}` | `kubectl top pod`, `kubectl describe node` (pressure events) |
| Предотвращение и управление OOM/evictions | requests/limits + Node resource management + eviction thresholds | настройки kubelet/eviction в Node config | `kubectl get events --field-selector reason=Evicted`, `dmesg` на Node |
| Маршрутизация исходящего трафика (egress) | SNAT на Node или egress‑gateway (service mesh) для контроля исхода | Egress через NAT или Istio egress gateway | `kubectl get svc -n istio-system`, проверка внешнего IP через `curl ifconfig.me` из Pod |
| Сетевая сегментация и политика доступа | NetworkPolicy управляет ingress/egress между Pod | `kind: NetworkPolicy\nspec:\n  podSelector: {matchLabels:{app: frontend}}\n  ingress: - from: - podSelector:{matchLabels:{role: api}}` | `kubectl get networkpolicy`, `kubectl describe networkpolicy` |
| Высокая производительность сервисов | kube-proxy mode ipvs, CNI tuning (MTU), service endpoints balancing | Включить ipvs: kube-proxy config, настроить CNI MTU | `kubectl get configmap kube-proxy -n kube-system -o yaml`, `ss -tn` / `ip route` |
| Грейсфул‑шутдаун и рестарт | graceful termination via preStop, terminationGracePeriodSeconds | `lifecycle:\n  preStop: {exec: {command: [\"/bin/sh\",\"-c\",\"sleep 10\"]}}\nterminationGracePeriodSeconds: 30` | `kubectl delete pod mypod --grace-period=30` |

!!! note
    Примечание: многие настройки влияют совместно (например, readinessProbe + Service + NetworkPolicy). Для диагностики часто используются: `kubectl describe`, `kubectl logs`, `kubectl get events`, `kubectl top`, `kubectl get endpoints` и сетевые трассировки (`tcpdump`/`curl`/`dig`) внутри/снаружи Pod.
