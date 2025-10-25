# Справочник название для DevOps инженера

## Контейнеризация и оркестрация
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| Docker | Контейнеризация образов и runtime | Локальная dev, CI jobs, изоляция приложений | `docker build -t app .` `docker run -p 80:80` | Dockerfile, /etc/docker/daemon.json | containerd, Podman, Docker Compose, Kubernetes |
| Helm | Пакетный менеджер K8s (charts) | Шаблонизация и деплой приложений в K8s | `helm install myapp ./chart` | Chart.yaml, values.yaml | Kubernetes, Helmfile, ArgoCD |
| Ingress | HTTP(S) маршрутизация в K8s | TLS termination, host/path routing | `kubectl get ingress` | ingress manifests | NGINX Ingress, Traefik, Istio Gateway |
| Egress | Контроль исходящего трафика из кластера | Egress policies, NAT, egress gateway | `kubectl get networkpolicy` | NetworkPolicy, EgressGateway cfg | Istio, Calico, egress proxies |
| cgroups | Ограничение ресурсов процессов | Ограничение CPU/memory/io для контейнеров/сервисов | `systemd-run --scope --slice=...` | /sys/fs/cgroup/ | systemd, Docker, Kubernetes |
| Linux namespace | Изоляция ресурсов (PID, NET, MNT и т.д.) | Контейнеры, сетевые неймспейсы | `ip netns add ns1` `unshare --pid` | container configs | runc, LXC, Docker, Kubernetes |

## Сетевые технологии
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| TCP | Надёжный транспортный протокол | HTTP, DB-протоколы | `ss -t` `tcpdump tcp port 80` | sysctl network params | iproute2, tcpdump, Wireshark |
| TCP/IP | Стек сетевых протоколов | Маршрутизация, обмен пакетами | `ip route` `ping` | /etc/sysctl.conf | iproute2, net-tools |
| NAT | Перевод адресов (SNAT/DNAT) | Egress, port forwarding | `iptables -t nat -A POSTROUTING -j MASQUERADE` | /etc/iptables/rules.v4 | iptables, nftables, cloud NAT |
| iptables | Netfilter для фильтрации/NAT | Firewall, DNAT/SNAT | `iptables -L` `iptables-save` | /etc/iptables/* | nftables, ufw, firewalld |
| VLAN | L2 сегментация сети | Сегментация трафика, multi-tenant | `ip link add link eth0 name eth0.10 type vlan id 10` | switch configs | Linux bridges, VXLAN, Cisco |
| Proxy | Forward / reverse proxy | Reverse proxy, кеширование, ACL | `curl --proxy http://proxy:3128` | nginx.conf, haproxy.cfg | NGINX, HAProxy, Envoy, Squid |
| DHCP и DORA | Автоматическое назначение IP (Discover/Offer/Request/Ack) | Автонастройка хостов, PXE | `dhclient` `tcpdump -n port 67 or 68` | dhcpd.conf | isc-dhcp-server, dnsmasq |
| Маска подсети | CIDR / netmask для сетей | Проектирование подсетей, маршруты | `ip addr show` `ipcalc 10.0.0.0/24` | network configs | VPC tools, iproute2 |

## Инфраструктура и автоматизация
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| ansible | Agentless автоматизация и оркестрация | Provisioning, конфигурация, деплой | `ansible-playbook site.yml` | ansible.cfg, hosts, playbooks | Terraform, CI (Jenkins/GitHub Actions) |
| Infrastructure as Code | Описание infra в коде (declarative) | Provisioning облака, repeatable infra | `terraform apply` | *.tf, cloud templates | Terraform, Pulumi, CloudFormation |
| LVM | Логический менеджер томов | Расширение разделов, snapshot | `pvcreate` `lvextend -L +10G` | LVM metadata | mdadm, ZFS, btrfs |
| S3 | Объектное хранилище (AWS S3 и совместимые) | Бэкапы, артефакты, статика | `aws s3 cp file s3://bucket/` | bucket policy JSON | MinIO, awscli, rclone |

## Системные / OS понятия
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| systemd | Init‑система и менеджер юнитов | Управление сервисами, таймеры, cgroups | `systemctl start svc` `journalctl -u svc` | /etc/systemd/system/*.service | systemctl, journalctl |
| systemctl | CLI для systemd | Управление юнитами | `systemctl status nginx` `systemctl enable svc` | N/A | systemd |
| daemon | Фоновый сервис/процесс | Агент мониторинга, демоны приложений | `systemctl start mydaemon` | unit files, init scripts | systemd, supervisord |
| SUID | Set‑UID бит — запуск с правами владельца | Утилиты с повышенными привилегиями | `find / -perm -4000 -type f` `chmod u+s /bin/foo` | N/A (файловые права) | sudo, capabilities |
| Права на изменения в Linux | Файловые права, sudo, ACL, SELinux | Контроль доступа к конфигам/файлам | `chmod` `chown` `setfacl` | /etc/sudoers, filesystem ACLs | SELinux, AppArmor |
| Zombie процесс | Завершённый процесс, ожидающий reaper | Диагностика проблем с wait()/parent | `ps aux | grep Z` | N/A | systemd, supervisor |
| iostat | Мониторинг IO / дисковая статистика | Диагностика дисковой нагрузки | `iostat -x 1` | N/A | sysstat, iotop |
| Bash | Shell и скриптовый язык | Скрипты автоматизации, CI шаги | `bash script.sh` `set -euo pipefail` | ~/.bashrc, /etc/bash.bashrc | sh, zsh, CI runners |
| JVM | Java Virtual Machine | Запуск Java приложений, GC tuning | `java -jar app.jar` `jmap -heap <pid>` | jvm.options, app config | JDK, Maven/Gradle, JFR |
| grep | Текстовый поиск в файлах/потоках | Поиск ошибок в логах, фильтрация вывода | `grep -RIn "ERROR" /var/log` `grep -E 'pat(tern)?' file` | N/A | ripgrep (rg), awk, sed |

## Надёжность, мониторинг и процессы
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| CI | Continuous Integration — сборка/тесты | Автоматизация тестов, PR checks | pipeline triggers | Jenkinsfile, .github/workflows/* | Jenkins, GitHub Actions, GitLab CI |
| CD | Continuous Delivery/Deployment | Автоматический деплой в staging/prod | pipeline deploy | pipeline configs | ArgoCD, Spinnaker, Flux |
| pipeline | Последовательность шагов CI/CD | Build → test → deploy | pipeline CLI / UI | pipeline scripts | Jenkins, Tekton, GitLab CI |
| 4 правила алертов | Правила для качественных оповещений | Actionable, Relevant, Contextual, Limited noise | alertmanager rules | alertmanager.yml, prometheus rules | Prometheus, Alertmanager, PagerDuty |
| RTO, RPO | Recovery Time/Point Objectives | DR-планы, бэкап/репликация | recovery playbooks | backup configs | Backup tools, replication |
| SRE | Практики reliability engineering | SLO/SLI, incident response, postmortems | N/A (процессы) | SLO docs, runbooks | Prometheus, Grafana, Opsgenie |
| Кольца защиты | Defense‑in‑depth: многоуровневая безопасность | Периметр → сеть → хост → приложение → данные | security policies | policy docs | WAF, IDS/IPS, firewall |
| Пирамида тестирования | Соотношение unit/integration/e2e | Планирование тестов CI | `pytest` `k6` | test configs | Testing frameworks, CI |

## Хранилища и базы данных
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| Базы данных | Реляционные и нереляционные СУБД | OLTP/OLAP, репликация, бэкапы | `psql` `mysqldump` `mongodump` | postgresql.conf, my.cnf, mongod.conf | PostgreSQL, MySQL, MongoDB, Redis |
| SQL | Язык запросов для реляционных БД | CRUD, агрегаты, транзакции | `SELECT ...` | schema.sql, migrations | ORMs, pgAdmin |
| NoSQL | Нереляционные БД (KV, document, column) | Масштабируемые/гибкие схемы | `redis-cli` `mongo` | DB config files | Cassandra, MongoDB, Redis |
| Типы данных | Примитивы и составные типы (DB/языки) | Схемы, валидация, сериализация | N/A (описательно) | SQL DDL, JSON Schema, migrations | ORMs, Protobuf, Avro |

## Безопасность и криптография
| Название | Описание | Примеры применения | Команды (примеры) | Ключевые конфигурационные файлы | Связанные инструменты |
|---|---|---|---|---|---|
| SSH | Протокол удалённого доступа и туннелей | Администрирование, SCP/SFTP, port‑forward | `ssh user@host` `scp file user@host:/tmp` | /etc/ssh/sshd_config, ~/.ssh/authorized_keys | OpenSSH, ssh-agent, Mosh |
| SHA | Криптографические хеши (SHA-1/256/512) | Проверка целостности, подписи артефактов | `sha256sum file` | N/A | openssl, gpg |
