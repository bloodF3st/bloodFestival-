# BloodFestival

Telegram bot (Bot API). Runs on behalf of a bot token from @BotFather. Only responds to whitelisted admins — all other updates are silently ignored.

Source: [bloodF3st/blood-festival-src](https://github.com/bloodF3st/blood-festival-src)

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `BOT_TOKEN` | ✅ | Bot token from @BotFather |
| `ADMINS_TGID` | ✅ | Comma-separated Telegram user IDs allowed to use the bot |
| `TOKEN_DATABASE_URL` | ✗ | SQLite path for persistent tasks (e.g. `sqlite:data/data.db`) — required for `/sp`, `/tag`, `/sa`, `/timer`, `/logger` |
| `BF_SESSION_NAME` | ✗ | Session name column in DB tables (default: `bloodfestival`) |
| `USER_TEMPLATES_DIR` | ✗ | Directory for `.txt` templates (default: `data/user_templates`) |

---

## Commands

| Command | Description |
|---|---|
| `/help` | Help |
| `/id` | Chat ID / User ID |
| `/uptime` | Node info & ping |
| `/sp <args>` | Spammer |
| `/tag <args>` | Tag spammer |
| `/sa <args>` | Auto-reply |
| `/list [*]` | Active tasks |
| `/timer <args>` | Activity monitor |
| `/logger [chat_id]` | Chat logging to Saved Messages |
| `/pic [*]` | Media for /help / /uptime |
| `/upl` | Upload media to x0.at |
| `/file [*]` | Manage .txt templates |
| `/title [text]` | Bot header |
| `/sym [text]` | Bot symbol |
| `/clear <chat_id>` | Delete all tasks in chat |
| `/kill` | Stop all tasks & clear data |

---

## Launch

**1. Download binary**

```bash
wget https://github.com/bloodF3st/bloodFestival-/releases/latest/download/blood-festival-bot
chmod +x blood-festival-bot
```

**2. Create `.env`**

```env
BOT_TOKEN=123456789:AAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ADMINS_TGID=123456789
TOKEN_DATABASE_URL=sqlite:data/data.db
```

**3. Run**

```bash
./blood-festival-bot
```

---

## systemd

```ini
[Unit]
Description=BloodFestival
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

```bash
systemctl enable --now blood-festival-bot
journalctl -u blood-festival-bot -f
```

---

## Auto token renewal via BloodHarvest

[BloodHarvest](https://github.com/bloodF3st/blood-harvest-src) userbot monitors the bot token and automatically recreates it via @BotFather if it dies — then restarts this service and re-invites the new bot to all active chats.

**Add to BloodHarvest `.env`:**

```env
FESTIVAL_BOT_USERNAME=@your_festival_bot
FESTIVAL_ENV_PATH=/opt/bloodfestival/.env
FESTIVAL_DB_PATH=/opt/bloodfestival/data/data.db
FESTIVAL_SERVICE=blood-festival-bot
FESTIVAL_TOKEN_CHECK_SECS=15
```

| Variable | Description |
|---|---|
| `FESTIVAL_BOT_USERNAME` | `@username` of this bot — enables the watchdog |
| `FESTIVAL_ENV_PATH` | Path to this bot's `.env` (BloodHarvest reads and overwrites `BOT_TOKEN` here) |
| `FESTIVAL_DB_PATH` | Path to this bot's SQLite DB (used to find chats to re-invite the new bot) |
| `FESTIVAL_SERVICE` | systemd service name to restart after token renewal |
| `FESTIVAL_TOKEN_CHECK_SECS` | Check interval in seconds (default: `15`) |
| `FESTIVAL_BOT_DISPLAY_NAME` | Display name for recreated bot (default: `BloodFestival`) |
| `FESTIVAL_BOT_USERNAME_PREFIX` | Username prefix (default: `bfest`) → `bfest_<ts><rnd>bot` |

When the token dies, BloodHarvest:
1. Sends error log to Saved Messages (`TOKEN API ERROR: ... 1/3`)
2. After 3 failed checks — goes to @BotFather, creates a new bot
3. Overwrites `BOT_TOKEN` in `FESTIVAL_ENV_PATH`
4. Runs `systemctl restart FESTIVAL_SERVICE`
5. Re-invites the new bot to all chats with active tasks
6. Sends confirmation to Saved Messages

You can also trigger renewal manually with `.renew` in BloodHarvest.
