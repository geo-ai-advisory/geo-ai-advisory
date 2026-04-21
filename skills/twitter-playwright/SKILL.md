# Twitter/X — публикация через Playwright

## Рабочий метод (подтверждён 26.03.2026)

Playwright UI automation обходит anti-bot защиту Twitter (GraphQL API блокируется error 226, twikit сломан, agent-twitter-client deprecated).

### Публикация поста

1. Открыть compose: `browser_navigate` → `https://x.com/compose/post`
2. Подождать 3-4 сек загрузку
3. Найти `textbox "Post text"` в snapshot
4. `browser_type` с `slowly: true` — текст тизера + URL
5. Подождать 1-2 сек — Twitter подтянет превью ссылки
6. Найти `button "Post"` → `browser_click`
7. Проверить alert "Your post was sent." + ссылку на твит

### Важно

- **slowly: true** обязателен — иначе Twitter может не обработать ввод
- Превью ссылки появляется автоматически (card preview)
- Новый аккаунт может показать "Unlock more on X" (graduated access) — это НЕ блокирует публикацию
- Профиль: `~/.playwright-profile` с залогиненным x.com

### Нерабочие методы (март 2026)

| Метод | Проблема |
|-------|----------|
| twikit 2.3.3 | KEY_BYTE indices (issue #408) + Cloudflare 403 (#396) |
| agent-twitter-client | deprecated, guest token 404 |
| curl v1.1 REST API | error 179, endpoint устарел |
| GraphQL API (перехват) | error 226 "automated request" |
| bird CLI (@steipete/bird) | deprecated, httpOnly cookies не извлечь из Playwright mock keychain |

### Альтернативы

1. **Official Twitter API Free Tier** — $5 одноразово, 500 постов/мес. `console.x.com` → App → OAuth 1.0a → Tweepy
2. **Postiz** (self-hosted) — нужны Twitter API ключи
3. **XActions** (github.com/nirholas/XActions) — Puppeteer, 133 stars
