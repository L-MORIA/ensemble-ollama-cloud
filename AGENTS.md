# AGENTS.md — Ensemble Ollama Cloud

## Обзор проекта

Ensemble Ollama Cloud — скил для Hermes Agent, реализующий ансамбль моделей через Ollama Cloud. Две бесплатные облачные модели решают задачу независимо, арбитр Nemotron-3 Ultra выбирает лучшее решение.

## Ключевые файлы

| Файл | Назначение |
|------|------------|
| `SKILL.md` | Скил для Hermes Agent |
| `README.md` | Инструкция для пользователя |
| `ARCHITECTURE.md` | Архитектура и схема работы |
| `TEST_REPORT.md` | Результаты тестирования |
| `AGENTS.md` | Этот файл |

## Технологии

- **Hermes Agent** — фреймворк
- **Ollama Desktop** v0.30.9 — локальный сервер
- **Ollama Cloud** — облачные модели (`:cloud`)
- **Модели:** minimax-m3, qwen3-coder-next, nemotron-3-ultra

## Установка

```bash
# 1. Настроить провайдера
hermes config set custom_providers '[...]'

# 2. Установить скил
hermes skills install https://github.com/L-MORIA/ensemble-ollama-cloud/raw/main/SKILL.md
```

## Запуск

```bash
# В сессии Hermes
/skill ensemble-ollama-cloud
```

## Тестирование

Полный отчёт — TEST_REPORT.md. Протестировано 3 задачи, сравнение с llm-ensemble (OpenCode Zen).

## Публикация

```bash
hermes skills publish ./SKILL.md --to github --repo L-MORIA/ensemble-ollama-cloud
```
