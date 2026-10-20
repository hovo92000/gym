# GYM

Дневник весов в зале. Одна HTML-страница без сборки: режим «Тренировка», карточки упражнений с историей по датам, классическая таблица, журнал по дням, заметки. Работает как приложение на iPhone (добавить на экран «Домой»), офлайн, с облаком Firebase, где у каждого пользователя своя программа.

**Открыть:** https://hovo92000.github.io/gym/

## Как это устроено

| Файл | Что делает |
|---|---|
| `index.html` | всё приложение: стили, логика, стартовые данные |
| `firebase-config.js` | конфиг Firebase; пока `null`, приложение хранит данные только в браузере |
| `firestore.rules` | правила доступа: каждый пользователь видит только `users/{свой uid}/...` |
| `sw.js` | service worker: сайт и Firebase SDK открываются без интернета |
| `manifest.json`, `icon-*.png` | иконка и полноэкранный режим на домашнем экране |

Данные в облаке: `users/{uid}/groups/{группа}` (упражнения и подходы), `users/{uid}/meta/notes`, `users/{uid}/meta/profile`. Запись подхода хранится как `{ v: "27,5", d: "2026-10-20" }`; записи, перенесённые из Apple Notes, без даты, строкой.

## Подключить Firebase (один раз, ~5 минут)

1. Зайди на https://console.firebase.google.com под своим Google-аккаунтом → **Add project** → имя `gym` → Google Analytics можно выключить → **Create project**.
2. **Build → Authentication → Get started → Sign-in method.** Включи три провайдера:
   - **Email/Password** (основной, работает и в приложении на домашнем экране);
   - **Google** (укажи support email);
   - **Anonymous** (вход гостем).
3. Там же вкладка **Settings → Authorized domains → Add domain** → `hovo92000.github.io`.
4. **Build → Firestore Database → Create database** → регион любой (например `europe-west3`) → **Start in production mode** → Create.
5. Вкладка **Rules** → замени содержимое на текст из `firestore.rules` этого репозитория → **Publish**.
6. **Project settings (шестерёнка) → General → Your apps → значок `</>` (Web)** → имя `gym`, Firebase Hosting не нужен → **Register app**. Скопируй объект `firebaseConfig`.
7. Открой `firebase-config.js` в репозитории и вставь конфиг:
   ```js
   window.FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "gym-xxxxx.firebaseapp.com",
     projectId: "gym-xxxxx",
     storageBucket: "gym-xxxxx.firebasestorage.app",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef"
   };
   ```
   Закоммить и запушь. Через минуту на сайте появится кнопка **Войти**.
8. Войди. При первом входе приложение спросит, с чего начать: стандартная программа, пустая, перенос данных с этого телефона или (только для hovo92000@gmail.com) таблицы из Notes.

`apiKey` в конфиге Firebase не секрет: доступ к данным ограничивают правила Firestore, а не ключ. Бесплатный план Spark: 50 000 чтений и 20 000 записей в день, для дневника этого хватает на сотни пользователей.

## Разработка

Исходник собирается из шаблона с двумя адаптерами хранения (Firebase для GitHub Pages и база артефакта claude.ai). Правки вносятся в `index.html` напрямую, если сборщика под рукой нет: это обычный HTML без зависимостей.
