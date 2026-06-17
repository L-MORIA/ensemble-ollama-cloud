# Архитектура Ensemble Ollama Cloud

## Схема работы

```
Пользователь: задача
    │
    ▼
┌──────────────────────────────────┐
│         ДИСПЕТЧЕР                │
│    (ensemble-ollama-cloud)       │
└────────────┬─────────────────────┘
             │
      ┌──────┴──────┐
      ▼              ▼
┌───────────┐  ┌───────────┐
│ minimax-m3│  │qwen3-     │
│ :cloud    │  │coder-next │
│           │  │:cloud     │
│ 12-37s    │  │ 3-6s      │
│ 1M ctx    │  │ 262K ctx  │
└─────┬─────┘  └─────┬─────┘
      │              │
      ▼              ▼
   Решение A      Решение B
      │              │
      └──────┬───────┘
             ▼
┌──────────────────────────────────┐
│     🏆 АРБИТР                    │
│  nemotron-3-ultra:cloud          │
│  NVIDIA 550B / 1M ctx / 55-90s   │
│  Оценивает по: безопасность >    │
│  качество > производительность   │
└────────────┬─────────────────────┘
             │
             ▼
     ┌──────────────┐
     │  ИТОГОВОЕ    │
     │  РЕШЕНИЕ     │
     │  + анализ    │
     └──────────────┘
```

## Компоненты

### 1. Ollama Desktop (port 11434)
Локальный сервер Ollama. Проксирует запросы к облачным моделям (`:cloud`). Требует:
- Установленное приложение Ollama
- Активный аккаунт на ollama.com
- Запущенный сервер (автоматически при старте приложения)

### 2. Облачные модели (ollama.com)
Модели запускаются на серверах ollama.com. Суффикс `:cloud` указывает использовать облачную версию.

| Модель | Размер | За что отвечает |
|--------|--------|-----------------|
| `minimax-m3:cloud` | ~1M ctx | Детальные ответы, 4+ варианта |
| `qwen3-coder-next:cloud` | 262K ctx | Быстрый чёткий код |
| `nemotron-3-ultra:cloud` | 550B / 1M ctx | Анализ и арбитраж |

### 3. Hermes Custom Provider
Провайдер `ollama-cloud` в Hermes config.yaml связывает Hermes с Ollama API.

```yaml
custom_providers:
  - name: ollama-cloud
    base_url: http://localhost:11434
    api_key: ''
    models:
      minimax-m3:cloud:
        context_length: 1048576
      nemotron-3-ultra:cloud:
        context_length: 1000000
      qwen3-coder-next:cloud:
        context_length: 262144
```

## Поток данных

```
Задача (текст)
  → [Диспетчер] формирует curl-запросы
    → [Ollama API] POST /api/generate
      → [Ollama Cloud] запускает модель
    → [Агенты] возвращают решения
  → [Арбитр] оценивает и выбирает
  → [Диспетчер] выводит результат
```

## Сравнение с llm-ensemble (OpenCode Zen)

| Параметр | llm-ensemble | ensemble-ollama-cloud |
|----------|-------------|----------------------|
| Провайдер | OpenCode CLI → OpenCode Zen | Ollama Desktop → Ollama Cloud |
| Агентов | 3 | 2 |
| Среднее время | ~3.5 мин | ~1.5 мин |
| Стабильность | Высокая | Средняя |
| Уникальность | Big Pickle, MiMo V2.5 | minimax-m3 (4+ варианта) |

## Ограничения

- Зависит от запущенного Ollama Desktop
- Бесплатные модели имеют rate limit
- minimax-m3 иногда уходит в оффтопик
- Nemotron-3 Ultra медленный (до 90s)
