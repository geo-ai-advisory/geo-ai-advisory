# Навык: Substack — загрузка обложки и управление постом через API

## Когда применять
При работе с Substack через Playwright: загрузка cover image, обновление метаданных поста, публикация.

## Проблема: Playwright setFiles() не работает для cover upload

Substack использует React — `setFiles()` устанавливает `input.files`, но НЕ триггерит React synthetic `onChange`. Результат: спиннер "Loading..." появляется, но никакой API-запрос не уходит, обложка не загружается.

## Решение: загрузка через Substack API напрямую

### Шаг 1: Подготовить base64 файла

```bash
python3 -c "
import base64
with open('/path/to/cover.png', 'rb') as f:
    data = base64.b64encode(f.read()).decode('ascii')
with open('/tmp/cover_b64.js', 'w') as out:
    out.write(f'window.__coverB64__ = \"{data}\";')
print('Written', len(data), 'chars')
"
```

### Шаг 2: Передать данные в браузер через addScriptTag

```javascript
await page.addScriptTag({ path: '/tmp/cover_b64.js' });
```

Важно: прямой передачей через evaluate не получится если файл > 50KB — используй addScriptTag с path.

### Шаг 3: Загрузить изображение на Substack

```javascript
const uploadResult = await page.evaluate(async () => {
  const b64 = window.__coverB64__;
  const dataUrl = 'data:image/png;base64,' + b64;

  const response = await fetch('/api/v1/image', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      image: dataUrl,
      postId: POST_ID  // числовой ID поста
    }),
    credentials: 'include'
  });

  return await response.json();
  // Ответ: { id, url, contentType, bytes, imageWidth, imageHeight }
});

const imageUrl = uploadResult.url;
```

### Шаг 4: Установить cover_image на пост

```javascript
const result = await page.evaluate(async (url) => {
  const response = await fetch('/api/v1/drafts/POST_ID', {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ cover_image: url }),
    credentials: 'include'
  });
  return await response.json();
  // Ответ содержит все поля поста включая cover_image
}, imageUrl);
```

PUT `/api/v1/drafts/{id}` мгновенно обновляет и черновик, и опубликованный пост одновременно.

### Шаг 5: Нажать "Update now" через UI

После установки cover_image через API нужно переопубликовать пост чтобы изменения применились к читателям. Continue кнопка может быть disabled если открыт File Settings сайдбар — принудительно кликни:

```javascript
const continueBtn = Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === 'Continue');
continueBtn.disabled = false;
continueBtn.click();
// Откроется диалог Publish с кнопкой "Update now"
```

## Другие полезные API

| Эндпоинт | Метод | Назначение |
|----------|-------|------------|
| `/api/v1/drafts/{id}` | GET | Получить данные черновика/поста |
| `/api/v1/drafts/{id}` | PUT | Обновить любые поля поста |
| `/api/v1/image` | POST | Загрузить изображение, получить URL |
| `/api/v1/image` | POST | Поддерживает: image (data URL), postId |

## Canonical URL
В стандартном редакторе Substack НЕТ поля canonical URL. Только SEO title, SEO description, post URL slug.

## Почему это работает
Substack хранит `cover_image` как поле в черновике/посте. PUT на `/api/v1/drafts` — это основной способ сохранения данных поста (тот же эндпоинт используется при автосохранении редактора). Изменение cover_image через этот API немедленно отражается на публичной странице поста (og:image).
