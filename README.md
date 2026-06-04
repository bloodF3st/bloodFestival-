# BloodFestival

Telegram Bot API бот на Rust (teloxide). Работает как управляющий интерфейс для blood-harvest userbot'а — принимает команды от белого списка администраторов и транслирует их в userbot. Все задачи хранятся в SQLite и переживают перезапуск.

---

## Возможности

### /sp — Спаммер
Запускает рассылку сообщений из `.txt`-шаблона в указанный чат с заданным интервалом.
```
/sp <chat_id> <интервал_мс> <файл.txt> [url] [префикс]
/sp del <id>
```

### /tag — Теггер
Периодически тегает пользователя в чате, отправляя текст из шаблона.
```
/tag <chat_id> <user_id> <интервал_мс> <файл.txt> [url] [префикс]
/tag del <id>
```

### /sa — Автоответчик
Автоматически отвечает конкретному пользователю в чате. Срабатывает на каждое его сообщение с дебаунсом.
```
/sa <chat_id> <user_id> <интервал_мс> <файл.txt> [url] [префикс]
/sa del <id>
```

### /timer — Таймер молчания
Следит за активностью пользователя в чате. При молчании дольше порога — уведомление администратору.
```
/timer <time>                           # reply на сообщение цели (chat_id авто)
/timer <user_id> <chat_id> <time>       # вручную
/timer del <id>
```
Форматы времени: `30m`, `2h`, `1d`, `90s`.

### /list — Список задач
Все активные задачи с их ID. Опционально — фильтр по типу.
```
/list
/list sp
/list tag
/list sa
/list log
```

### /logger — Лог чата
Пересылает сообщения из чата в ЛС администраторам. Можно нацелить на конкретного пользователя.
```
/logger <chat_id>
/logger <chat_id> <user_id>
/logger del <id>
```

### /upl — Загрузка медиа
Загружает прикреплённое фото/видео/GIF/стикер на x0.at и возвращает ссылку.
```
/upl           # прикрепи медиа к команде или ответь на сообщение с медиа
```

### /file — Управление шаблонами
Загрузка, просмотр и удаление `.txt`-шаблонов.
```
/file list
/file del <файл.txt>
# Загрузка: отправь .txt файл, бот сохранит
```

### /pic — Медиа для команд
Устанавливает картинку, которая прикрепляется к ответам `/help` и `/uptime`.
```
/pic help [url]
/pic uptime [url]
/pic id [url]
```

### /sym — Символ бота
Меняет символ-префикс во всех ответах бота (по умолчанию `⛧`).
```
/sym ☽
/sym ⛧
```

### /title — Заголовок
Меняет отображаемое имя бота.
```
/title Новое Имя
```

### /uptime — Статус
Показывает uptime бота, версию, getMe и статистику задач.

### /id — ID
Показывает ID текущего чата и пользователя.

### /help — Справка
Список всех команд с описанием.

---

## Токен-страж (TokenSafe / Renew)

Blood-harvest автоматически следит за токеном blood-festival. Если токен умирает (401 Unauthorized):

1. Через BotFather создаётся новый бот
2. `.env` blood-festival и blood-harvest обновляются с новым токеном и username
3. `systemctl restart blood-festival-bot` — бот перезапускается с новым токеном
4. Новый бот приглашается во все чаты где работал предыдущий
5. В ntfy приходит уведомление

Настраивается через переменные `FESTIVAL_*` в `.env` blood-harvest.

---

## Переменные окружения

| Переменная | Обязательно | Описание |
|---|---|---|
| `BOT_TOKEN` | ✅ | Токен от @BotFather |
| `ADMINS_TGID` | ✅ | ID администраторов через запятую: `123456789,987654321` |
| `TOKEN_DATABASE_URL` | ✅ | SQLite путь: `sqlite:data/data.db` |
| `BF_SESSION_NAME` | — | Имя сессии в БД (по умолчанию `bloodfestival`) |
| `USER_TEMPLATES_DIR` | — | Папка шаблонов (по умолчанию `data/user_templates`) |
| `NTFY_URL` | — | URL канала ntfy для уведомлений |

---

## Установка и запуск

### 1. Скачать бинарник

Скачай актуальный релиз со страницы [Releases](../../releases) — файл `blood-festival-bot`.

### 2. Разместить файлы

```bash
mkdir -p /opt/bloodfestival/data /opt/bloodfestival/data/user_templates
cp blood-festival-bot /opt/bloodfestival/
chmod +x /opt/bloodfestival/blood-festival-bot
```

### 3. Создать `.env`

```bash
cat > /opt/bloodfestival/.env << 'EOF'
BOT_TOKEN=1234567890:AAF...токен от BotFather...
ADMINS_TGID=123456789,987654321
TOKEN_DATABASE_URL=sqlite:data/data.db
BF_SESSION_NAME=bloodfestival
USER_TEMPLATES_DIR=data/user_templates
NTFY_URL=https://ntfy.sh/MyChannel
EOF
```

### 4. Настройка systemd

```bash
cat > /etc/systemd/system/blood-festival-bot.service << 'EOF'
[Unit]
Description=blood festival bot
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/opt/bloodfestival
EnvironmentFile=/opt/bloodfestival/.env
ExecStart=/opt/bloodfestival/blood-festival-bot
Restart=on-failure
RestartSec=10
StartLimitIntervalSec=120
StartLimitBurst=5

MemoryMax=200M
MemoryHigh=150M
CPUQuota=25%
TasksMax=256

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable blood-festival-bot
systemctl start blood-festival-bot
```

### 5. Управление

```bash
systemctl status blood-festival-bot      # статус
systemctl restart blood-festival-bot     # перезапуск
journalctl -u blood-festival-bot -f      # живые логи
journalctl -u blood-festival-bot -n 100  # последние 100 строк
```

---

## Шаблоны `.txt`

Каждая строка файла — отдельное сообщение. При каждой отправке выбирается случайная строка.

Загрузка: отправь `.txt` файл боту — он сохранится автоматически.

Удаление: `/file del имя.txt`

Просмотр: `/file list`

---

## Связка с BloodHarvest

Blood-festival управляет через Bot API, blood-harvest выполняет реальные действия через MTProto. Стандартная схема развёртывания:

```
Telegram Admin → /sp → blood-festival-bot → SQLite → blood-harvest userbot → чат
```

Blood-harvest читает задачи из общей SQLite БД и выполняет отправку от имени пользовательского аккаунта.

---

## Требования к серверу

- Linux x86-64
- glibc 2.17+
- ~20 МБ RAM в idle, до 200 МБ под нагрузкой
- SQLite (встроен в бинарник, отдельно не нужен)

---

## Аллокатор памяти (jemalloc)

Бинарник собран с [jemalloc](https://github.com/jemalloc/jemalloc) вместо стандартного системного аллокатора.

**Зачем:** glibc malloc держит освобождённую память у себя и не возвращает её ОС — RSS процесса со временем только растёт. jemalloc периодически отдаёт неиспользуемую память обратно системе.

Настройка через `[Service]` секцию systemd:

```ini
Environment=MALLOC_CONF=background_thread:true,dirty_decay_ms:10000,muzzy_decay_ms:10000,narenas:2
```

| Параметр | Что делает |
|---|---|
| `background_thread:true` | Запускает фоновый поток-уборщик памяти |
| `dirty_decay_ms:10000` | Через 10 сек неиспользуемая память возвращается ОС |
| `muzzy_decay_ms:10000` | То же для «мягко» освобождённых страниц |
| `narenas:2` | Ограничить число арен — меньше фрагментации |

Без `background_thread:true` уборка происходит только при следующей аллокации — то есть в простаивающем боте не происходит вообще.

**На практике:** бот обработал нагрузку, RSS вырос до 50 МБ → через 10 секунд простоя jemalloc отдаёт лишнее обратно → RSS падает до ~15 МБ. С glibc эти 50 МБ висели бы постоянно.