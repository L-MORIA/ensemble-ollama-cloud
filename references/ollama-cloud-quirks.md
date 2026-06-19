# Ollama Cloud API quirks

## Как вызывать облачные модели

**Ключевое правило:** обязательно добавлять суффикс `:cloud` к имени модели.

```bash
# ✅ Правильно — облачная модель
curl -X POST http://localhost:11434/api/generate \
  -d '{"model":"minimax-m3:cloud","prompt":"task","stream":false}'

# ❌ Неправильно — ищет локально, model not found
curl -X POST http://localhost:11434/api/generate \
  -d '{"model":"minimax-m3","prompt":"task","stream":false}'
```

## Доступные облачные модели

Проверить список:
```bash
curl -s 'https://ollama.com/api/tags?cloud=true'
```

Бесплатные рабочие (проверено):
- `minimax-m3:cloud` — 1M ctx, кодинг, детальные ответы (12-37s)
- `qwen3-coder-next:cloud` — 262K ctx, быстрый кодинг (3-6s)
- `nemotron-3-super:cloud` — NVIDIA, 1M ctx, арбитр (55-90s)

Требуют подписку:
- `deepseek-v4-flash:cloud` — ❌ subscription required
- `kimi-k2.7-code:cloud` — ❌ not available

Нестабильны:
- `nemotron-3-super:cloud` — иногда отвечает, иногда пустой ответ

## minimax-m3 особенности

- На русские промпты может уходить в оффтопик (генерация нерелевантного кода)
- **Рекомендация:** формулировать задачу на английском
- 1M контекст — подходит для больших задач
- Возвращает 4+ варианта решения (diversity)

## Nemotron-3 Ultra особенности

- Медленный (55-90s на один запрос)
- Иногда возвращает пустой ответ (timeout) — повторить
- При повторении с тем же промптом может ответить
- Хорошо аргументирует выбор, синтезирует гибриды

## Rate limits

- Бесплатные модели ~20 запросов/мин
- При превышении — `429 Too Many Requests`
- Нужно подождать 10-30 сек
- При параллельных запросах ошибка 429 вероятнее

## Подключение к Hermes

```bash
hermes config set custom_providers '[{"name":"ollama-cloud","base_url":"http://localhost:11434","api_key":"","models":{"minimax-m3:cloud":{"context_length":1048576},"nemotron-3-super:cloud":{"context_length":1000000},"qwen3-coder-next:cloud":{"context_length":262144}}}]'
```

Проверить: `hermes model` → должен появиться "Ollama Cloud".

## Диагностика

```bash
# 1. Сервер работает?
ollama ps

# 2. API отвечает?
curl http://localhost:11434/api/tags

# 3. Модель отвечает?
curl -X POST http://localhost:11434/api/generate \
  -d '{"model":"minimax-m3:cloud","prompt":"say OK","stream":false}'
```

**Важно:** для работы облачных моделей Ollama Desktop должен быть запущен и пользователь должен быть авторизован (`ollama signin`).
