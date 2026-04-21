# VC.ru — публикация через Osnova REST API

## Аутентификация

Токен: значение `auth-refresh-token` из localStorage на vc.ru
Заголовок: `X-Device-Token: {token}`

Получение через Playwright:
```js
localStorage.getItem('auth-refresh-token')
// → {"token":"...","expTimestamp":...}
```

## Профиль

```bash
curl -s "https://api.vc.ru/v2.1/subsite/me" -H "X-Device-Token: $TOKEN"
```

## Создание поста

```bash
curl -s -X POST "https://api.vc.ru/v2.1/editor" \
  -H "X-Device-Token: $TOKEN" \
  -F "entry=$JSON"
```

### Формат JSON

```json
{
  "type": 1,
  "user_id": 1781320,
  "subsite_id": 1781320,
  "title": "Заголовок статьи",
  "entry": {
    "blocks": [
      {"type": "text", "data": {"text": "<p>Параграф</p>"}},
      {"type": "text", "data": {"text": "<h2>Заголовок секции</h2>"}},
      {"type": "text", "data": {"text": "<ul><li>Пункт 1</li><li>Пункт 2</li></ul>"}}
    ]
  },
  "is_published": false
}
```

- `is_published: false` → черновик, `true` → публикация

## КРИТИЧНЫЕ БАГИ API

1. **Блок `header`** → 500 "технические работы". Заголовки ТОЛЬКО через `<h2>` в text-блоках
2. **Блок `list`** → ошибка валидации. Списки через `<ul><li>` или `<ol><li>` в text-блоках
3. **Rate limiting** → 500 при частых запросах. Пауза ~5-10 сек между POST запросами
4. **Блок `table`** → не проверен. Безопаснее через HTML `<table>` в text-блоке

## Рабочие блоки

Единственный надёжный тип: `"type": "text"` с HTML внутри `data.text`:
- `<p>` — параграф
- `<h2>` — заголовок
- `<ul><li>` / `<ol><li>` — списки
- `<b>` — жирный
- `<i>` — курсив
- `<a href="...">` — ссылка

## Эндпоинты

| Метод | URL | Назначение |
|-------|-----|------------|
| GET | /v2.1/subsite/me | Профиль |
| POST | /v2.1/editor | Создание/публикация |
| GET | /v2.8/timeline | Лента |
| GET | /v2.8/search | Поиск |
