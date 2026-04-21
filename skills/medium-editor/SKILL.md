# Навык: Medium Editor — вставка кликабельной ссылки через Playwright

## Когда применять
Нужно добавить или исправить ссылку в статье на Medium через Playwright (редактор contenteditable).

## Как делать

### 1. Открыть редактор
```js
await page.goto('https://medium.com/p/POST_ID/edit');
```

### 2. Поставить курсор в нужное место через JS
```js
await page.evaluate(() => {
  const paragraphs = document.querySelectorAll('p');
  let targetP = null;
  for (const p of paragraphs) {
    if (p.innerText.includes('Нужный текст')) { targetP = p; break; }
  }
  const range = document.createRange();
  range.selectNodeContents(targetP);
  range.collapse(false); // false = в конец
  const sel = window.getSelection();
  sel.removeAllRanges();
  sel.addRange(range);
  document.querySelector('[contenteditable="true"]').focus();
});
```

### 3. Вставить URL через execCommand
```js
await page.evaluate(() => {
  document.execCommand('insertText', false, 'https://example.com');
});
```

### 4. Нажать пробел — Medium автолинкует URL
```js
await page.keyboard.press('Space');
// После этого Medium создаёт <a href="...">
```

### 5. Убрать лишний пробел
```js
await page.keyboard.press('Backspace');
```

### 6. Проверить что ссылка создалась
```js
const result = await page.evaluate(() => {
  const paragraphs = document.querySelectorAll('p');
  for (const p of paragraphs) {
    if (p.innerText.includes('Нужный текст')) {
      return { html: p.innerHTML, hasLink: !!p.querySelector('a') };
    }
  }
});
// hasLink должно быть true
```

### 7. Сохранить
```js
await page.getByRole('button', { name: 'Save and publish' }).click();
// Ожидать alert "Your changes have been published."
```

## Почему это работает
- `document.execCommand('insertText')` — единственный способ вставить текст в contenteditable который Medium editor воспринимает корректно и записывает в undo history
- Medium автоматически превращает plain URL в `<a>` тег при нажатии пробела после URL
- JS `window.getSelection().addRange()` НЕ триггерит Ctrl+K в Medium — редактор не признаёт программное выделение

## КРИТИЧЕСКИ ВАЖНО: что нельзя делать
- **НЕ использовать `browser_type` с ref редактора** — это вызывает Playwright `.fill()`, который СТИРАЕТ весь контент contenteditable без возможности undo
- **НЕ использовать `.fill()` напрямую** — то же самое
- **Ctrl+K после JS-выделения не работает** — Medium не реагирует на программное выделение

## Признаки успеха
- В снапшоте редактора: `link "https://example.com" [cursor=pointer]`
- В HTML параграфа: `<a href="https://medium.com/r/?url=...">https://example.com</a>`
- После публикации: alert "Your changes have been published."
