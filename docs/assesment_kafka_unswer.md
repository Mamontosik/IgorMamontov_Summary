## Kafka — распределенная система обмена сообщениями

### Компоненты

| Компонент | Короткое описание | Почему важно |
|---|---|---|
| Broker | Процесс сервера, хранит партиции и обслуживает запросы продюсеров/консьюмеров | Масштабирование, отказоустойчивость через репликацию |
| Topic | Логическая категория сообщений (например, "orders") | Организация потоков, изоляция по смыслу |
| Partition | Append‑only лог внутри topic; единица параллелизма | Параллелизм и порядок внутри партиции |
| Producer | Публикует сообщения в topic (может настраивать acks/retries/batching) | Управление надежностью публикации |
| Consumer | Читает сообщения по offset; в составе consumer group распределяет нагрузку | Горизонтальное масштабирование обработчиков |
| Consumer Group | Группа консьюмеров, читающая topic совместно | Каждая партиция назначается одному члену группы |

!!! note "Производительность"
 Правильное планирование числа партиций и репликации критично: слишком мало партиций ограничит параллелизм, а слишком много — создаст накладные расходы на метаданные.

### Репликация и роли

| Тема | Кратко | На что смотреть |
|---|---|---|
| Replication | Копирование партиций на несколько брокеров (leader/followers) | ISR, replication lag, under_replicated_partitions |
| Leader / Follower | Лидер обслуживает записи/чтение, followers реплицируют | Election, failover, impact on availability |
| Offset | Позиция в партиции — целое число | Управление offset'ами важно для семантики доставки |

### Политики хранения

| Политика | Описание | Примечания |
|---|---|---|
| Retention | Удаление сегментов по времени или объёму | log.retention.hours / log.retention.bytes |
| Log compaction | Сохранение последней записи для каждого ключа (compaction) | Используется для хранения состояния (compact topics) |

### Координация и доп. сервисы

| Сервис | Назначение | Комментарий |
|---|---|---|
| Zookeeper / KRaft | Координация метаданных (ZK — старые версии, KRaft — newer) | Миграция ZK → KRaft требует планирования |
| Schema Registry | Хранение/валидация схем (Avro/JSON/Protobuf) | Облегчает эволюцию форматов |
| Exactly‑once (EOS) | Транзакционные механизмы для "ровно один раз" | Сложнее в настройке; влияет на throughput |

#### Краткая таблица этапов

| Шаг | Что происходит | Что показывать/проверять |
|---|---|---|
| Produce | Producer отправляет запись в topic → партиция (append) | produce rate, request latency, errors (RecordTooLarge) |
| Persist | Leader пишет в лог, репликация followers | leader, ISR, replication lag, under_replicated_partitions |
| Consume | Consumer читает по offset и коммитит | consumer lag, offsets, rebalances |
| Coordination | Контроллер (ZK/KRaft) управляет лидерами | leader elections, controller status |

### Популярные ошибки и как их диагностировать/исправлять

| Ошибка / симптом | Типичная причина | Диагностика (команды / признаки) | Быстрое решение |
|---|---|---|---|
| "Leader not available" | Лидер партиции недоступен / контроллер проблемен | `kafka-topics.sh --describe --topic <topic>` (leader = -1); controller/broker logs | Перезапустить broker, проверить контроллер; preferred‑replica‑election при необходимости |
| Under‑replicated partitions (URP) | Реплики отстают или недоступны | `kafka-topics.sh --describe` (under_replicated_partitions), broker logs, метрики | Проверить IO/сеть; восстановить реплики; изменить настройки replica.fetch* при необходимости |
| Consumer lag (большая задержка) | Консьюмер не успевает, горячие партиции | `kafka-consumer-groups.sh --describe --group <group>`; Prometheus consumer lag | Увеличить parallelism, оптимизировать обработку, увеличить партиции/consumer'ов |
| OffsetOutOfRangeException | Чтение несуществующего offset (retention) | Логи консьюмера; `kafka-consumer-groups.sh --describe` | Сброс offsets (`--reset-offsets --to-earliest|--to-latest`) и execute |
| RecordTooLargeException | Сообщение больше max.message.bytes | Broker/producer logs | Увеличить message.max.bytes / max.request.size или разбить payload |
| Broker OOM / GC pauses | Недостаточно памяти JVM / плохие GC настройки | Broker logs, JMX GC metrics | Настроить heap/GC, перераспределить нагрузку |
| Disk full | Недостаточно места на диске | Broker logs; `df -h` на broker | Очистка/увеличение диска, настроить retention/log.dirs |
| AuthN/AuthZ (SASL/ACL) | Неправильные сертификаты или ACL | Broker logs: AuthenticationException/AuthorizationException | Проверить сертификаты/ACLs, kafka-acls.sh |
| ZooKeeper / Controller issues | ZooKeeper недоступен или контроллер вне строя | ZK logs, controller logs | Восстановить ensemble; проверить quorum; перезапустить сервисы |
| NotEnoughReplicas / NotEnoughReplicasAfterAppend | Недостаточно реплик в ISR для acks=all | Producer errors; проверка ISR | Добавить реплики или изменить политику acks/min.insync.replicas |
| ProducerFenced / Transactional errors | Конфликт transactional.id | Producer logs | Использовать уникальные transactional.id; корректно закрывать продюсер |
| Frequent rebalances | Нестабильные консьюмеры / heartbeat timeout | Consumer logs, frequent rebalance messages | Увеличить session.timeout.ms/heartbeat.interval.ms; оптимизировать lifecycle |
| Schema Registry incompatible | Невыполнена политика совместимости схем | Producer errors; schema-registry logs | Привести схему в совместимое состояние или обновить policy |
| Too many open files (EMFILE) | OS limits too low | Broker logs; `lsof` | Увеличить ulimit -n / systemd LimitNOFILE |
| Log segment corruption | Повреждённые сегменты лога | Broker logs; `kafka-dump-log.sh` | Восстановить из бэкапа или аккуратно удалить повреждённые сегменты |
| Network RequestTimedOut | Сеть/MTU/перегрузка | Client/broker RequestTimedOut | Проверить сеть, увеличить timeouts, оптимизировать batching |
