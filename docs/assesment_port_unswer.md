# Порты — краткий справочник

Кратко: таблица для быстрого поиска; раскрываемые секции — для деталей и проверок.

## Список популярных портов
| Порт | Протокол | Сервис | Применение | Быстрая проверка |
|---:|:---:|---|---|---|
| 80 | TCP | HTTP | Веб (dev/test) | `ss -ltn sport = :80` |
| 443 | TCP | HTTPS | Веб (prod/TLS) | `ss -ltn sport = :443` |
| 22 | TCP | SSH | Админ/ssh/git | `ss -ltn sport = :22` |
| 53 | UDP/TCP | DNS | Резолвинг / zone transfer | `dig @<dns> example.com` |
| 67/68 | UDP | DHCP | Авто IP (DORA) | `tcpdump -n udp port 67 or 68 -c 5` |
| 3306 | TCP | MySQL | БД | `ss -ltn | grep 3306` |
| 5432 | TCP | PostgreSQL | БД | `ss -ltn | grep 5432` |
| 6443 | TCP | K8s API | Kubernetes API | `kubectl get nodes --server=https://host:6443` |
| 2379/2380 | TCP | etcd | KV store (k8s) | `ss -ltn | grep 2379` |
| 9090 | TCP | Prometheus | Monitoring | `ss -ltn | grep 9090` |
| 3000 | TCP | Grafana | Dashboards | `curl -I http://host:3000` |
| 5672 | TCP | RabbitMQ | Messaging | `ss -ltn | grep 5672` |
| 27017 | TCP | MongoDB | NoSQL | `ss -ltn | grep 27017` |
| 6379 | TCP | Redis | In‑memory | `redis-cli -h host ping` |
| 445 | TCP | SMB | Windows shares | `smbclient -L //host` |

---

<details>
<summary><strong>Web & TLS</strong></summary>

- 80/TCP — HTTP (dev/test). Check: `curl -I http://<host>`<br>  
- 443/TCP — HTTPS (prod/TLS). Check: `openssl s_client -connect host:443 -servername host`

</details>

<details>
<summary><strong>Администрирование</strong></summary>

- 22/TCP — SSH. Check: `ssh -v user@host`<br>  
- 3389/TCP — RDP (Windows)<br>  
- 23/TCP — Telnet (не рекомендуется)

</details>

<details>
<summary><strong>DNS / DHCP / NTP</strong></summary>

- 53/UDP,TCP — DNS. Check: `dig @<dns> example.com`<br>  
- 67/68/UDP — DHCP (DORA). Check: `tcpdump -n udp port 67 or 68 -c 10`<br>  
- 123/UDP — NTP. Check: `ntpq -p host`

</details>

<details>
<summary><strong>Databases</strong></summary>

- 3306/TCP — MySQL/MariaDB (`ss -ltn | grep 3306`)<br>  
- 5432/TCP — PostgreSQL (`pg_isready -h host`)<br>  
- 27017/TCP — MongoDB (`ss -ltn | grep 27017`)<br>  
- 6379/TCP — Redis (`redis-cli -h host ping`)

</details>

<details>
<summary><strong>Messaging & Orchestration</strong></summary>

- 5672/TCP — RabbitMQ (AMQP)<br>  
- 15672/TCP — RabbitMQ UI (`curl -I http://host:15672`)<br>  
- 9092/9093/TCP — Kafka brokers <br> 
- 6443/10250/TCP — Kubernetes API / kubelet

</details>

<details>
<summary><strong>Storage & Fileservices</strong></summary>

- 445/TCP — SMB/CIFS (`smbclient -L //host`) <br> 
- 2049/TCP — NFS (`ss -ltn | grep 2049`) <br> 
- 5000/TCP — Local Docker registry (`curl -I http://host:5000/v2/`)

</details>

---

## Рекомендации по использованию
- Для быстрого локального аудита: `ss -ltn` + `ss -lun`.  
- Для сетевого трафика/пакетов: `tcpdump -n -i <iface> port <num> -c 100`.  
- Используйте `nmap` аккуратно (может считаться сканированием).