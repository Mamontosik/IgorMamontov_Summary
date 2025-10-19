# Полезные команды для установки и настройки Kubernetes

## Работа с файлом Dockerfile при формировании контейнера

!!! note "Лучшие практики инструкций dockerfile"

    **Используйте `COPY` вместо `ADD`, если не нужно распаковывать архивы, явное распаковывание в RUN**
    <br>
    ``` yaml
    COPY app.tar.gz /tmp/
    RUN tar -xzf /tmp/app.tar.gz -C /app && rm /tmp/app.tar.gz
    ```
    **Минимизируйте число `COPY` и `ADD`, чтобы уменьшить размер образа, копируем только нужные файлы**
    <br>
    ``` yaml
    COPY src/ /app/src/
    COPY requirements.txt /app/
    ```
    ``` yaml
    Добавьте .dockerignore
    .git
    node_modules
    __pycache__
    *.log
    ```
    **Копируйте только изменяемые файлы, чтобы ускорить кэширование, сначала зависимости, потом код**
    <br>
    ``` yaml
    COPY requirements.txt /app/
    RUN pip install -r /app/requirements.txt

    COPY src/ /app/src/
    ```
    **Не используйте `ADD` для загрузки файлов из интернета, используем RUN curl + COPY**
    <br>
    ```yaml
    RUN curl -L -o /tmp/file.tar.gz https://example.com/file.tar.gz
    COPY file.tar.gz /app/
    ```

## Использование и настройка дашборда Lens

!!! note "Установка и настройка Lens"

    **Скачивание и установка дистрибутива**
    <br>
    Необходимо перейти на сайт https://k8slens.dev/download и скачать необходимый дистрибутив под свою операционную систему

    **Документация по работе с Lens**
    <br>
    Ознакомитья с шагами по настройке и управлению Lens можно по ссылке https://docs.k8slens.dev/getting-started/activate-lens-desktop/

    **Подключение доступного кластера и созданных namespace для отображения в Lens через загрузку kuberconfig**
    <br>
    1. Переходим на страницу "Add Clusters from kuberconfig"
    <br>
    2.  Находим настройки kuberconfig в каталоге через команду
    ```bash
    cat .kube/config
    ```
    3. Сохраняем получившийся результат на локальной машине по адресу /kuber/config в разделе приложения Lens

## Управление контекстами в рамках оркестрации

!!! note "Переключение между openshift и kubernetes"

    **Что такое контекст?**
    <br>
    Это набор параметров, которые определяют, с каким кластером Kubernetes и пространством имен вы работаете.

    **Команды для работы с context**
    <br>
    ```bash
    kubectl config get-contexts - доступные контаксты и указание активного
    ```
    <br>
    ``` bash
    kubectl config use-context <context-name> - переключение на нужный контекст
    ```
