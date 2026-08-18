# 🔒 Google SERP Parser — Приватный технический README

> [!CAUTION]
> Этот файл **только для разработчика**. Не публиковать на GitHub и не включать в релиз.
> Хранится в `.agents/`, который исключён из git через `.gitignore`.

**Версия: 1.8.11**

---

## 📂 Структура проекта

```
GoogleParserChrome/
├── manifest.json              # Манифест расширения Manifest V3
├── background.js              # Service Worker: очередь, потоки (1-5), хранилище
├── content.js                 # Content Script для парсинга DOM Google
├── serp-parser.js             # Универсальный модуль извлечения данных из HTML
├── ui.html                    # Разметка интерфейса
├── ui.css                     # Стили Glassmorphic Dark Mode
├── ui.js                      # Движок UI, SERP Diff, снимки, экспорт
├── license.js                 # JS-модуль LicenseManager (загрузка Wasm, проверка ключа)
├── lib/
│   ├── xlsx.full.min.js       # SheetJS для .xlsx
│   └── wasm_exec.js           # Среда выполнения Go Wasm
├── wasm/
│   ├── main.go                # Go-верификатор Ed25519 (исходник, в .gitignore)
│   └── main.wasm              # Скомпилированный Wasm-модуль (в релизе)
├── icons/                     # Иконки (16-128px, SVG)
├── tools/                     # Go-утилиты (исключены из релиза и git)
│   ├── build_release.go       # Сборка zip-архива
│   ├── publish_release.go     # Публикация на GitHub Releases
│   ├── cleanup_releases.go    # Массовое удаление релизов и тегов
│   └── keygen.go              # CLI-генератор ключей Ed25519
├── go.mod                     # Go-модуль serpparser (исключён из git)
├── .gitignore                 # Исключения: .agents/, tools/, go.mod, wasm/main.go, dist/, *.zip
├── .agents/
│   ├── AGENTS.md              # Правила для ИИ-агентов
│   └── README_PRIVATE.md      # Этот файл
└── README.md                  # Публичный README для GitHub (продающий)
```

---

## 🔑 Система лицензирования Ed25519 + Go Wasm

### Цепочка лицензирования
1. `tools/keygen.go` — CLI-инструмент для генерации ключей и лицензий (только для разработчика)
2. `wasm/main.go` — Go-верификатор подписи Ed25519, компилируется в `wasm/main.wasm`
3. `license.js` — JS-модуль `LicenseManager`: загрузка Wasm, проверка ключа, хранилище
4. `manifest.json` — `license.js` в `web_accessible_resources`, CSP с `wasm-unsafe-eval`

### Быстрые команды keygen

```bash
# Сгенерировать пару ключей (один раз!)
go run tools/keygen.go -genkeys

# Выдать лицензию пользователю
go run tools/keygen.go -privkey "<PRIVATE_KEY_HEX>" -plan 1m -user "user@example.com"

# Пересобрать Wasm после изменений в wasm/main.go
GOOS=js GOARCH=wasm go build -o wasm/main.wasm ./wasm/
```

### Маппинг планов
| Фраза | Флаг `-plan` | Срок |
|---|---|---|
| 5 минут / тест | `5m` | 5 мин |
| 1 час | `1h` | 1 час |
| месяц / 30 дней | `1m` | 1 мес |
| 3 месяца / квартал | `3m` | 3 мес |
| год / 12 месяцев | `12m` | 12 мес |
| N дней (произвольно) | `<N*24>h` | N дней |

### Маппинг кодов в читаемые названия (`plan_name`)
- `1m` → "1 месяц"
- `3m` → "3 месяца"
- `12m` → "12 месяцев"
- и т.д.

### Мастер-ключ для разработки
- `month_license_2026` — обходит криптопроверку, используется в `license.js`

### Выданные лицензии
| Дата | Пользователь | План | Истекает | Ключ |
|---|---|---|---|---|
| 2026-08-16 | user_123 | 1 месяц (1m) | 2026-09-15 | `eyJwYXlsb2Fk...` |
| 2026-08-16 | user@example.com | 5 дней (120h) | 2026-08-21 | — |

### Правило автогенерации лицензий (Antigravity Rule)
Файл `/Users/kirill/.gemini/config/rules/license_keygen.md` — при запросах на естественном языке ИИ автоматически:
1. Распознаёт период ("на месяц", "на год", "на 100 дней")
2. Маппит в соответствующий флаг `-plan`
3. Запускает `go run tools/keygen.go`
4. Выводит готовый ключ активации

---

## 🔧 Сборка и публикация релиза

### Требования
- Go 1.21+
- GitHub CLI (`gh`) и `gh auth login`

### Шаги
```bash
# 1. Изменить версию в manifest.json
# 2. Записать изменения в раздел Changelog в README_PRIVATE.md и README.md
go run tools/build_release.go    # Сборка zip в dist/
go run tools/publish_release.go  # Публикация на GitHub Releases
```

### Исключения из релиза (build_release.go)
- `.go` исходники (`wasm/main.go`)
- `tools/`, `go.mod`, `.agents/`

### Исключения из репозитория (.gitignore)
- `.agents/`, `tools/`, `go.mod`
- `wasm/main.go` (только скомпилированный `wasm/main.wasm`)
- `dist/`, `.DS_Store`, `*.zip`

---

## 🧹 Очистка истории Git и релизов

> [!WARNING]
> Операция необратима — вся история коммитов будет удалена.

```bash
# Шаг 1 — Удаление всех релизов и тегов на GitHub
go run tools/cleanup_releases.go

# Шаг 2 — Очистка истории Git (создание сиротской ветки main)
git checkout --orphan temp_branch
git add -A
git commit -m "Initial commit"
git branch -D main
git branch -m temp_branch main
git push -f origin main
```

---

## 📝 Журнал изменений (полный технический Changelog)

### Версия 1.8.11 (Разделение README на публичный и приватный)
- [x] **Два README**: Создан приватный технический README (`.agents/README_PRIVATE.md`) со всей внутренней инфой (ключи, сборка, git-очистка, маппинги).
- [x] **Публичный README**: Очищен от технической информации, ключей, лицензий, сборки/публикации.

### Версия 1.8.10 (Правило очистки релизов)
- [x] **Правило AGENTS.md**: Расширено правило №6 — очистка включает удаление всех GitHub-релизов и тегов.
- [x] **Go-утилита**: Создан `tools/cleanup_releases.go` — массовое удаление релизов и тегов.
- [x] **Документация**: README сфокусирован на продажах и примерах.
- [x] **Очистка**: убрана техническая кухня.

### Версия 1.8.9 (Продающий README)
- [x] README переписан: продающий фокус на подписку 500₽/мес, функции, установка, примеры.
- [x] Бейдж версии в ui.html обновлён (раньше был `v1.0.0`).

### Версия 1.8.8 (Очистка репозитория)
- [x] `.gitignore` исключён из Git-репозитория.

### Версия 1.8.7 (Очистка папок разработки)
- [x] Папки `.agents/`, `tools/`, `go.mod` исключены из Git.

### Версия 1.8.6 (Фильтрация релиза)
- [x] `build_release.go` исключает `.go` файлы из zip.

### Версия 2.0.0 (Система лицензирования Ed25519 + Go Wasm)
- [x] `tools/keygen.go` — CLI-генератор Ed25519 ключей и лицензий
- [x] `wasm/main.go` — Go-верификатор, скомпилирован в `wasm/main.wasm`
- [x] `license.js` — JS-модуль `LicenseManager`
- [x] Маппинг кодов планов в читаемые названия (`plan_name`)
- [x] `manifest.json` обновлён: `license.js` в `web_accessible_resources`
- [x] `go.mod` создан для сборки модуля (`serpparser`)

### Версия 1.8.5 (Автоматизация GitHub-релизов)
- [x] `tools/build_release.go` и `tools/publish_release.go`
- [x] `.agents/AGENTS.md` с правилами версионирования

### Версия 1.8.1–1.8.4 (Лицензирование: модалка, статус, фикс)
- [x] Модальное окно активации лицензии, плашка статуса
- [x] Уведомление «Спасибо за покупку!» при активации
- [x] Фикс гонки Wasm при быстром вводе ключа
- [x] Сгенерирован ключ на 1 месяц

### Версия 1.8.0 (Типы выдачи: веб/картинки/видео)
- [x] Селектор типа выдачи, многоуровневые парсеры DOM, lazy-load, интеграция с Service Worker

### Версия 1.7.0 (Быстрые действия)
- [x] Скачать/копировать ссылки и сниппеты, Clipboard API с fallback

### Версия 1.6.0 (SERP Diff & Snapshots)
- [x] Режим сравнения, KPI-карточки, фильтры, система снимков, экспорт сравнения

### Версия 1.5.0 (Временные диапазоны)
- [x] Параметр `tbs` (час/24ч/неделя/месяц/год)

### Версия 1.4.0 (Модалки и Toast)
- [x] Кастомное модальное окно подтверждения, Toast-уведомления

### Версия 1.3.0 (Парсинг через вкладки)
- [x] Tab-based scraping, настройка переноса фокуса

### Версия 1.2.0 (Кнопка перезагрузки)
- [x] `chrome.runtime.reload()`

### Версия 1.1.0 (Полноэкранный режим)
- [x] Открытие в отдельной вкладке

### Версия 1.0.0 (Начальный релиз)
- [x] Парсинг, Excel/CSV/JSON, Glassmorphism Dark Mode UI
