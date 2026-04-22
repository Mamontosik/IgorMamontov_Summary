# LLM FAQ

## Модели LLM

| Название | Описание | Примеры применения | Связанные инструменты |
| --- | --- | --- | --- |
| **Ollama** | CLI/сервер для локальных LLM (Llama, Mistral, Gemma и др.) | Локальный запуск, R&D прототипирование | llama.cpp, LM Studio |
| **LM Studio** | GUI для локальных LLM (GGUF + llama.cpp) | Локальный запуск на CPU/GPU, разработка | Ollama, RAG |
| **TensorZero** | LLMOps платформа (A/B тесты, роутинг, Autopilot) | Продакшн LLM с A/B тестированием | vLLM, Ollama, OpenAI |

### Ollama

**Что это?** CLI-инструмент и сервер для запуска локальных больших языковых моделей. Использует llama.cpp для эффективного инференса и поддерживает множество моделей (Llama, Mistral, Gemma, Qwen и др.). Прост в установке и использовании.

**Для чего используется?** Локальный запуск LLM, R&D прототипирование, тестирование моделей без облачных сервисов.

**Ключевые особенности:**

- Простой CLI-интерфейс
- Встроенный HTTP-сервер
- Поддержка GPU (NVIDIA, AMD, Apple Silicon)
- Модель Modelfile для настройки
- Множество готовых моделей

**Команда установки:**

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**Команда запуска:**

```bash
ollama run llama3
ollama serve
```

**Полезные команды:**

```bash
ollama list           # Список установленных моделей
ollama pull <model>  # Скачать модель
ollama rm <model>    # Удалить модель
```

**Конфигурация (Modelfile):**

- `temperature` - креативность модели (0-2)
- `num_ctx` - размер контекста
- `num_gpu` - количество GPU для инференса
- `top_p` - nucleus sampling
- `top_k` - top-k sampling
- `repeat_penalty` - штраф за повторы

**Пример Modelfile:**

```bash
FROM llama3
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
SYSTEM You are a helpful AI assistant.
```

### LM Studio

**Что это?** Графический интерфейс (GUI) для запуска локальных LLM с использованием формата GGUF и llama.cpp. Отлично подходит для разработки и тестирования моделей без командной строки.

**Для чего используется?** Локальный запуск LLM на CPU/GPU, разработка и тестирование, прототипирование RAG-систем.

**Ключевые особенности:**

- Удобный GUI-интерфейс
- Поддержка GGUF-формата
- Выбор GPU-бэкенда (Metal, CUDA, ROCm)
- Встроенный чат-интерфейс
- Локальный OpenAI-совместимый API

**Установка:** .exe (Windows), .dmg (macOS), .AppImage (Linux)

**Конфигурация:**

- `GPU-backend` - тип GPU (Metal/CUDA/ROCm)
- `offloading` - выгрузка слоев на GPU
- `context size` - размер контекста
- `GPU layers` - количество слоев на GPU

**API-доступ:**

```
localhost:1234/v1/chat/completions
localhost:1234/v1/completions
```

### TensorZero

**Что это?** Open-source LLMOps-платформа для production LLM. Включает A/B-тестирование, умный роутинг между моделями и Autopilot для автоматической оптимизации промптов и выбора модели.

**Для чего используется?** Production LLM с A/B-тестированием, роутинг запросов между моделями, автоматическая оптимизация промптов.

**Ключевые особенности:**

- A/B-тестирование моделей
- Умный роутинг между LLM
- Autopilot (автооптимизация промптов)
- Сбор и анализ метрик
- Поддержка множества бэкендов
- OpenAI-совместимый API

**Поддерживаемые бэкенды:** Ollama, vLLM, OpenAI, Anthropic и др.

**Команда:** Конфигурация gateway.yaml

**Конфигурация:**

- `Router` - правила роутинга
- `Autopilot` - политики автооптимизации
- `datasets` - наборы данных для тестирования
- `alerts` - оповещения

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

---

## RAG

### Концепция

**Что это?** Retrieval-Augmented Generation (RAG) — подход к работе с LLM, при котором модель дополняет свои знания внешней информацией из базы знаний в реальном времени. Вместо переобучения модели под каждую задачу, RAG позволяет "подключить" нужные данные в момент запроса.

**Когда использовать RAG:**

- Базы знаний, документация, FAQ
- Корпоративные данные
- Информация, которая часто обновляется
- Нужно ссылаться на конкретные источники

**RAG vs Fine-tuning vs Промпты:**

| Подход | Когда использовать |
| --- | --- |
| **Промпты** | Простые задачи, модель уже знает нужную информацию |
| **RAG** | Нужны актуальные данные или специфическая база знаний |
| **Fine-tuning** | Нужно изменить поведение/стиль модели, особые форматы вывода |

### Архитектура RAG

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Документы   │ →  │  Chunking   │ →  │ Embedding   │ →  │ Vector DB   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                               ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Ответ     │ ←  │  Генерация  │ ←  │  Поиск      │ ←  │  Запрос     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### Chunking

**Что это?** Разбиение документов на меньшие фрагменты (chunks) для индексации. Каждый чанк должен содержать законченную смысловую единицу.

**Стратегии:**

- **По тексту** — абзацы, параграфы
- **По токенам** — фиксированное количество токенов (256-1024)
- **Рекурсивное** — иерархическое разбиение (по заголовкам, предложениям)

**Пример кода:**

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " "]
)

chunks = splitter.split_text(long_document)
```

### Vector DB

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

collection.add(
    documents=["текст документа"],
    ids=["doc1"]
)
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

### Embedding модели

| Модель | Размерность | Особенности |
| --- | --- | --- |
| **sentence-transformers/all-MiniLM-L6-v2** | 384 | Быстрая, хорошее качество |
| **BAAI/bge-small-en-v1.5** | 1024 | Высокое качество, RAG |
| **intfloat/e5-small-v2** | 384 | Multilingual |
| **text-embedding-3-small** | 1536 | OpenAI API |

### Поиск

**Semantic Search** — поиск по смыслу, а не по ключевым словам.

```python
results = collection.query(
    query_texts=["запрос пользователя"],
    n_results=5
)
```

**Hybrid Search** — комбинация семантического и текстового поиска.

```python
from rankify import HybridRetriever

retriever = HybridRetriever(
    semantic_model=embedding_model,
    keyword_index=keyword_index,
    weights=[0.7, 0.3]
)

results = retriever.retrieve("запрос")
```

**Re-ranking** — переранжировка результатов для повышения точности.

```python
from cohere import Client

reranker = Client(api_key="cohere-key")
results = reranker.rerank(
    query="запрос",
    documents=initial_results,
    top_n=3
)
```

### Инъекция контекста

**Пример LangChain:**

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

**Пример LlamaIndex:**

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

### Фреймворки

**LangChain** — универсальный фреймворк для работы с LLM.

```python
from langchain.schema import Document
from langchain_community.vectorstores import Chroma

docs = [Document(page_content=chunk) for chunk in chunks]
vectorstore = Chroma.from_documents(docs, embedding)
```

**LlamaIndex** — специализированный фреймворк для RAG.

```python
from llama_index import SimpleDirectoryReader
from llama_index import VectorStoreIndex

reader = SimpleDirectoryReader(input_files=["doc.txt"])
documents = reader.load_data()
index = VectorStoreIndex.from_documents(documents)
```

### RAG vs Memory

| Характеристика | RAG | Memory |
| --- | --- | --- |
| **Данные** | База знаний | История чата |
| **Назначение** | Факты, документы | Контекст диалога |
| **Обновление** | Периодическое | После каждого сообщения |

### Метрики RAG

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

---

## Обслуживание LLM

| Название | Описание | Примеры применения | Связанные инструменты |
| --- | --- | --- | --- |
| **vLLM** | Высокопроизводительный движок инференса LLM с PagedAttention, Continuous Batching и OpenAI-совместимым API | Production API для LLM (чат-боты, RAG, агенты, поисковые системы) | LMCache, TensorZero, GPUStack |
| **LMCache** | Кэширование KV-кеша LLM на DRAM/диск | Повторные запросы, RAG, чат-сессии | vLLM, RAG-системы |
| **GPUStack** | Оркестратор GPU (open-source) | Управление несколькими GPU, Kubernetes | vLLM, Kubernetes |

### vLLM

**Что это?** Высокопроизводительный движок инференса LLM с открытым исходным кодом, разработанный для production-нагрузок. Использует технологию PagedAttention для эффективного управления памятью и поддерживает Continuous Batching для обработки множества параллельных запросов.

**Для чего используется?** Развертывание production API для LLM-приложений: чат-боты, RAG-системы, агенты, поисковые системы с ИИ.

**Ключевые особенности:**

- PagedAttention - эффективное управление KV-кешем
- Continuous Batching - динамическая обработка запросов
- OpenAI-совместимый API (/v1/chat/completions, /v1/completions)
- Tensor Parallelism - распределение модели на несколько GPU
- Поддержка множества моделей: Llama, Mistral, Qwen и др.

**Команда запуска:**

```bash
docker run -it --gpus all -p 8000:8000 vllm/vllm-openai --model meta-llama/Llama-3-8b-instruct
```

**Ключевые параметры:**

- `tensor-parallelism` - количество GPU для одной модели
- `max_num_seqs` - максимальное количество параллельных запросов
- `max_model_len` - максимальная длина контекста
- `max_num_batched_tokens` - максимальное количество токенов в батче

**Интеграция с OpenAI SDK:**

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")
response = client.chat.completions.create(model="Llama-3-8b", messages=[...])
```

### LMCache

**Что это?** Система кэширования KV-кеша для LLM. Позволяет сохранять и повторно использовать вычисленные ключи и значения внимания между запросами, что значительно ускоряет обработку повторяющихся или похожих запросов.

**Для чего используется?** Ускорение повторных запросов, RAG-систем с длинными контекстами, многоходовых чат-сессий.

**Ключевые особенности:**

- Кэширование на DRAM или диске
- Поддержка различных бэкендов (Redis, диск)
- Политика вытеснения (LRU, LFU и др.)
- Интеграция с vLLM, Ollama
- Совместное использоваие кэша между запросами

**Интеграция:** Redis или диск backend

**Ключевые параметры:**

- `cache_size` - размер кэша
- `eviction_policy` - политика вытеснения (LRU, LFU)
- `ttl` - время жизни записей
- `backend` - тип хранилища (redis, disk)

**Пример использования с Redis:**

```bash
# Запуск с Redis
redis-server --port 6379
# Подключение к LMCache
```

### GPUStack

**Что это?** Open-source оркестратор GPU с поддержкой нескольких видеокарт и интеграцией с Kubernetes. Позволяет управлять GPU-ресурсами в кластере и распределять нагрузку между несколькими узлами.

**Для чего используется?** Управление несколькими GPU, развертывание в Kubernetes-кластерах, масштабирование инференса.

**Ключевые особенности:**

- Поддержка NVIDIA и AMD GPU
- Интеграция с Kubernetes
- Автоматическое распределение нагрузки
- Мониторинг GPU-ресурсов
- Open-source решение

**Команда:** Kubernetes с GPU-node

**Ключевые параметры:**

- `GPU node config` - конфигурация GPU-узлов
- `replicas` - количество реплик
- `resources.nvidia.com/gpu` - запрос GPU-ресурсов

---

## Fine-tuning

### Когда применять

| Подход | Когда использовать |
| --- | --- |
| **Промпты** | Базовая функциональность, модель уже знает |
| **RAG** | Нужны внешние данные, актуальная информация |
| **Fine-tuning** | Особый стиль, формат, поведение модели |

**Когда точно нуженFine-tuning:**

- Специфический формат вывода (JSON, XML)
- Особый стиль общения
- narrow domain expertise
- Сложные задачи

### Типы Fine-tuning

**SFT (Supervised Fine-Training):** Классическое дообучение на парах (input, output).

**DPO (Direct Preference Optimization):** Оптимизация на основе предпочтений (какой ответ лучше).

**RLHF (Reinforcement Learning from Human Feedback):** Обучение с подкреплением от человека.

### Инструменты

| Инструмент | Описание |
| --- | --- |
| **Axolotl** | Универсальный инструмент для fine-tuning |
| **Unsloth** | Быстрый fine-tuning (до 2x быстрее, меньше памяти) |
| **OpenAI API** | Fine-tuning через OpenAI API |
| **DeepSpeed** | Distributed training от Microsoft |

### Пример: Axolotl

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

### Пример: Unsloth

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

# Обучение с SFTTrainer
```

---

## Управление контекстом

### Context Window

**Лимиты популярных моделей:**

| Модель | Context Window |
| --- | --- |
| GPT-4 | 128k токенов |
| Claude 3.5 | 200k токенов |
| Gemini 1.5 | 2M токенов |
| Llama 3.1 | 128k токенов |
| Mistral Large | 32k токенов |

### Стратегии управления

**Truncation** — обрезать контекст до лимита:

```python
def truncate_context(messages, max_tokens):
    total_tokens = sum(len(tokenizer.encode(m["content"])) for m in messages)
    while total_tokens > max_tokens and len(messages) > 1:
        messages.pop(1)  # Удаляем старые сообщения
        total_tokens = sum(len(tokenizer.encode(m["content"])) for m in messages)
    return messages
```

**Rolling Window** — оставляем последние N сообщений:

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
        messages[0],  # system
        {"role": "assistant", "content": f"Резюме: {summary_response}"},
        messages[-1]  # последний запрос
    ]
```

### Оптимальный размер chunk

| Тип данных | Размер чанка |
| --- | --- |
| Код | 128-256 токенов |
| Документация | 256-512 токенов |
| Разговоры | 512-1024 токенов |
| Длинные статьи | 512-1024 токенов |

### Cost Optimization

**Советы:**

- Используйте `max_tokens` для ограничения вывода
- Выбирайте модель с нужным context window
- Кэшируйте повторяющиеся промпты
- Используйте более дешёвые модели для простых задач

```python
# Пример: разные модели для разных задач
def route_by_complexity(query):
    if len(query) < 100:
        return "gpt-4o-mini"
    elif len(query) < 1000:
        return "gpt-4o"
    else:
        return "gpt-4o-turbo"
```

---

## Оценка и выбор стека

### LLM-as-Judge

Использование LLM для оценки качества ответов других LLM.

**Пример:**

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

### Метрики

| Метрика | Описание |
| --- | --- |
| **Faithfulness** | Соответствие фактам |
| **Hallucination rate** | % выдуманной информаци |
| **Latency** | Время отклика |
| **Cost per 1k tokens** | Стоимость |
| **Pass rate** | Проход тестов |

### Выбор стека

| Задача | Инструменты |
| --- | --- |
| **Локальный запуск / R&D** | Ollama, LM Studio |
| **RAG** | LangChain, LlamaIndex, Qdrant |
| **Production API** | vLLM + Kubernetes/GPUStack |
| **Кэширование** | LMCache |
| **A/B тестирование / роутинг** | TensorZero |
| **Fine-tuning** | Axolotl, Unsloth |
| **Оценка** | LLM-as-Judge, RAGAS |