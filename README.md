# geo-ai-advisory

Публичный набор скиллов для Claude Code / Codex и лендинг AI-advisory практики.

- Лендинг: [geo-ai-advisory.github.io/geo-ai-advisory](https://geo-ai-advisory.github.io/geo-ai-advisory/)
- Автор / контакт: [@danielspe](https://t.me/danielspe)

Репозиторий собран из универсальных скиллов автоматизации на базе Claude Code. Внутренние, компания-специфичные скиллы вынесены в отдельные приватные репозитории и сюда не попадают.

---

## Установка

Скиллы работают в Claude Code / Codex через папку `~/.claude/skills/<skill-name>/SKILL.md`.

### Один скилл
```bash
mkdir -p ~/.claude/skills/<skill-name>
curl -o ~/.claude/skills/<skill-name>/SKILL.md \
  https://raw.githubusercontent.com/geo-ai-advisory/geo-ai-advisory/main/skills/<skill-name>/SKILL.md
```

### Все скиллы разом
```bash
git clone https://github.com/geo-ai-advisory/geo-ai-advisory.git /tmp/geo-ai-advisory
cp -r /tmp/geo-ai-advisory/skills/* ~/.claude/skills/
```

После установки перезапусти Claude Code — скиллы появятся в списке и будут активироваться по своим триггерам / slash-командам.

### Самообучающаяся система Claude (CLAUDE.md)
Шаблоны глобального и проектного `CLAUDE.md` лежат в `guides/claude-learning-system/`. Поставить всё автоматически помогает скилл [`setup`](skills/setup/SKILL.md) — вызовом `/setup`.

---

## Скиллы

### Яндекс Трекер
| Скилл | Команда | Что делает |
|---|---|---|
| [tracker](skills/tracker/SKILL.md) | `/tracker` | Интерактивный помощник по Трекеру: задачи, worklog-и, статистика, переходы |
| [tracker_add_task](skills/tracker_add_task/SKILL.md) | `/tracker_add_task` | Интерактивное создание задачи с резолвом ассайни и дедлайном |
| [tracker_report_active](skills/tracker_report_active/SKILL.md) | `/tracker_report_active` | Отчёт по задачам и часам сотрудника за период |
| [tracker-tips](skills/tracker-tips/SKILL.md) | — | Резолв аккаунтов Трекера и тонкости MCP |

### GitLab
| Скилл | Команда | Что делает |
|---|---|---|
| [gitlab_dev_report](skills/gitlab_dev_report/SKILL.md) | `/gitlab_dev_report` | Продуктивность разработчика: коммиты, строки, MR, acceptance rate |
| [gitlab_compar](skills/gitlab_compar/SKILL.md) | `/gitlab_compar` | Сравнение работы разработчиков за период (HTML с табами) |
| [gitlab_fulltime_report](skills/gitlab_fulltime_report/SKILL.md) | `/gitlab_fulltime_report` | Полный отчёт по разработчику за всё время + quality |
| [gitlab-find-dev-repos](skills/gitlab-find-dev-repos/SKILL.md) | — | Поиск всех репозиториев разработчика в GitLab |

### HTML и публикация
| Скилл | Команда | Что делает |
|---|---|---|
| [html](skills/html/SKILL.md) | `/html` | Дизайн и вёрстка сильного standalone HTML-артефакта |
| [html-push](skills/html-push/SKILL.md) | `/html-push` | Публикация HTML на GitHub Pages с публичной ссылкой |

### Встречи и календарь
| Скилл | Команда | Что делает |
|---|---|---|
| [meet](skills/meet/SKILL.md) | `/meet` | Telemost + Google Calendar, резолв участников из Sheets |

### Telegram
| Скилл | Команда | Что делает |
|---|---|---|
| [telegram_daily](skills/telegram_daily/SKILL.md) | `/telegram_daily` | Дайджест за 24 часа: mentions, вопросы без ответа, сводка целевых групп |

### Публикация контента
| Скилл | Команда | Что делает |
|---|---|---|
| [twitter-playwright](skills/twitter-playwright/SKILL.md) | — | Постинг в Twitter/X через Playwright (API мертвы) |
| [substack-tips](skills/substack-tips/SKILL.md) | — | Загрузка обложки и управление постом через Substack API |
| [medium-editor](skills/medium-editor/SKILL.md) | — | Medium: вставка кликабельной ссылки через execCommand |
| [vc-ru-api](skills/vc-ru-api/SKILL.md) | — | Публикация на VC.ru через Osnova REST API |
| [threads-api](skills/threads-api/SKILL.md) | — | Threads API: создание приложения и публикация |

### Сервисные
| Скилл | Команда | Что делает |
|---|---|---|
| [setup](skills/setup/SKILL.md) | `/setup` | Установка системы самообучения Claude: глобальный и проектный CLAUDE.md, журналы, режим |
| [create](skills/create/SKILL.md) | `/create` | Создание нового проекта (папка + CONTEXT.md + скилл-команда) |
| [audio_script](skills/audio_script/SKILL.md) | `/audio_script` | Транскрибация звонков (local Whisper) + генерация скрипта AI call-бота |
| [playwright-tips](skills/playwright-tips/SKILL.md) | — | Блокировка JS-редиректов и обход anti-DDoS |

### Поиск кандидатов (hh.ru)
| Скилл | Команда | Что делает |
|---|---|---|
| [hh](skills/hh/SKILL.md) | `/hh` | Интерактивный ассистент поиска кандидатов на hh.ru по раундовой методологии |
| [hh-dm-pm](skills/hh-dm-pm/SKILL.md) | `/hh-dm-pm` | Двойной поиск Delivery Manager + Project Manager с отдельными скорами |
| [hh-resume-search](skills/hh-resume-search/SKILL.md) | — | Раундовая методология поиска резюме через Playwright |

---

## Лицензия

Скиллы распространяются «как есть» для использования в Claude Code / Codex. Вы можете свободно адаптировать их под свои проекты.

---

## Контакты

- Telegram: [@danielspe](https://t.me/danielspe)
- Лендинг: [geo-ai-advisory.github.io/geo-ai-advisory](https://geo-ai-advisory.github.io/geo-ai-advisory/)
