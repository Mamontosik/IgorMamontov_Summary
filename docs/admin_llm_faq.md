# LLM FAQ

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
```
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
- Совместное использование кэ��а между запросами

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
- ` replicas` - количество реплик
- `resources.nvidia.com/gpu` - запрос GPU-ресурсов

---

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

**Прим��р Modelfile:**
```
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

## Выбор стека

| Задача | Инструменты |
| --- | --- |
| Локальный запуск / R&D | Ollama, LM Studio |
| Production API | vLLM + Kubernetes/GPUStack |
| Кэширование | LMCache |
| A/B тестирование / роутинг | TensorZero |