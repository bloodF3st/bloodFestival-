# bloodFestival

Telegram-бот на базе Bot API (token-режим). Только для авторизованных администраторов.

## Команды

| Команда | Описание |
|---|---|
| `/help`, `/start` | Справка |
| `/id` | ID чата и пользователя |
| `/timer <args>` | Таймер слежки |
| `/sp <args>` | Спамер |
| `/tag <args>` | Теггер |
| `/sa <args>` | Автоответчик |
| `/list` | Список активных задач и логгеров |
| `/uptime` | Uptime и getMe |
| `/pic <args>` | Медиа к /help и /uptime |
| `/file <args>` | Шаблоны .txt |
| `/upl <args>` | Загрузка медиа |
| `/logger <args>` | Лог чата в ЛС админам |

## Установка

1. Скачай бинарник из [Releases](../../releases)
2. Создай `.env` на основе `.env.example`
3. Запусти:

```bash
chmod +x blood-festival-bot
./blood-festival-bot
```

Или через systemd / docker — см. ниже.

## Переменные окружения

| Переменная | Обязательно | Описание |
|---|---|---|
| `BOT_TOKEN` | ✅ | Токен бота от @BotFather |
| `ADMINS_TGID` | ✅ | Telegram ID админов через запятую (напр. `123456,789012`) |
| `TOKEN_DATABASE_URL` | ☑️ | PostgreSQL URL для БД бота (без БД задачи не сохраняются между перезапусками) |
| `DATABASE_URL` | ☑️ | Запасной вариант если `TOKEN_DATABASE_URL` не задан |
| `BF_SESSION_NAME` | ❌ | Имя сессии в БД (по умолч. `bloodfestival`) |
| `USER_TEMPLATES_DIR` | ❌ | Папка с шаблонами `.txt` (по умолч. `data/user_templates`) |

> ⚠️ Если БД не задана — задачи (`/timer`, `/sp`, `/tag`, `/sa`) и логгеры не переживают перезапуск.

## Зависимости рантайма

Бинарник скомпилирован статически (rustls, без OpenSSL). Никаких системных зависимостей не требуется.

- **OS**: Linux x86_64 (Ubuntu 20.04+, Debian 11+)
- **glibc**: ≥ 2.31
- **PostgreSQL**: 13+ (опционально, для персистентности задач)

## Стек

| Библиотека | Версия | Назначение |
|---|---|---|
| [teloxide](https://github.com/teloxide/teloxide) | 0.12 | Telegram Bot API фреймворк |
| [tokio](https://tokio.rs) | 1 | Async runtime (multi-thread) |
| [sqlx](https://github.com/launchbadge/sqlx) | 0.8 | PostgreSQL (async, rustls, migrate) |
| [reqwest](https://github.com/seanmonstar/reqwest) | 0.12 | HTTP-клиент (rustls, multipart) |
| [tracing](https://github.com/tokio-rs/tracing) | 0.1 | Структурированные логи |
| [tracing-subscriber](https://github.com/tokio-rs/tracing) | 0.3 | Форматирование логов + env-filter |
| [anyhow](https://github.com/dtolnay/anyhow) | 1 | Обработка ошибок |
| [dotenvy](https://github.com/allan2/dotenvy) | 0.15 | Загрузка `.env` |
| [chrono](https://github.com/chronotope/chrono) | 0.4 | Дата/время |
| [rand](https://github.com/rust-random/rand) | 0.8 | Генерация случайных чисел |
| [once_cell](https://github.com/matklad/once_cell) | 1 | Ленивая инициализация |
| [url](https://github.com/servo/rust-url) | 2.5 | Парсинг URL |

## Запуск через systemd

```ini
[Unit]
Description=bloodFestival Telegram Bot
After=network-online.target

[Service]
Type=simple
WorkingDirectory=/opt/bloodfestival
EnvironmentFile=/opt/bloodfestival/.env
ExecStart=/opt/bloodfestival/blood-festival-bot
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## Сборка

Исходный код закрыт. Бинарник собран с:

```toml
[profile.release]
strip = true
opt-level = 3
lto = true
codegen-units = 1
panic = "abort"
```

