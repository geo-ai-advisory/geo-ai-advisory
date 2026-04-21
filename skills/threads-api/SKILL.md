# Threads API — создание приложения и публикация

## Создание Meta App (обход Business Portfolio)

**Проблема:** Новый визард с use case "Threads API" требует Business Portfolio с Full Control на шаге "Компания". Даже при наличии Full Control в портфолио, developer portal может показывать "Нет доступных компаний".

**Решение — старый визард:**
1. Открыть `https://developers.facebook.com/apps/creation/`
2. Шаг "Сценарии использования" → фильтр **"Другое (5)"** → выбрать **"Другое"** (внизу, "This option is going away soon")
3. Перенаправит на старый визард с 2 шагами: **Тип → Информация**
4. Тип: выбрать **"Потребительское"** (Consumer)
5. Информация: название, email. **Бизнес-портфолио = "Необязательно"!**
6. Нажать "Создание приложения" → ввести пароль Facebook

## Настройка Threads API в приложении

После создания приложения:
1. Dashboard → **Settings → Basic** → записать App ID и App Secret
2. Добавить **Threads API** как продукт (если не добавлен автоматически)
3. Settings → добавить **OAuth Redirect URI** (можно `https://oauth.pstmn.io/v1/callback`)
4. **App Roles → Roles → Add People → Threads Tester** → ввести Threads username
5. В Threads → Settings → Website Permissions → Invites → **принять приглашение**

## Получение токена (OAuth flow)

1. Открыть в браузере:
```
https://threads.net/oauth/authorize?client_id={APP_ID}&redirect_uri={REDIRECT_URI}&scope=threads_basic,threads_content_publish&response_type=code
```
2. Авторизоваться → получить `code` из redirect URL
3. Обменять code на short-lived token:
```bash
curl -X POST https://graph.threads.net/oauth/access_token \
  -d "client_id={APP_ID}" \
  -d "client_secret={APP_SECRET}" \
  -d "grant_type=authorization_code" \
  -d "redirect_uri={REDIRECT_URI}" \
  -d "code={CODE}"
```
4. Обменять на long-lived token (60 дней):
```bash
curl "https://graph.threads.net/access_token?grant_type=th_exchange_token&client_secret={APP_SECRET}&access_token={SHORT_LIVED_TOKEN}"
```

## Публикация через threads-mcp-server

```bash
# Установка
npm install -g threads-mcp-server

# Бинарник = threads-mcp (НЕ threads-mcp-server)
THREADS_ACCESS_TOKEN="{TOKEN}" threads-mcp

# Инструмент: publish_thread
# 40+ инструментов включая аналитику
```

## Альтернативы (без Meta App)

- **Late.dev** — бесплатно 20 постов/мес, $13/мес за 120 постов. Один POST запрос.
- **Playwright automation** — хрупкий, но не нужен токен
- **PyThreads** (`pip install pythreads`) — Python-обёртка над официальным API

## Важно

- В **development mode** можно публиковать на свой аккаунт без App Review
- Short-lived token живёт 1 час, long-lived — 60 дней
- Long-lived token можно обновлять до истечения
- "Другое" в визарде помечено как "soon deprecated" — может исчезнуть
