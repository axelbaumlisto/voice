---
title: Разработка
layout: default
---

# Разработка

Voxis — это приложение на базе Tauri v2. Бэкенд написан на Rust, а фронтенд — на React 18, TypeScript и Vite.

## Архитектура

### Фронтенд (`src/`)

- **Маршрутизация**: React Router обрабатывает страницы Настроек, Истории, Словаря и Онбординга.
- **Хуки**: `src/hooks/` содержит пользовательские хуки для асинхронных данных, состояния записи и настроек.
- **Команды**: `src/lib/commands.ts` предоставляет типизированные обертки для всех вызовов Tauri `invoke`.
- **Компоненты**: Организованы по доменам (`src/components/dictionary`, `src/components/history` и т.д.).
- **Движок тем**: `src/theme-engine/` содержит ThemeHost, контракт, рендереры и исходники встроенных тем.

### Бэкенд (`src-tauri/`)

- Два бинарных файла: `voice` (основное приложение) и `typing_bench` (бенчмарк задержки автопечати).
- Модули:
  - `orchestrator/` - координация рабочих процессов (горячая клавиша -> запись -> транскрипция -> вывод).
  - `audio/` - запись звука через cpal и VAD.
  - `transcription/` - HTTP-клиент, совместимый с Whisper.
  - `output/` - буфер обмена и автопечать.
  - `storage/` - хранилище на базе SQLite (конфигурация, история, поставщики и т.д.).
  - `theme_engine/` - загрузчик скриптов тем (Rust ничего не знает о визуальном отображении).
  - `commands/` - команды Tauri, доступные фронтенду.

## Поток данных

1. Пользователь нажимает горячую клавишу -> `HotkeyListener` обнаруживает нажатие.
2. `Orchestrator::on_hotkey_pressed()` запускает `AudioRecorder`.
3. Пользователь отпускает клавишу -> звук добавляется в очередь `TranscriptionQueue`.
4. Воркер очереди обрабатывает: транскрипция -> применение словаря -> опциональный LLM -> вывод.
5. Фронтенд получает обновления состояния через события Tauri (`state-changed`, `error`).

## Команды из `package.json`

```bash
bun install
bun run dev                 # Vite frontend dev server
bun run harness             # Vite server for /harness.html
bun run build               # build themes, type-check, Vite build
bun run build:themes        # bundle builtin themes to src-tauri/themes/
bun run tauri dev           # Tauri dev app
bun run tauri build         # production Tauri build
bun run test:run            # Vitest once
bun run test:coverage       # Vitest coverage
bun run test:e2e            # Playwright tests
bun run test:all            # Vitest + Playwright
bun run test:rust           # cargo build examples + cargo test
bun run lint                # ESLint over src/**/*.ts(x)
```

Тесты Rust также можно запускать напрямую:

```bash
cd src-tauri && cargo test
```

## GitHub Pages docs

Сайт документации лежит в `docs-site/` и собирается workflow `.github/workflows/pages.yml` при push в `main`, затрагивающем `docs-site/**` или сам workflow. Используются `actions/configure-pages`, `actions/jekyll-build-pages` с source `docs-site`, `actions/upload-pages-artifact` и `actions/deploy-pages`.

Не добавляйте hosted URL или скриншоты, которых нет. В публичных docs не должно быть credentials и содержимого локальных БД.
