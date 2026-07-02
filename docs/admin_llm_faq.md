# LLM FAQ

## Модели LLM

!!! info "Сводка инструментов"
    | Название | Описание | Примеры применения | Связанные инструменты |
    | --- | --- | --- | --- |
    | **Ollama** | CLI/сервер для локальных LLM (Llama, Mistral, Gemma и др.) | Локальный запуск, R&D прототипирование | llama.cpp, LM Studio |
    | **LM Studio** | GUI для локальных LLM (GGUF + llama.cpp) | Локальный запуск на CPU/GPU, разработка | Ollama, RAG |
    | **TensorZero** | LLMOps платформа (A/B тесты, роутинг, Autopilot) | Продакшн LLM с A/B тестированием | vLLM, Ollama, OpenAI |

!!! tip "Ollama"
    **Что это?** CLI-инструмент и сервер для запуска локальных LLM. Использует llama.cpp для эффективного инференса.

    **Для чего используется?** Локальный запуск LLM, R&D прототипирование, тестирование моделей без облачных сервисов.

    **Ключевые особенности:**
    - Простой CLI-интерфейс
    - Встроенный HTTP-сервер
    - Поддержка GPU (NVIDIA, AMD, Apple Silicon)
    - Модель Modelfile для настройки
    - Множество готовых моделей

    **Установка и запуск:**
    ```bash
    curl -fsSL https://ollama.com/install.sh | sh
    ollama run llama3
    ollama serve
    ```

    **Полезные команды:**
    ```bash
    ollama list           # Список установленных моделей
    ollama pull <model>   # Скачать модель
    ollama rm <model>     # Удалить модель
    ```

    **Параметры Modelfile:**

    | Параметр | Описание |
    |----------|----------|
    | `temperature` | Креативность модели (0–2) |
    | `num_ctx` | Размер контекста |
    | `num_gpu` | Количество GPU для инференса |
    | `top_p` | Nucleus sampling |
    | `top_k` | Top-k sampling |
    | `repeat_penalty` | Штраф за повторы |

    **Пример Modelfile:**
    ```bash
    FROM llama3
    PARAMETER temperature 0.7
    PARAMETER num_ctx 4096
    SYSTEM You are a helpful AI assistant.
    ```

!!! tip "LM Studio"
    **Что это?** GUI для запуска локальных LLM в формате GGUF через llama.cpp.

    **Для чего используется?** Локальный запуск LLM на CPU/GPU, прототипирование RAG-систем.

    **Ключевые особенности:**
    - Удобный GUI-интерфейс
    - Поддержка GGUF-формата
    - Выбор GPU-бэкенда (Metal, CUDA, ROCm)
    - Встроенный чат-интерфейс
    - Локальный OpenAI-совместимый API

    **API-доступ:**
    ```bash
    localhost:1234/v1/chat/completions
    localhost:1234/v1/completions
    ```

    **Параметры:**

    | Параметр | Описание |
    |----------|----------|
    | `GPU-backend` | Тип GPU (Metal/CUDA/ROCm) |
    | `offloading` | Выгрузка слоёв на GPU |
    | `context size` | Размер контекста |
    | `GPU layers` | Количество слоёв на GPU |

!!! tip "TensorZero"
    **Что это?** Open-source LLMOps-платформа для production LLM: A/B-тестирование, роутинг, Autopilot.

    **Для чего используется?** Production LLM с A/B-тестированием, роутинг запросов, автооптимизация промптов.

    **Ключевые особенности:**
    - A/B-тестирование моделей
    - Умный роутинг между LLM
    - Autopilot (автооптимизация промптов)
    - Сбор и анализ метрик
    - Поддержка множества бэкендов
    - OpenAI-совместимый API

    **Поддерживаемые бэкенды:** Ollama, vLLM, OpenAI, Anthropic и др.

    **Параметры gateway.yaml:**

    | Параметр | Описание |
    |----------|----------|
    | `Router` | Правила роутинга |
    | `Autopilot` | Политики автооптимизации |
    | `datasets` | Наборы данных для тестирования |
    | `alerts` | Оповещения |

    **Пример gateway.yaml:**
    ```yaml
    gateway:
      type: openai
      url: http://localhost:8000/v1
      models:
        - name: gpt-4
        - name: claude-3
    routing:
      policies:
        - name: latency-optimized
          type: minimum_latency
        - name: quality-optimized
          type: highest_score
    ```

## RAG

!!! info "Концепция RAG"
    Retrieval-Augmented Generation — подход, при котором LLM дополняет знания внешней информацией из базы знаний в реальном времени. Вместо переобучения — «подключение» данных в момент запроса.

    **Когда использовать RAG:**
    - Базы знаний, документация, FAQ
    - Корпоративные данные
    - Информация, которая часто обновляется
    - Нужно ссылаться на конкретные источники

!!! info "RAG vs Fine-tuning vs Промпты"
    | Подход | Когда использовать |
    | --- | --- |
    | **Промпты** | Простые задачи, модель уже знает нужную информацию |
    | **RAG** | Нужны актуальные данные или специфическая база знаний |
    | **Fine-tuning** | Нужно изменить поведение/стиль модели, особые форматы вывода |

!!! tip "Архитектура RAG"
    ```text
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │ Документы   │ →  │  Chunking   │ →  │ Embedding   │ →  │ Vector DB   │
    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                       ↓
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │   Ответ     │ ←  │  Генерация  │ ←  │  Поиск      │ ←  │  Запрос     │
    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
    ```

### Chunking

!!! tip "Разбиение документов"
    **Что это?** Разбиение документов на фрагменты (chunks) для индексации. Каждый чанк — законченная смысловая единица.

    **Стратегии:**
    | Стратегия | Описание |
    |-----------|----------|
    | По тексту | Абзацы, параграфы |
    | По токенам | Фиксированное количество токенов (256–1024) |
    | Рекурсивное | Иерархическое разбиение (заголовки → предложения) |

    **Пример (LangChain):**
    ```python
    from langchain.text_splitter import RecursiveCharacterTextSplitter

    splitter = RecursiveCharacterTextSplitter(
        chunk_size=512,
        chunk_overlap=50,
        separators=["\n\n", "\n", ". ", " "]
    )
    chunks = splitter.split_text(long_document)
    ```

!!! tip "Векторные БД (Vector DB)"
    | Название | Описание | Когда использовать |
    | --- | --- | --- |
    | **Chroma** | Встроенная Vector DB для Python/LangChain | Прототипирование, малые объёмы |
    | **FAISS** | Библиотека от Meta для поиска ближайших соседей | Локальный поиск, большие объёмы |
    | **Qdrant** | Production Vector DB с API | Production, высокая нагрузка |
    | **Pinecone** | Managed Vector DB | Облако, масштабирование |

    **Пример Chroma:**
    ```python
    import chromadb
    from chromadb.utils.embedding_functions import SentenceTransformerEmbeddingFunction

    client = chromadb.Client()
    collection = client.create_collection(
        name="documents",
        embedding_function=SentenceTransformerEmbeddingFunction()
    )
    collection.add(documents=["текст документа"], ids=["doc1"])
    ```

    **Пример Qdrant:**
    ```python
    from qdrant_client import QdrantClient
    from qdrant_client.models import Distance, VectorParams

    client = QdrantClient(host="localhost", port=6333)
    client.recreate_collection(
        collection_name="documents",
        vectors_config=VectorParams(size=384, distance=Distance.COSINE)
    )
    ```

!!! info "Embedding модели"
    | Модель | Размерность | Особенности |
    | --- | --- | --- |
    | **sentence-transformers/all-MiniLM-L6-v2** | 384 | Быстрая, хорошее качество |
    | **BAAI/bge-small-en-v1.5** | 1024 | Высокое качество, RAG |
    | **intfloat/e5-small-v2** | 384 | Multilingual |
    | **text-embedding-3-small** | 1536 | OpenAI API |

!!! tip "Поиск"
    **Semantic Search** — поиск по смыслу:
    ```python
    results = collection.query(query_texts=["запрос пользователя"], n_results=5)
    ```

    **Hybrid Search** — семантический + текстовый поиск:
    ```python
    from rankify import HybridRetriever

    retriever = HybridRetriever(
        semantic_model=embedding_model,
        keyword_index=keyword_index,
        weights=[0.7, 0.3]
    )
    results = retriever.retrieve("запрос")
    ```

    **Re-ranking** — переранжировка для точности:
    ```python
    from cohere import Client

    reranker = Client(api_key="cohere-key")
    results = reranker.rerank(query="запрос", documents=initial_results, top_n=3)
    ```

!!! info "Инъекция контекста"
    **LangChain:**
    ```python
    from langchain.chains import RetrievalQA
    from langchain_community.llms import OpenAI

    qa = RetrievalQA.from_chain_type(
        llm=OpenAI(),
        chain_type="stuff",
        retriever=vectorstore.as_retriever()
    )
    result = qa.invoke("вопрос")
    ```

    **LlamaIndex:**
    ```python
    from llama_index import VectorStoreIndex, StorageContext

    index = VectorStoreIndex.from_documents(documents)
    query_engine = index.as_query_engine()
    response = query_engine.query("вопрос")
    ```

    **Prompt с контекстом:**
    ```python
    prompt = f"""
    Используй следующий контекст для ответа:
    {context}

    Вопрос: {question}
    Ответ:
    """
    ```

!!! tip "Фреймворки"
    **LangChain** — универсальный фреймворк для работы с LLM:
    ```python
    from langchain.schema import Document
    from langchain_community.vectorstores import Chroma

    docs = [Document(page_content=chunk) for chunk in chunks]
    vectorstore = Chroma.from_documents(docs, embedding)
    ```

    **LlamaIndex** — специализированный фреймворк для RAG:
    ```python
    from llama_index import SimpleDirectoryReader
    from llama_index import VectorStoreIndex

    reader = SimpleDirectoryReader(input_files=["doc.txt"])
    documents = reader.load_data()
    index = VectorStoreIndex.from_documents(documents)
    ```

!!! note "RAG vs Memory"
    | Характеристика | RAG | Memory |
    | --- | --- | --- |
    | **Данные** | База знаний | История чата |
    | **Назначение** | Факты, документы | Контекст диалога |
    | **Обновление** | Периодическое | После каждого сообщения |

!!! info "Метрики RAG"
    **RAGAS (Retrieval-Augmented Generation Assessment):**
    | Метрика | Описание |
    | --- | --- |
    | **Faithfulness** | Насколько ответ соответствует контексту |
    | **Answer Relevance** | Насколько ответ релевантен запросу |
    | **Context Precision** | Точность извлечённого контекста |
    | **Context Recall** | Полнота извлечённого контекста |

    **Базовые метрики:**
    ```python
    def calculate_precision(retrieved, relevant):
        return len(retrieved & relevant) / len(retrieved)

    def calculate_recall(retrieved, relevant):
        return len(retrieved & relevant) / len(relevant)
    ```

## Обслуживание LLM

!!! info "Сводка инструментов"
    | Название | Описание | Примеры применения | Связанные инструменты |
    | --- | --- | --- | --- |
    | **vLLM** | Высокопроизводительный движок инференса с PagedAttention и Continuous Batching | Production API для LLM (чат-боты, RAG, агенты) | LMCache, TensorZero, GPUStack |
    | **LMCache** | Кэширование KV-кеша LLM на DRAM/диск | Повторные запросы, RAG, чат-сессии | vLLM, RAG |
    | **GPUStack** | Оркестратор GPU (open-source) | Управление несколькими GPU, Kubernetes | vLLM, Kubernetes |

!!! tip "vLLM"
    **Что это?** Высокопроизводительный движок инференса LLM с PagedAttention для управления памятью и Continuous Batching для параллельных запросов.

    **Для чего используется?** Production API: чат-боты, RAG, агенты, поисковые системы.

    **Ключевые особенности:**
    - PagedAttention — эффективное управление KV-кешем
    - Continuous Batching — динамическая обработка запросов
    - OpenAI-совместимый API (`/v1/chat/completions`, `/v1/completions`)
    - Tensor Parallelism — распределение модели на несколько GPU
    - Поддержка множества моделей: Llama, Mistral, Qwen и др.

    **Запуск:**
    ```bash
    docker run -it --gpus all -p 8000:8000 vllm/vllm-openai --model meta-llama/Llama-3-8b-instruct
    ```

    **Параметры:**

    | Параметр | Описание |
    |----------|----------|
    | `tensor-parallelism` | Количество GPU для одной модели |
    | `max_num_seqs` | Максимум параллельных запросов |
    | `max_model_len` | Максимальная длина контекста |
    | `max_num_batched_tokens` | Максимум токенов в батче |

    **Интеграция с OpenAI SDK:**
    ```python
    from openai import OpenAI
    client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")
    response = client.chat.completions.create(model="Llama-3-8b", messages=[...])
    ```

!!! tip "LMCache"
    **Что это?** Система кэширования KV-кеша — сохранение и повторное использование вычисленных ключей и значений внимания.

    **Для чего используется?** Ускорение повторных запросов, RAG с длинными контекстами, многоходовые чат-сессии.

    **Ключевые особенности:**
    - Кэширование на DRAM или диске
    - Поддержка бэкендов (Redis, диск)
    - Политика вытеснения (LRU, LFU)
    - Интеграция с vLLM, Ollama
    - Совместное использование кэша между запросами

    **Параметры:**

    | Параметр | Описание |
    |----------|----------|
    | `cache_size` | Размер кэша |
    | `eviction_policy` | Политика вытеснения (LRU, LFU) |
    | `ttl` | Время жизни записей |
    | `backend` | Тип хранилища (redis, disk) |

!!! tip "GPUStack"
    **Что это?** Open-source оркестратор GPU с интеграцией Kubernetes.

    **Для чего используется?** Управление GPU-ресурсами в кластере, распределение нагрузки между узлами.

    **Ключевые особенности:**
    - Поддержка NVIDIA и AMD GPU
    - Интеграция с Kubernetes
    - Автоматическое распределение нагрузки
    - Мониторинг GPU-ресурсов

    **Параметры:**

    | Параметр | Описание |
    |----------|----------|
    | `GPU node config` | Конфигурация GPU-узлов |
    | `replicas` | Количество реплик |
    | `resources.nvidia.com/gpu` | Запрос GPU-ресурсов |

## Fine-tuning

!!! info "Когда применять"
    | Подход | Когда использовать |
    | --- | --- |
    | **Промпты** | Базовая функциональность, модель уже знает |
    | **RAG** | Нужны внешние данные, актуальная информация |
    | **Fine-tuning** | Особый стиль, формат, поведение модели |

!!! warning "Когда точно нужен Fine-tuning"
    - Специфический формат вывода (JSON, XML)
    - Особый стиль общения
    - Narrow domain expertise
    - Сложные задачи

!!! info "Типы Fine-tuning"
    | Тип | Описание |
    | --- | --- |
    | **SFT** (Supervised Fine-Training) | Классическое дообучение на парах (input, output) |
    | **DPO** (Direct Preference Optimization) | Оптимизация на основе предпочтений |
    | **RLHF** (Reinforcement Learning from Human Feedback) | Обучение с подкреплением от человека |

!!! tip "Инструменты"
    | Инструмент | Описание |
    | --- | --- |
    | **Axolotl** | Универсальный инструмент для fine-tuning |
    | **Unsloth** | Быстрый fine-tuning (до 2x быстрее, меньше памяти) |
    | **OpenAI API** | Fine-tuning через OpenAI API |
    | **DeepSpeed** | Distributed training от Microsoft |

!!! tip "Пример: Axolotl"
    **Конфигурация:**
    ```yaml
    model: meta-llama/Llama-3-8b
    base_model: meta-llama/Llama-3-8b
    load_in_4bit: true
    strict: false
    datasets:
      - path: dataset.jsonl
        type: chat_template
    sequence_len: 2048
    max_steps: 500
    learning_rate: 2e-4
    ```

    **Запуск:**
    ```bash
    accelerate config default
    accelerate launch axolotl/train.py config.yaml
    ```

!!! tip "Пример: Unsloth"
    ```python
    from unsloth import FastLanguageModel
    import torch

    model, tokenizer = FastLanguageModel.from_pretrained(
        model_name="meta-llama/Llama-3.1-8B-Instruct",
        max_seq_length=2048,
        dtype=torch.float16,
        load_in_4bit=True
    )
    model = FastLanguageModel.get_peft_model(
        model,
        r=16,
        target_modules=["q_proj", "k_proj", "v_proj", "o_proj"]
    )
    ```

## Управление контекстом

!!! info "Context Window популярных моделей"
    | Модель | Context Window |
    | --- | --- |
    | GPT-4 | 128k токенов |
    | Claude 3.5 | 200k токенов |
    | Gemini 1.5 | 2M токенов |
    | Llama 3.1 | 128k токенов |
    | Mistral Large | 32k токенов |

!!! tip "Стратегии управления контекстом"
    **Truncation** — обрезать контекст до лимита:
    ```python
    def truncate_context(messages, max_tokens):
        total_tokens = sum(len(tokenizer.encode(m["content"])) for m in messages)
        while total_tokens > max_tokens and len(messages) > 1:
            messages.pop(1)
            total_tokens = sum(len(tokenizer.encode(m["content"])) for m in messages)
        return messages
    ```

    **Rolling Window** — оставить последние N сообщений:
    ```python
    def rolling_window(messages, keep_last=10):
        system = messages[0] if messages[0]["role"] == "system" else None
        conversation = [m for m in messages if m["role"] != "system"]
        result = conversation[-keep_last:]
        if system:
            result = [system] + result
        return result
    ```

    **Summarize** — сжатие длинного контекста:
    ```python
    def summarize_old_messages(messages):
        if len(messages) <= 2:
            return messages
        summary_prompt = "Создай краткое резюме следующего диалога:"
        summary_response = llm.predict(summary_prompt, messages[1:-1])
        return [
            messages[0],
            {"role": "assistant", "content": f"Резюме: {summary_response}"},
            messages[-1]
        ]
    ```

!!! tip "Оптимальный размер chunk"
    | Тип данных | Размер чанка |
    | --- | --- |
    | Код | 128–256 токенов |
    | Документация | 256–512 токенов |
    | Разговоры | 512–1024 токенов |
    | Длинные статьи | 512–1024 токенов |

!!! tip "Cost Optimization"
    - Используйте `max_tokens` для ограничения вывода
    - Выбирайте модель с нужным context window
    - Кэшируйте повторяющиеся промпты
    - Используйте более дешёвые модели для простых задач

    ```python
    def route_by_complexity(query):
        if len(query) < 100:
            return "gpt-4o-mini"
        elif len(query) < 1000:
            return "gpt-4o"
        else:
            return "gpt-4o-turbo"
    ```

## Оценка и выбор стека

!!! tip "LLM-as-Judge"
    Использование LLM для оценки качества ответов других LLM.

    ```python
    from openai import OpenAI

    def judge_response(question, answer):
        client = OpenAI()
        prompt = f"""
    Оцени ответ по шкале 1-10:
    Вопрос: {question}
    Ответ: {answer}

    Критерии:
    - Правильность
    - Полнота
    - Ясность

    Оцени и дай краткий комментарий.
    """
        response = client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        return response.choices[0].message.content
    ```

!!! info "Метрики"
    | Метрика | Описание |
    | --- | --- |
    | **Faithfulness** | Соответствие фактам |
    | **Hallucination rate** | % выдуманной информации |
    | **Latency** | Время отклика |
    | **Cost per 1k tokens** | Стоимость |
    | **Pass rate** | Проход тестов |

!!! tip "Выбор стека"
    | Задача | Инструменты |
    | --- | --- |
    | **Локальный запуск / R&D** | Ollama, LM Studio |
    | **RAG** | LangChain, LlamaIndex, Qdrant |
    | **Production API** | vLLM + Kubernetes/GPUStack |
    | **Кэширование** | LMCache |
    | **A/B тестирование / роутинг** | TensorZero |
    | **Fine-tuning** | Axolotl, Unsloth |
    | **Оценка** | LLM-as-Judge, RAGAS |

## OpenCode

!!! tip "Обзор"
    **Что это?** Open-source AI coding agent (140K+ GitHub stars). Доступен как CLI, desktop app и IDE extension.

    **Ключевые особенности:**
    - 75+ LLM провайдеров через Models.dev
    - LSP enabled — автоматически загружает нужные LSP для LLM
    - Multi-session — несколько агентов параллельно на одном проекте
    - Share links — расшарить сессию
    - Privacy-first — не хранит код и контекст

    **Установка:**
    ```bash
    curl -fsSL https://opencode.ai/install | sh
    ```

!!! tip "SKILL — переиспользуемые инструкции для AI агента"
    **Для чего используется:**
    - Следование командным конвенциям
    - Автоматизация процессов (release, review, deployment)
    - Стиль документации
    - Проектные workflow

    **Формат SKILL.md:**
    ```markdown
    ---
    name: git-release
    description: Create consistent releases and changelogs
    license: MIT
    compatibility: opencode
    metadata:
      audience: maintainers
      workflow: github
    ---

    ## What I do
    - Draft release notes from merged PRs
    - Propose a version bump
    - Provide a copy-pasteable `gh release create` command
    ```

    **Поля frontmatter:**

    | Field | Required | Notes |
    | --- | --- | --- |
    | `name` | ✅ | Lowercase, 1-64 chars, hyphen separators |
    | `description` | ✅ | 1-1024 chars, для выбора скилла агентом |
    | `license` | ❌ | Напр. MIT |
    | `compatibility` | ❌ | Напр. `opencode` |
    | `metadata` | ❌ | Key-value map |

    **Расположение (по приоритету):**
    - Project: `.opencode/skills/<skill-name>/SKILL.md`
    - Global: `~/.config/opencode/skills/<skill-name>/SKILL.md`
    - Claude-compatible: `.claude/skills/<skill-name>/SKILL.md`

    **Permissions (в opencode.json):**
    ```json
    {
      "permission": {
        "skill": {
          "*": "allow",
          "pr-review": "allow",
          "internal-*": "deny",
          "experimental-*": "ask"
        }
      }
    }
    ```

!!! tip "AGENT — специализированный AI агент"
    **Формат AGENT.md:**
    ```markdown
    ---
    name: code-reviewer
    description: Expert at reviewing code changes
    permission:
      skill:
        "pr-review": "allow"
        "security-*": "allow"
    tools:
      - read
      - grep
      - bash
    ---

    ## How I work
    1. Ищу изменения в diff
    2. Проверяю common issues
    3. Предлагаю улучшения
    ```

    **Расположение:**
    - `.opencode/agents/<agent-name>.md`
    - `~/.config/opencode/agents/<agent-name>.md`

!!! tip "MCP Servers"
    Model Context Protocol — открытый стандарт для подключения внешних инструментов к AI агентам (1200+ серверов).

    **Для чего используется:**
    - Интеграция с GitHub (issues, PRs, repos)
    - Работа с базами данных (PostgreSQL, SQLite)
    - Filesystem доступ
    - Browser automation (Playwright)
    - Sentry, Slack, Notion и др.

    **Типы MCP серверов:**

    | Server | Tools | Context Cost |
    | --- | --- | --- |
    | GitHub | Search code, issues, PRs | ~5K tokens |
    | PostgreSQL | Query, describe schemas | ~2K tokens |
    | Filesystem | Read/write files | ~500 tokens |
    | Playwright | Browser automation | ~3K tokens |
    | Sentry | Query errors | ~2K tokens |

    **Конфигурация local MCP:**
    ```json
    {
      "mcp": {
        "github": {
          "type": "local",
          "command": ["npx", "-y", "@modelcontextprotocol/server-github"],
          "env": {
            "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxx"
          }
        }
      }
    }
    ```

    **Конфигурация remote MCP:**
    ```json
    {
      "mcp": {
        "sentry": {
          "type": "remote",
          "url": "https://your-sentry-mcp.com",
          "headers": {
            "Authorization": "Bearer YOUR_API_KEY"
          }
        }
      }
    }
    ```

    **OAuth авторизация:**
    ```bash
    opencode mcp auth <server-name>
    ```

!!! info "Commands — slash commands"
    Определяются через Markdown файлы.

    **Расположение:**
    - `.opencode/commands/<command-name>.md`
    - `~/.config/opencode/commands/<command-name>.md`

    **Пример:**
    ```markdown
    ---
    name: /review
    description: Run code review on current changes
    ---

    ## Steps
    1. Get git diff
    2. Run pr-review skill
    3. Run security-audit skill
    4. Format summary
    ```

!!! info "Plugins"
    Расширения функциональности на TypeScript или npm пакетах.

    **Расположение:**
    - `.opencode/plugins/<plugin-name>.ts`
    - `~/.config/opencode/plugins/`

    **Конфигурация (opencode.json):**
    ```json
    {
      "plugins": {
        "my-plugin": {
          "enabled": true
        }
      }
    }
    ```

!!! tip "Configuration (opencode.json)"
    **Основные секции:**
    ```json
    {
      "$schema": "https://opencode.ai/config.json",
      "model": "claude-3-5-sonnet",
      "instructions": ["CONTRIBUTING.md", ".cursor/rules/*.md"],
      "mcp": { ... },
      "agents": { ... },
      "permission": {
        "tool": { "*": "allow" },
        "skill": { "*": "allow" }
      },
      "keybinds": { ... },
      "theme": "dark"
    }
    ```

    **Директории конфигурации:**
    ```text
    ~/.config/opencode/           # Global
    ├── opencode.json
    ├── AGENTS.md
    ├── agents/
    ├── commands/
    ├── plugins/
    ├── skills/
    ├── tools/
    └── themes/

    .opencode/                    # Project (traverses to git root)
    ├── agents/
    ├── commands/
    ├── plugins/
    ├── skills/
    └── themes/
    ```

!!! tip "Tool Access Control"
    **Permissions в opencode.json:**
    ```json
    {
      "permission": {
        "tool": {
          "*": "allow",
          "bash": "ask",
          "write": "deny"
        },
        "skill": {
          "*": "allow",
          "internal-*": "deny"
        }
      }
    }
    ```

    **Уровни доступа:**
    - `allow` — разрешено
    - `ask` — запросить подтверждение
    - `deny` — запрещено
