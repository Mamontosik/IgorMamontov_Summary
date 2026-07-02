# Полезные команды для работы с репозиторием git

## Основные команды

!!! tip "Клонирование репозитория"
    | Команда | Описание |
    |---------|----------|
    | `git clone https://github.com/user/repo.git` | Стандартный клон |
    | `git clone --branch <branch_name> https://github.com/user/repo.git` | Клон конкретной ветки |
    | `git clone --depth 1 https://github.com/user/repo.git` | Shallow clone (без истории коммитов) |

!!! tip "Работа с изменениями (status, add, commit)"
    | Команда | Описание |
    |---------|----------|
    | `git status` | Базовый статус |
    | `git status --short` | Краткий статус |
    | `git status --untracked-files=no` | Игнорировать untracked-файлы |
    | `git add .` | Добавить все изменения |
    | `git add src/file.js` | Добавить конкретный файл |
    | `git add -u` | Добавить изменения без новых файлов |
    | `git commit -m "message"` | Коммит с сообщением |
    | `git commit -a -m "message"` | Коммит всех изменений (включая untracked) |
    | `git commit --amend` | Исправить последний коммит |

!!! tip "Синхронизация с удалённым репозиторием (push, pull, fetch)"
    | Команда | Описание |
    |---------|----------|
    | `git push origin main` | Отправить изменения |
    | `git push --force origin main` | Форсированная отправка |
    | `git push --tags` | Отправить только теги |
    | `git pull origin main` | Забрать и объединить |
    | `git pull --rebase origin main` | Забрать с rebase вместо merge |
    | `git fetch origin && git merge origin/main` | Fetch и merge отдельно |

!!! tip "Управление ветками (branch, checkout, merge, rebase)"
    | Команда | Описание |
    |---------|----------|
    | `git branch -a` | Список всех веток |
    | `git branch feature-branch` | Создать новую ветку |
    | `git checkout feature-branch` | Переключиться на ветку |
    | `git branch -d feature-branch` | Удалить ветку |
    | `git merge feature-branch` | Слияние веток |
    | `git merge --ff-only feature-branch` | Fast-forward слияние |
    | `git merge --no-ff feature-branch` | Слияние с коммитом |
    | `git rebase master` | Перенести коммиты на вершину другой ветки |

!!! warning "Отмена и восстановление изменений"
    | Команда | Описание |
    |---------|----------|
    | `git stash` | Временно сохранить незакоммиченные изменения |
    | `git stash pop` | Восстановить из stash |
    | `git revert <commit_hash>` | Отменить коммит через новый коммит |
    | `git reset --hard HEAD~1` | Сбросить HEAD на 1 назад |
    | `git cherry-pick <commit_hash>` | Применить коммит из другой ветки |

!!! info "История и сравнение (log, diff)"

    **Просмотр истории:**

    | Команда | Описание |
    |---------|----------|
    | `git log` | История коммитов |
    | `git log --oneline` | Краткий вывод (один коммит — одна строка) |
    | `git log --author="John Doe"` | Фильтр по автору |

    **Сравнение версий (git diff):**

    | Назначение | Команда | Описание |
    |------------|---------|----------|
    | Базовое | `git diff HEAD` | Различия с последним коммитом |
    | Между ветками | `git diff main feature-branch` | Различия между ветками |
    | По файлу | `git diff -- src/file.js` | Только изменения в файле |
    | Индекс | `git diff --cached` | Индекс vs последний коммит |
    | Индекс | `git diff --staged` | Стадированные vs последний коммит |
    | Формат | `git diff --color` | Цветная подсветка |
    | Формат | `git diff --word-diff` | Различия на уровне слов |
    | Формат | `git diff --patch` | Полные патчи |
    | Фильтрация | `git diff --diff-filter=D` | Только удалённые файлы |
    | Фильтрация | `git diff --name-only` | Только имена изменённых файлов |
    | Фильтрация | `git diff --name-status` | Имена + статус (M/A/D) |
    | Статистика | `git diff --stat` | Компактный отчёт о размере изменений |
    | Статистика | `git diff --shortstat` | Сжатая статистика |
    | Статистика | `git diff --numstat` | Числа (добавлено/удалено строк) |
    | Дополнительно | `git diff --relative` | Относительные пути |
    | Дополнительно | `git diff --full-index` | Полный хеш коммита |
    | Дополнительно | `git diff --binary` | Включая бинарные файлы |
    | Дополнительно | `git diff --exit-code` | Ненулевой код при различиях |
    | Дополнительно | `git diff --quiet` | Ничего не выводить, если нет различий |
    | Дополнительно | `git diff --no-index file1 file2` | Сравнить 2 файла вне Git |

## Паттерны ветвления

!!! info "Стратегии ветвления"
    | Стратегия | Суть | Когда использовать |
    |-----------|------|-------------------|
    | **Git Flow** | main + develop + feature/release/hotfix | Классические релизы |
    | **GitHub Flow** | Коммиты в main через pull request | Continuous delivery |
    | **GitLab Flow** | Git Flow + CI/CD + protected branches | Enterprise с CI/CD |
    | **Trunk-Based Development** | Одна main + feature flags | High-velocity DevOps |

## Игнорирование файлов

!!! tip "Три уровня .gitignore"
    | Уровень | Файл | Особенность |
    |---------|------|-------------|
    | Локальный (коммитится) | `.gitignore` | Для всех участников проекта |
    | Локальный (не коммитится) | `.git/info/exclude` | Личные файлы, заметки |
    | Глобальный (система) | `~/.config/git/ignore` | Для всей ОС (`.DS_Store`, `Thumbs.db`) |

!!! tip "Глобальный .gitignore"
    | Команда | Описание |
    |---------|----------|
    | `git config --global core.excludesFile ~/.gitignore_global` | Указать свой путь |
    | `git config --global --unset core.excludesFile` | Вернуть стандартный путь |

!!! tip "Диагностика: git check-ignore"
    `git check-ignore -v <file>` — показывает, какой файл и какая строка игнорирует файл:
    ```
    git check-ignore -v .DS_Store
    ```

    | Уровень | Файл | Вывод команды |
    |---------|------|---------------|
    | Локальный (коммитится) | `.gitignore` | `.gitignore:1:.DS_Store` |
    | Локальный (не коммитится) | `.git/info/exclude` | `.git/info/exclude:7:.DS_Store` |
    | Глобальный (система) | `~/.config/git/ignore` | `/Users/user/.config/git/ignore:2:.DS_Store` |
    | Кастомный глобальный | `~/.gitignore_global` | `/Users/user/.gitignore_global:1:.DS_Store` |

    Если файл ничем не игнорируется — команда не выдаёт ничего.
