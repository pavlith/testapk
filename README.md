# TestAPK — Svelte 5 + shadcn-svelte + Capacitor

Тестовое мобильное Android-приложение: задачи, поиск, фильтры, статистика, светлая/тёмная тема и нативная вибрация. Данные сохраняются локально, сервер не нужен.

## Скачать APK

1. Открой [GitHub Actions → Build Android APK](https://github.com/pavlith/testapk/actions/workflows/android-apk.yml).
2. Открой **последний успешный** запуск на `main`.
3. В разделе **Artifacts** скачай `testapk-debug-apk` (ZIP).
4. Извлеки `app-debug.apk` и передай его на Android-телефон.
5. Открой APK на телефоне и разреши установку из этого источника, если Android запросит подтверждение.

Это **debug APK**: он уже подписывается отладочным ключом Android, поэтому устанавливается вручную, но не предназначен для публикации в Google Play. Не вводи в тестовую версию чувствительные данные.

Если workflow в первый раз ещё не запустился, открой вкладку Actions и посмотри его статус. Также доступен ручной запуск через **Run workflow**.

## Локальный запуск

Требуется Node.js >=22. Для сборки на ПК также нужны JDK 21 и Android SDK.

```bash
git clone https://github.com/pavlith/testapk.git
cd testapk
npm install
npm run dev
```

Открыть веб-версию: `http://localhost:5173`.

## Локальная сборка Android

```bash
npm run build
npx cap add android     # выполнить один раз
npx cap sync android
cd android
./gradlew assembleDebug
```

Результат: `android/app/build/outputs/apk/debug/app-debug.apk`.

Папка `android/` генерируется из конфигурации Capacitor и не хранится в этом репозитории. CI делает то же самое перед каждой сборкой.

## Стек

- Svelte 5 (runes), TypeScript, Vite 7
- Tailwind CSS 4, shadcn-svelte-style локальные компоненты в `src/lib/components/ui`
- Capacitor 8: Android WebView, Device, Haptics
- GitHub Actions: автоматически собирает APK при push в `main`

Добавлять дополнительные готовые компоненты shadcn-svelte можно командой:

```bash
npx shadcn-svelte@latest add dialog
```
