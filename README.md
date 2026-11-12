# KHAN - TENGRI — Backend

Backend сайта тур-агентства на Node.js, Express и MongoDB. Включает бронирование туров, SMS-подтверждение, уведомления менеджеру и админку для туров, гидов и бронирований.

## Что внутри

- `POST /api/bookings` — приём брони (имя, телефон, тур, число человек, сообщение)
- OTP-подтверждение номера по SMS (выключаемое, `REQUIRE_OTP` в `.env`)
- Email-уведомление менеджеру при новой брони (Resend, бесплатно)
- `GET /api/tours`, `GET /api/guides` — публичные списки для сайта
- Админка на `/admin.html`: вход по паролю, вкладки «Брони», «Туры», «Команда» — добавление, редактирование, удаление, скидки на туры со сроком действия. Для команды доступны фото, краткая и полная биография, роль, стаж, языки и направления.
- SMS клиенту при нажатии «Одобрить» (выключаемое, `SMS_ON_APPROVE`) + кнопка открыть WhatsApp с готовым текстом

## Быстрый старт

```bash
npm install
```

Создай локальный `.env` на основе `.env.example`, укажи новый `MONGODB_URI` и задай `ADMIN_PASSWORD`. Для начала можно оставить SMS выключенным (`SMS_PROVIDER=console`, `REQUIRE_OTP=false`).

```bash
npm run dev
```

Проверка: `http://localhost:5000/health` → `{"ok":true,...}`.

Админка: `http://localhost:5000/admin.html` → вход по `ADMIN_PASSWORD` → вкладка «Туры» → «Загрузить примеры» (появится 6 туров по Кыргызстану), то же на вкладке «Команда». Фотографии сотрудников сжимаются в браузере (до 350 КБ) и хранятся в MongoDB вместе с профилем. Адрес `/admin.htm` также перенаправляется на `/admin.html`.

## Автоперевод в админке

В формах туров и команды есть кнопка «Перевести на английский»: она заполняет EN-поля, а перед сохранением текст можно проверить и изменить. Для работы добавьте `OPENAI_API_KEY` в `.env` бэкенда и перезапустите сервер. При необходимости модель задаётся переменной `OPENAI_TRANSLATION_MODEL` (по умолчанию `gpt-4o-mini`). Ключ остаётся только на сервере и не передаётся в браузер.

## Новое подключение и деплой

1. Создай отдельную базу MongoDB и добавь её URI в `MONGODB_URI`. Не используй URI от другого проекта.
2. Запушь эту копию проекта в новый GitHub-репозиторий. Не добавляй `.env` и не удаляй `.gitignore`.
3. Создай новый Web Service в Render из этого репозитория, выбрав Blueprint из `render.yaml` (или укажи Root Directory `tour-agency-backend`, Build Command `npm ci`, Start Command `npm start`).
4. В Render задай `MONGODB_URI`, `ADMIN_PASSWORD` и `CLIENT_ORIGIN`. В `CLIENT_ORIGIN` укажи точный адрес нового сайта; можно перечислить несколько origin через запятую.
5. Создай новый Vercel-проект из того же репозитория с Root Directory `tour-agency-frontend`. В переменной `VITE_API_URL` укажи URL нового backend в Render и запусти новый deployment.
6. Не копируй значения из старых `.env` или настроек старых проектов. Секреты вводятся только в локальный `.env` или в панели новых сервисов.

`CLIENT_ORIGIN` не имеет production-значения по умолчанию: добавь URL нового frontend, чтобы разрешить ему запросы к API. Завершающий `/` допустим. Frontend удаляет завершающие слеши из `VITE_API_URL`.

## Структура

```
tour-agency-backend/
├── src/
│   ├── server.js
│   ├── config/db.js
│   ├── models/{Tour,Guide,Booking}.js
│   ├── controllers/{catalogController,bookingController,uploadController}.js
│   ├── routes/{catalog,bookings}.js
│   ├── services/{smsService,emailService}.js
│   ├── middleware/{adminAuth,validate,errorHandler}.js
│   └── seed/defaults.js
└── public/admin.html
```
