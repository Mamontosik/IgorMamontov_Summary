### Web

- **80/TCP** — HTTP (нешифрованный веб). Применение: веб‑сайты, API (dev/test).
- **443/TCP** — HTTPS (HTTP over TLS). Применение: защищённый веб, API (prod).

**Проверка:** `ss -ltn sport = :80` / `ss -ltn sport = :443` / `curl -I http(s)://<host>`

### Доступ / администрирование

- **22/TCP** — SSH / SFTP (удалённый доступ, git over SSH).
- **3389/TCP** — RDP (удалённый рабочий стол Windows).
- **23/TCP** — Telnet (deprecated, небезопасно).

**Проверка:** `ss -ltn sport = :22` / `ssh -v user@host`

### Почта

- **25/TCP** — SMTP (transfer between mail servers).
- **587/TCP** — Submission (SMTP client submission).
- **465/TCP** — SMTPS (legacy SMTP over TLS).
- **110/TCP** — POP3.
- **143/TCP** — IMAP.
- **993/TCP** — IMAPS.
- **995/TCP** — POP3S.

**Проверка:** `ss -ltn sport = :25` / `openssl s_client -starttls smtp -connect host:587`

### Сетевая инфраструктура (DNS/DHCP/NTP)

- **53/UDP, TCP** — DNS (UDP для резолвинга, TCP для zone transfer / больших ответов).
- **67/68/UDP** — DHCP (server/client).
- **123/UDP** — NTP.

**Проверка:** `dig @<dns> example.com` / `tcpdump -n udp port 67 or udp port 68 -c 20`

### Каталог / аутентификация

- **389/TCP,UDP** — LDAP.
- **636/TCP** — LDAPS.
- **3268/3269/TCP** — AD Global Catalog.

**Проверка:** `ldapsearch -H ldap://host -x` / `ss -ltn sport = :389`

### Базы данных

- **3306/TCP** — MySQL / MariaDB.
- **5432/TCP** — PostgreSQL.
- **1433/TCP** — MS SQL Server.
- **27017/TCP** — MongoDB.
- **6379/TCP** — Redis (in‑memory).

**Проверка:** `ss -ltn | grep 5432` / `pg_isready -h host`

### Messaging / очереди

- **5672/TCP** — AMQP (RabbitMQ).
- **15672/TCP** — RabbitMQ Management UI.
- **9092/TCP** — Kafka broker (PLAINTEXT).
- **9093/TCP** — Kafka broker (SSL).
- **2181/TCP** — ZooKeeper (координация для старых кластеров).

**Проверка:** `ss -ltn | grep 9092` / `curl http://host:15672` (если management доступен)

### Хранилище / файловые сервисы

- **445/TCP** — SMB / CIFS (Windows shares, Samba).
- **2049/TCP,UDP** — NFS.
- **5000/TCP** — Docker Registry (локальный).

**Проверка:** `rpcinfo -p host` / `ss -ltn | grep 2049`

### Оркестрация / кластерные сервисы

- **6443/TCP** — Kubernetes API server.
- **10250/TCP** — Kubelet API.
- **2379/TCP** — etcd client.
- **2380/TCP** — etcd peer.

**Проверка:** `kubectl get nodes --server=https://host:6443` / `ss -ltn | grep 6443`

### Мониторинг / логирование / визуализация

- **9090/TCP** — Prometheus.
- **3000/TCP** — Grafana.
- **5601/TCP** — Kibana.
- **161/162/UDP** — SNMP (agent/trap).

**Проверка:** `ss -ltn | grep 9090` / `curl http://host:3000`

### Прочее / вспомогательные

- **9418/TCP** — Git (git://).
- **Ephemeral** — Client ephemeral ports (динамические) — TCP/UDP.

**Проверка:** `ss -tn sport :ephemeral-range` (проверить на хосте диапазон ephemeral портов)
