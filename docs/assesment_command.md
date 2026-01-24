# Список популярных команд которые спрашивают на собеседованиях

!!! note ""

    Список популярных команд по Git, Docker, Kubernetes, Ansible, Terraform, Linux, CI/CD Pipeline, Gitlab, Jenkins


| Категория | Команда | Описание |
| --- | --- | --- |
| **Git** | `git init` | Инициализация нового репозитория. |
|  | `git clone <url>` | Клонирование репозитория. |
|  | `git add <file>` | Добавление файла к коммиту. |
|  | `git commit -m "сообщение"` | Создание коммита. |
|  | `git push` | Отправка изменений. |
|  | `git pull` | Получение изменений. |
|  | `git branch` | Список веток. |
|  | `git checkout <branch>` | Переключение ветки. |
| --- | --- | --- |
| **Docker** | `docker build -t <image_name> .` | Создание образа. |
|  | `docker run -d -p 80:80 <image_name>` | Запуск контейнера. |
|  | `docker ps` | Список контейнеров. |
|  | `docker stop <container_id>` | Остановка контейнера. |
|  | `docker rm <container_id>` | Удаление контейнера. |
|  | `docker images` | Список образов. |
|  | `docker rmi <image_id>` | Удаление образа. |
| --- | --- | --- |
| **Kubernetes (kubectl)** | `kubectl get pods` | Список подов. |
|  | `kubectl get services` | Список сервисов. |
|  | `kubectl describe pod <pod_name>` | Информация о поде. |
|  | `kubectl logs <pod_name>` | Логи пода. |
|  | `kubectl apply -f <file.yaml>` | Применение конфигурации. |
|  | `kubectl delete pod <pod_name>` | Удаление пода. |
|  | `kubectl exec -it <pod_name> -- /bin/bash` | Подключение к поду. |
| --- | --- | --- |
| **Ansible** | `ansible-playbook <playbook.yml>` | Запуск плейбука. |
|  | `ansible <host> -m ping` | Проверка хостов. |
|  | `ansible <host> -m command -a 'uptime'` | Выполнение команды. |
|  | `ansible-galaxy install <role>` | Установка роли. |
| --- | --- | --- |
| **Terraform** | `terraform init` | Инициализация. |
|  | `terraform plan` | Планирование изменений. |
|  | `terraform apply` | Применение изменений. |
|  | `terraform destroy` | Удаление ресурсов. |
| --- | --- | --- |
| **Linux (bash)** | `ls` | Список файлов. |
|  | `cd <directory>` | Перемещение в каталог. |
|  | `pwd` | Текущий каталог. |
|  | `cp <source> <destination>` | Копирование файлов. |
|  | `mv <source> <destination>` | Перемещение файлов. |
|  | `rm <file>` | Удаление файла. |
|  | `mkdir <directory>` | Создание каталога. |
|  | `grep <pattern> <file>` | Поиск шаблона. |
|  | `find <directory> -name <pattern>` | Поиск файлов. |
|  | `chmod <permissions> <file>` | Изменение прав. |
|  | `chown <user>:<group> <file>` | Изменение владельца. |
|  | `top` | Мониторинг процессов. |
|  | `ps aux` | Список процессов. |
| --- | --- | --- |
| **GitLab CI/CD** | `.gitlab-ci.yml` | Конфигурация пайплайна. |
|  | `gitlab-runner register` | Регистрация Runner. |
|  | `gitlab-runner list` | Список Runner. |
| --- | --- | --- |
| **Jenkins** | `jenkins-cli.jar` | Утилита CLI. |
|  | `jenkins-jobs create <job_name>` | Создание задачи. |
|  | `jenkins-jobs build <job_name>` | Запуск задачи. |
|  | `jenkins-jobs list` | Список задач. |
| --- | --- | --- |
| **GitHub Actions** | `workflow_dispatch` | Ручной запуск. |
|  | `.github/workflows/<workflow>.yml` | Конфигурация workflow. |
