# Ensemble Ollama Cloud

**Одна задача → 2 облачные модели → 🏆 арбитр Nemotron-3 Ultra выбирает лучшее.**

Скил для [Hermes Agent](https://hermes-agent.nousresearch.com), реализующий ансамбль моделей через **Ollama Cloud** — бесплатные облачные модели с ограниченным доступом.

## Возможности

- 🤖 **2 агента** — minimax-m3 (1M ctx, детальные ответы) + qwen3-coder-next (быстрый кодинг)
- 🏆 **Арбитр Nemotron-3 Ultra** — NVIDIA, глубокая аналитика
- ⚡ **~1.5 мин на задачу**
- 🔄 **Дополняет OpenCode Zen** — minimax-m3 даёт другой diversity

## Быстрый старт

### 1. Настройка провайдера

```bash
hermes config set custom_providers '[{"name":"ollama-cloud","base_url":"http://localhost:11434","api_key":"","models":{"minimax-m3:cloud":{"context_length":1048576},"nemotron-3-super:cloud":{"context_length":1000000},"qwen3-coder-next:cloud":{"context_length":262144}}}]'
```

### 2. Установка скила

```bash
# Из репозитория
hermes skills install https://github.com/L-MORIA/ensemble-ollama-cloud/raw/main/SKILL.md
```

### 3. Загрузка в сессию

```bash
/skill ensemble-ollama-cloud
```

### 4. Запуск

```
Задача: "Напиши парсер CSV на Python"
→ Agent A (minimax-m3): 4 варианта решения
→ Agent B (qwen3-coder-next): clean code
→ 🏆 Арбитр: выбрал решение B — безопаснее
```

## Модели

| Роль | Модель | Время | Контекст |
|------|--------|-------|----------|
| 🏆 **Арбитр** | `nemotron-3-super:cloud` | 55-90s | 1M |
| 🤖 **Агент A** | `minimax-m3:cloud` | 12-37s | 1M |
| 🤖 **Агент B** | `qwen3-coder-next:cloud` | 3-6s | 262K |

## Требования

- Ollama Desktop (установлен, аккаунт зарегистрирован)
- Ollama сервер запущен (port 11434)
- Hermes (любая версия)

## Структура проекта

```
ensemble-ollama-cloud/
├── SKILL.md           # Сам скил
├── README.md          # Инструкция
├── ARCHITECTURE.md    # Архитектура
├── AGENTS.md          # Для AI-агентов
├── TEST_REPORT.md     # Отчёт тестов
├── LICENSE            # MIT
└── .gitignore
```

## Лицензия

MIT
