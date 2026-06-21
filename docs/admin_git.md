# Полезные команды для работы с репозиторием git

## Команды git

!!! note Самые часто используемые команды git

    **Клонирование удаленного репозитория на локальный компьютер**
    <br>
    Команда git clone
    ```
    git clone https://github.com/user/repo.git [Стандартный клон]
    git clone --branch <branch_name> https://github.com/user/repo.git [Клон конкретной ветки]
    git clone --depth 1 https://github.com/user/repo.git [Клон только с историей коммитов (shallow clone)]
    ```

    **Отображение текущего состояния рабочего каталога и индекса**
    <br>
    Команда git status
    ``` 
    git status [Базовый статус]
    git status --short [Краткий статус]
    git status --untracked-files=no [Игнорировать изменения в файлах]
    ```

    **Добавление изменений в индекс (staging area)**
    <br>
    Команда git add
    ```
    git add . [Добавляет все изменения]
    git add src/file.js [Добавить конкретный файл]
    git add -u [Добавить изменения, но не добавлять новые файлы]
    ```

    **Создание снимка текущего состояния файлов и индексов**
    <br>
    Команда git commit
    ```
    git commit -m "Added new feature" [Коммит с сообщением]
    git commit -a -m "Fixed bug" [Коммит всех изменений (включая untracked files)]
    git commit --amend [Ампендинг коммита (исправить последний коммит)]
    ```

    **Отправка локальных коммитов на удалённый репозиторий**
    <br>
    Команда git push
    ```
    git push origin main [Обычная отправка изменений]
    git push --force origin main [Форсированная отправка (перезаписать удалённые изменения)]
    git push --tags [Только tags]  
    ```

    **Загрузка изменения из удалённого репозитория и объединяет их с локальной веткой**
    <br>
    Команда git pull
    ```
    git pull origin main [Простое объединение изменений]
    git pull --rebase origin main [Merge strategy]
    git fetch origin && git merge origin/main [Fetch and merge отдельно]
    ```

    **Управляние ветками (создание, переключение, удаление)**
    <br>
    Команда git branch
    ```
    git branch -a [Показывает все ветки]
    git branch feature-branch [Создать новую ветку]
    git checkout feature-branch [Переключиться на ветку]
    git branch -d feature-branch [Удалить ветку]
    ```

    **Переключение активной ветки или восстановление файлов**
    <br>
    Команда git checkout
    ```
    git checkout feature-branch
    ```

    **Объединение изменений из одной ветки в другую**
    <br>
    Команда git merge
    ```
    git merge feature-branch [Слияние ветки]
    git merge --ff-only feature-branch [Слияние с изменением истории (fast-forward)]
    git merge --no-ff feature-branch [Слияние с разрешением конфликтов]
    ```

    **Отображение истории коммитов**
    <br>
    Команда git log
    ```
    :git log [История коммитов]
    :git log --oneline [Краткий вывод]
    :git log --author="John Doe" [Фильтрация по автору]
    ```

    **Временное сохранение изменений, не готовых для коммита**
    <br>
    Команда git stash
    ```
    git stash
    git stash pop
    ```

    **Отмена изменений, внесённых конкретным коммитом, через создание нового коммита**
    <br>
    Команда git revert
    ```
    git revert <commit_hash>
    ```

    **Применение коммита из другой ветки в текущую**
    <br>
    Команда git cherry-pick
    ```
    git cherry-pick <commit_hash>
    ```

    **Сбрасывает HEAD к указанному коммиту**
    <br>
    Команда git reset
    ```
    git reset --hard HEAD~1
    ```

    **Переносит коммиты на вершину другой ветки**
    <br>
    Команда git rebase
    ```
    git rebase master
    ```

    **Сравнение изменений между различными версиями файлов в Git**
    <br>
    Команда git diff
    ```
    git diff HEAD [Показать различия с последним коммитом]
    git diff main feature-branch [Показать различия между ветками]
    git diff -- src/file.js [Показать только изменения в файле]
    git diff --cached [Показывает разницу между индексированными изменениями и последним коммитом] 
    git diff --staged [Показывает разницу между стадированным индексом и последним коммитом]
    git diff --color [Выводит изменения с подсветкой цветов]
    git diff --word-diff [Показывает разницу на уровне слов, а не строк]
    git diff --diff-filter=D [Показывает только удалённые файлы]
    git diff --stat [Показывает компактный отчёт о размере изменений]
    git diff --patch [Показывает полные патчи с изменениями]
    git diff --shortstat [Показывает сжатую статистику изменений]
    git diff --name-only [Показывает только имена изменённых файлов]
    git diff --name-status [Показывает имена файлов и их статус (modified, added, deleted)]
    git diff --numstat [Показывает изменения в виде чисел (добавленные/удалённые строки)]
    git diff --relative [Показывает относительные пути файлов]
    git diff --full-index [Показывает полный хеш коммита]
    git diff --binary [Включает двоичные файлы в вывод]
    git diff --exit-code [Завершает команду с ненулевым кодом, если есть различия]
    git diff --quiet [Ничего не выводит, если нет различий]
    git diff --no-index file1.txt file2.txt [Позволяет сравнить файлы, не находящиеся под контролем Git]
    ```

!!! note Паттерны ветления

    Перечисление паттернов git
    
    1. Git Flow — основная ветка main/master, разработка в develop, релизы и фичи — отдельные ветки.
    2.  GitHub Flow — коммиты прямо в main через pull-request'ы.
    3.  GitLab Flow — сочетание Git Flow и CI/CD: возможность работать как с protected branches, так и напрямую.
     4.  Trunk-Based Development — вся работа в одной главной ветке (main), feature flags для управления выпуском.

!!! note Игнорирование файлов в Git

    Файлы можно игнорировать на трёх разных уровнях, а не только через `.gitignore`.

    **1. `.gitignore`**

    Коммитится в репозиторий вместе с кодом. Действует для всех участников проекта.
    ```
    .DS_Store
    node_modules/
    *.log
    ```

    **2. `.git/info/exclude`**

    Лежит в директории `.git` каждого репозитория, но **не коммитится**. Удобно для локальных файлов, которые не должны попасть в общий `.gitignore` (личные заметки, локальные конфиги).
    ```
    notes.txt
    local-config.env
    ```

    **3. `~/.config/git/ignore` (глобальный)**

    Глобальный файл для всей системы. Не привязан к репозиторию. Идеально для файлов вроде `.DS_Store` на macOS.
    ```
    .DS_Store
    Thumbs.db
    ```

    **Кастомный путь к глобальному файлу**

    Можно переопределить путь через `core.excludesFile`:
    ```
    git config --global core.excludesFile ~/.gitignore_global
    ```
    Вернуть стандартный путь:
    ```
    git config --global --unset core.excludesFile
    ```

    **Диагностика: какой файл игнорирует конкретный файл**

    Команда `git check-ignore -v <file>` показывает, какой именно файл и какая строка отвечает за игнорирование:
    ```
    git check-ignore -v .DS_Store
    ```

    Примеры вывода `git check-ignore -v .DS_Store`:

    | Уровень | Файл | Вывод команды |
    |---------|------|---------------|
    | Локальный (коммитится) | `.gitignore` | `.gitignore:1:.DS_Store` |
    | Локальный (не коммитится) | `.git/info/exclude` | `.git/info/exclude:7:.DS_Store` |
    | Глобальный (система) | `~/.config/git/ignore` | `/Users/user/.config/git/ignore:2:.DS_Store` |
    | Кастомный глобальный | `~/.gitignore_global` | `/Users/user/.gitignore_global:1:.DS_Store` |

    Если файл ничем не игнорируется — команда не выдаёт ничего.
