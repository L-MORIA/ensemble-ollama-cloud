---
name: ensemble-ollama-cloud
description: "Ensemble через Ollama Cloud: 2 бесплатные облачные модели (minimax-m3, qwen3-coder-next) + арбитр Nemotron-3 Super. Для задач, где нужен diversity или OpenCode Zen недоступен."
version: 1.1.0
author: Hermes Agent
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [ensemble, ollama, cloud, arbiter, multi-model, coding, minimax, nemotron]
    related_skills: [llm-ensemble, subagent-driven-development, plan]
---

# Ensemble Ollama Cloud

Ансамбль моделей через **Ollama Cloud** — бесплатные облачные модели с ограниченным доступом.

**Одна задача → 2 облачные модели → 🏆 арбитр Nemotron-3 Super выбирает или синтезирует лучшее.**

## Зачем нужен этот скил, если есть llm-ensemble?

| llm-ensemble (OpenCode Zen) | ensemble-ollama-cloud |
|----------------------------|----------------------|
| 3 агента (DeepSeek, MiMo, Big Pickle) | 2 агента (minimax-m3, qwen3-coder-next) |
| ~3.5 мин на задачу | **~1.5 мин на задачу** |
| Всегда стабильно | minimax-m3 иногда уходит в оффтопик |
| | **minimax-m3 даёт 4+ варианта решения** |

**Лучшая стратегия:** использовать оба скила в зависимости от задачи.

## Модели

Все модели доступны через Ollama Cloud (суффикс `:cloud` = облачные, не локальные).

| Роль | Модель | Характеристика |
|------|--------|----------------|
| 🏆 **Арбитр** | `nemotron-3-super:cloud` | NVIDIA, 1M ctx, глубокая аналитика, 30-90s |
| 🤖 **Агент A** | `minimax-m3:cloud` | 1M ctx, отлична для кода, даёт 4+ варианта |
| 🤖 **Агент B** | `qwen3-coder-next:cloud` | Быстрый кодинг-специалист, 262K ctx |

> ❌ `deepseek-v4-flash:cloud` — требует подписку.
> ❌ `kimi-k2.7-code:cloud` — не отвечает.

## Требования

- **Ollama Desktop** — установлен, аккаунт зарегистрирован
- **Ollama сервер** — запущен (port 11434)
- **Hermes custom_provider** — `ollama-cloud` настроен в config.yaml

### Настройка провайдера в Hermes

```bash
hermes config set custom_providers '[{"name":"ollama-cloud","base_url":"http://localhost:11434","api_key":"","models":{"minimax-m3:cloud":{"context_length":1048576},"nemotron-3-super:cloud":{"context_length":1000000},"qwen3-coder-next:cloud":{"context_length":262144}}}]'
```

Проверить: `hermes model` → должен появиться "Ollama Cloud" в списке.

## Процедура

### Шаг 1 — Формулировка задачи

Чёткая задача на русском или английском.

### Шаг 2 — Запуск агентов

```bash
# Агент A: minimax-m3
curl -s -X POST http://localhost:11434/api/generate \
  -d '{"model":"minimax-m3:cloud","prompt":"ЗАДАЧА","stream":false}'

# Агент B: qwen3-coder-next
curl -s -X POST http://localhost:11434/api/generate \
  -d '{"model":"qwen3-coder-next:cloud","prompt":"ЗАДАЧА","stream":false}'
```

### Шаг 3 — Арбитр

```bash
curl -s -X POST http://localhost:11434/api/generate \
  -d '{"model":"nemotron-3-super:cloud","prompt":"Ты арбитр. Оцени решения...","stream":false}'
```

### Шаг 4 — Вывод результата

Сравнить решения, предъявить итог.

## Ограничения

- Бесплатные облачные модели имеют rate limit (~20 запросов/мин)
- minimax-m3 иногда уходит в оффтопик на русских промптах — лучше писать на английском
- Nemotron-3 Super медленный (30-90s на задачу)
- Работает только при запущенном Ollama Desktop
- Без суффикса `:cloud` модель не найдётся

## Производительность (замеры на 2026-06-19)

| Модель | Простая задача | Токенов | Сложная задача | Токенов |
|--------|---------------|---------|----------------|---------|
| `minimax-m3:cloud` | 3s | 55 | 147s | 5 471 |
| `qwen3-coder-next:cloud` | 0.6s | 19 | 12s | 1 220 |
| `nemotron-3-super:cloud` | 4s | 35 | 36s | 1 136 |

> Простая задача: is_palindrome. Сложная задача: Sliding Window Log Rate Limiter.
> Токены = количество сгенерированных токенов за запрос. Время = полное, включая сеть.
> Для простых задач ensemble не оправдан — используйте одну модель.

## Принцип безопасности

Безопасность — приоритет №1. Арбитр оценивает решения по критерию:
`безопасность > качество > производительность > читаемость`

## Common Pitfalls

1. **Забыть `:cloud`** — без суффикса модель не найдётся
2. **Не проверять перед утверждением** — всегда тестировать модель через API, прежде чем писать, что она работает
3. **Ollama Desktop не запущен** — сервер на 11434 не ответит
4. **minimax-m3 уходит в оффтопик** — используй английские промпты
5. **Rate limit** — при параллельном запуске 429 Too Many Requests
6. **Nemotron-3 Super иногда timeout** — повторить запрос (30-90s)
7. **Не для простых задач** — ensemble не оправдан
8. **Названия моделей меняются** — всегда проверять через `curl localhost:11434/api/tags` перед записью в скил

## Verification Checklist

- [ ] `ollama ps` — сервер работает
- [ ] `curl http://localhost:11434/api/tags` — API отвечает
- [ ] `curl -X POST ... minimax-m3:cloud` — модель отвечает
- [ ] `curl -X POST ... qwen3-coder-next:cloud` — модель отвечает
- [ ] `curl -X POST ... nemotron-3-super:cloud` — модель отвечает
- [ ] `hermes model` → Ollama Cloud в списке провайдеров

## Reference

- `references/ollama-cloud-quirks.md` — полный справочник по API, rate limits, диагностике
