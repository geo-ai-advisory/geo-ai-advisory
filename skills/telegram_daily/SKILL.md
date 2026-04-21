---
name: telegram_daily
description: Use when user invokes /telegram_daily or asks for a Telegram daily digest — mentions, direct tags, unanswered questions in personal chats, and summary of target groups.
---

# Telegram Daily Digest

## Overview

Получает сводку активности в Telegram за последние 24 часа:
1. **Кто тебя тегал** — прямые упоминания @username во всех чатах
2. **Сводка целевых групп** — все сообщения из групп, содержащих ключевое слово (задаётся пользователем)
3. **Вопросы без ответа** — личные чаты где тебе написали, но ответа нет

**Команда:** `/telegram_daily [keyword]`

- Без аргумента — используйте ключевое слово из вашего конфига (подставьте название своей компании/проекта)
- С аргументом: `/telegram_daily Acme` — ищет группы содержащие "Acme"

**Требования:** Telegram MCP сервер (инструкция установки ниже в этом файле)

---

## Пошаговый сценарий

### 1. Вызвать get_daily_summary

```
mcp__telegram__get_daily_summary()
```

Результат может быть большим (>50k символов) и сохранится в файл. Обрабатывать через Python, не читать полностью.

### 2. Извлечь данные из файла

Если результат сохранился в файл — читать его через Python-скрипт:

```python
import json, sys

with open('<PATH_TO_FILE>') as f:
    data = json.load(f)
text = data['result']

MY_USERNAME = 'your_username'  # из get_me() — определяется автоматически

# Секция упоминаний
idx_target = text.find('ЦЕЛЕВЫЕ ГРУППЫ') or text.find('GROUPS')
mentions_text = text[:idx_target] if idx_target != -1 else text

# Прямые теги (не от самого пользователя)
direct = []
for line in mentions_text.split('\n'):
    if f'@{MY_USERNAME}' in line and f'({MY_USERNAME})' not in line.split('|')[0] if '|' in line else True:
        direct.append(line.strip())

# Целевая секция
target_text = text[idx_target:] if idx_target != -1 else ''

print('DIRECT:', '\n'.join(direct))
print('TARGET:', target_text[:5000])
```

### 3. Если результат пришёл напрямую (не файл)

Разбить на секции по разделителю `╔` или ключевым словам `ЗАДАЧИ` / `ГРУППЫ`, обработать аналогично.

### 4. Сформировать итоговый дайджест

Вывести пользователю в формате:

```
## 📬 Прямые обращения к @{username}

**1. [ЧЧ:ММ] {Чат} — {Отправитель}**
"{текст сообщения}"
→ [требует ответа / информационно]

...

## 📊 Сводка групп [{keyword}] — {N} групп, {M} сообщений

**{Название группы} ({K} сообщений)**
- [ЧЧ:ММ] {Автор}: {краткое содержание}
...

## ⚠️ Требуют ответа
- {список чатов где тебя ждут}
```

### 5. Расставить приоритеты

При подаче итога всегда выделить:
- Сообщения с вопросами ("?", "подскажи", "когда", "можно", "ждём") — **требуют ответа**
- Упоминания от внешних партнёров (не коллег) — **высокий приоритет**
- Информационные сообщения — отдельным блоком, без call-to-action

---

## Настройка ключевого слова для групп

По умолчанию скилл фильтрует группы по keyword, заданному пользователем. Чтобы изменить — передать как аргумент:
- `/telegram_daily` → без фильтра / фильтр из конфига
- `/telegram_daily MyCompany` → фильтр "MyCompany"

Если MCP сервер поддерживает параметр `group_filter` — передавать его напрямую.
Если нет — фильтровать вручную из `get_daily_summary` по ключевому слову в названии чата.

---

## Установка Telegram MCP

### Шаг 1. Получить API-ключи Telegram

1. Открыть [https://my.telegram.org](https://my.telegram.org)
2. Войти под своим номером телефона
3. Перейти в **API development tools**
4. Создать новое приложение (любое название, например "claude-mcp")
5. Скопировать `App api_id` и `App api_hash`

### Шаг 2. Создать структуру директорий

```bash
mkdir -p ~/.claude/mcp-servers/telegram/session
```

### Шаг 3. Собрать сервер Telegram MCP

Вариант A — готовый open-source Telegram MCP:
- https://github.com/chaindead/telegram-mcp — на Python, поддерживает daily summary / mentions.
- Следуйте `README` репозитория для установки и создания `session/user.session`.

Вариант B — написать минимальный сервер самостоятельно (на Telethon / Pyrogram) со следующим набором инструментов:
- `get_daily_summary()` — собирает сообщения за 24 часа
- `get_mentions(hours: int)` — только упоминания @me
- `get_target_summary(keyword: str, hours: int)` — группы, чьё имя содержит keyword
- `send_message(chat_id, text)` — отправка сообщения

Положите файлы сервера в `~/.claude/mcp-servers/telegram/` (см. структуру в шаге 2).

### Шаг 4. Вставить свои API-ключи

Открыть `~/.claude/mcp-servers/telegram/server.py` и `auth.py`, заменить:

```python
API_ID = YOUR_API_ID        # ← вставить цифры из my.telegram.org
API_HASH = "YOUR_API_HASH"  # ← вставить строку из my.telegram.org
```

### Шаг 5. Создать виртуальное окружение и установить зависимости

```bash
cd ~/.claude/mcp-servers/telegram
python3 -m venv venv
./venv/bin/pip install -r requirements.txt
```

### Шаг 6. Авторизоваться (один раз)

```bash
cd ~/.claude/mcp-servers/telegram
./venv/bin/python3 auth.py
```

Скрипт спросит:
1. Номер телефона (`+79001234567`)
2. Код из Telegram-приложения
3. Пароль 2FA (если включён)

После успешной авторизации создастся файл `session/user.session` — он хранит сессию, не передавать третьим лицам.

### Шаг 7. Добавить в конфиг Claude Desktop

Открыть `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) или аналог:

```json
{
  "mcpServers": {
    "telegram": {
      "command": "/Users/YOUR_USERNAME/.claude/mcp-servers/telegram/venv/bin/python3",
      "args": [
        "/Users/YOUR_USERNAME/.claude/mcp-servers/telegram/server.py"
      ]
    }
  }
}
```

> Заменить `YOUR_USERNAME` на реальное имя пользователя системы.

### Шаг 8. Добавить разрешения в settings.json

Открыть `~/.claude/settings.json`, добавить в `permissions.allow`:

```json
"mcp__telegram__*"
```

### Шаг 9. Перезапустить Claude Code

После перезапуска доступны инструменты:
- `get_daily_summary` — полный дайджест за 24ч
- `get_mentions` — только упоминания (параметр `hours`)
- `get_target_summary` — только целевые группы (параметр `hours`, `keyword`)
- `send_message` — отправить сообщение в любой чат

---

## Безопасность

- Сессионный файл `session/user.session` — это полный доступ к аккаунту. **Не публиковать, не передавать.**
- API_ID и API_HASH — это ключи приложения (не аккаунта). Их утечка позволит создать новое приложение от твоего имени, но **не даёт доступ к сообщениям**. Тем не менее лучше хранить в env-переменных.
- MCP работает локально, данные не покидают твой компьютер.

---

## Частые проблемы

| Проблема | Решение |
|----------|---------|
| `Session file not found` | Запустить `auth.py` заново |
| `FloodWaitError` | Telegram ограничил запросы — подождать N секунд |
| `PhoneNumberInvalidError` | Номер с кодом страны: `+79001234567` |
| `SessionPasswordNeededError` | Включена 2FA — скрипт спросит пароль автоматически |
| Инструменты не появляются в Claude | Проверить путь к `venv/bin/python3` в конфиге |
| Результат слишком большой | Нормально — файл сохраняется, читать через Python grep |
