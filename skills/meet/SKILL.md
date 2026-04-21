---
name: meet
description: Use when user asks to create a meeting (создать встречу, запланировать встречу, meet). Creates Telemost meeting, looks up attendees in Google Sheets, creates Google Calendar event.
---

# Meet — создание встречи

## Критично: защита от дублей

- В рамках одной пользовательской задачи допускается **не более одного** вызова `mcp__gdrive__calendar_create_event`.
- До создания новой встречи **обязательно** проверить `primary` календарь через `mcp__gdrive__calendar_list_events` в окне от `startTime - 2 часа` до `endTime + 2 часа`.
- Если найдено событие с тем же `summary` и тем же временем начала (допуск до 5 минут), считать встречу уже созданной: новое событие и новый Телемост не создавать.
- Если после создания нужна проверка, использовать `mcp__gdrive__calendar_get_event` по `eventId`, а не повторный `calendar_create_event`.

## Алгоритм (строго по порядку, всё параллельно где возможно)

### Шаг 0: Нормализация

- Сначала вычисли точные `summary`, `startTime`, `endTime`, часовой пояс и финальный список участников.
- Относительные даты (`сегодня`, `завтра`, `понедельник`) сразу переводи в абсолютную дату.

### Шаг 1: Проверка, что встреча ещё не создана

- Вызови `mcp__gdrive__calendar_list_events` для `calendarId=primary` в окне вокруг встречи.
- Сравни `summary` и время начала найденных событий.
- При совпадении верни уже существующее событие и **заверши задачу без новых create-вызовов**.

### Шаг 2 + 3 параллельно: Телемост и поиск участников

**Телемост** — используй `mcp__telemost__create_meeting`.
Если недоступен — curl:
```
curl -s -X POST https://cloud-api.yandex.net/v1/telemost-api/conferences \
  -H "Authorization: OAuth y0__xCw_J2mqveAAhiTzD4gjLG10hZZ_G31o4pxiLUdDqqGQxUjPFcldw" \
  -H "Content-Type: application/json" -d '{}'
```

**Поиск участников** — если указаны имена (не email):
- Таблица: `1kAjDg4HOTfc1gzvpV2rOK7KwWeYgIxcUvTxp_rdaHak`, sheetId `1920373701`
- Колонка A = Имя, Колонка D = Gmail
- Искать через `mcp__gdrive__gsheets_read`, брать email из колонки D
- Никогда не искать через Gmail

### Шаг 4: Google Календарь

`mcp__gdrive__calendar_create_event`:
- `calendarId`: `primary`
- `summary`: название от пользователя, иначе "Встреча"
- `startTime` / `endTime`: ISO 8601, МСК = `+03:00`, длительность 1 час по умолчанию
- `description`: ссылка Телемост из шага 1
- `attendees`: Gmail из шага 2
- `sendUpdates`: `all`

### Шаг 5: Верификация без повторного создания

- Сохрани `eventId` из ответа `calendar_create_event`.
- Если нужно перепроверить результат, вызови `mcp__gdrive__calendar_get_event` по этому `eventId`.
- Никогда не делай второй `calendar_create_event` в той же задаче.

## Ответ пользователю
- Название, дата/время МСК
- Ссылка Телемост
- Кто приглашён (имя + email)
