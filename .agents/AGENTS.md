# Правила и инструкции для AI-агентов (AGENTS.md)

1. **Язык общения**: Всегда отвечать исключительно на русском языке.
2. **Ведение README**: Обязательно записывать все внесённые изменения в `README.md` в раздел Журнала изменений (Changelog).
3. **Версионирование проекта**: При любых изменениях в коде или файлах проекта обязательно повышать patch-версию (например, 1.8.5 -> 1.8.6) во всех следующих файлах:
   - `manifest.json` (основной файл версий проекта)
   - `README.md` (в шапке и в разделе Changelog)
   - любых HTML-файлах с отображаемой версией (например, `ui.html`)
   - любых JS-файлах с хардкод-версией (например, `ui.js`)
   - `PRIVACY_POLICY.md` и `privacy_policy.html` (если присутствуют)
4. **Автоматическая сборка релиза**: После любого изменения версии обязателен запуск утилиты сборки релиза на Go:
   ```bash
   go run tools/build_release.go
   ```
5. **Автоматическая публикация релиза на GitHub**: После сборки релиза обязательно загружать собранный `.zip` архив в раздел GitHub Releases с помощью утилиты:
   ```bash
   go run tools/publish_release.go
   ```
   или команды:
   ```bash
   gh release create vX.Y.Z dist/GoogleParser_vX.Y.Z.zip --title "GoogleParser vX.Y.Z" --notes "Релиз версии vX.Y.Z"
   ```
6. **Автоматическая очистка истории коммитов и релизов**: По просьбе пользователя выполнять безопасную очистку:
   - **Шаг 1 — Удаление всех релизов и тегов на GitHub**:
     ```bash
     go run tools/cleanup_releases.go
     ```
   - **Шаг 2 — Очистка истории Git (создание сиротской ветки main)**:
     ```bash
     git checkout --orphan temp_branch
     git add -A
     git commit -m "Initial commit"
     git branch -D main
     git branch -m temp_branch main
     git push -f origin main
     ```
7. **Состав релиза и репозитория**:
   - **Загрузка в Git-репозиторий**: В Git-репозиторий загружаются **только `README.md` и `promo.jpg` (`icons/promo.jpg`)**. Все остальные файлы исключаются из репозитория с помощью `.gitignore`.
   - **Содержимое релиза (build_release.go)**: В zip-архив релиза включаются **только файлы, необходимые для работы приложения** (без `.go` исходников, утилит `tools/` и `go.mod`).


## Основные переменные проекта

```ini
PROJECT_NAME   = GoogleParser     # имя проекта (используется в zip-файлах и названиях релизов)
ZIP_PREFIX     = GoogleParser     # префикс имени zip-архива
TAG_PREFIX     = v                # префикс git-тега (например, "v1.8.5")
MANIFEST_FILE  = manifest.json    # файл, хранящий "version"
DISTRIBUTIVE   = dist             # папка для собранных zip-артефактов
```

## Краткое руководство для ИИ
- Изменения начинаются с повышения `version` в `manifest.json`.
- Записать изменения в `README.md` → раздел **Changelog**.
- Собрать дистрибутив: `go run tools/build_release.go` (создаёт `dist/GoogleParser_vX.Y.Z.zip`).
- Опубликовать релиз: `go run tools/publish_release.go` (запускает `gh release create vX.Y.Z ...`).
- При запросе на очистку истории Git и релизов — выполнить безопасную процедуру (пункт 6): сначала `go run tools/cleanup_releases.go`, затем очистку Git-истории.
