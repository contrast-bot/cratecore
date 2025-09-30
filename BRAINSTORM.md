# CrateCore Discord Bot - Feature Brainstorm v2

<br>

## Overview

CrateCore is a text-based Discord bot simulating a crate/item economy game inspired by CS2/CSGO case opening.
Users earn **Points**, prestige to earn **Gems**, and spend Gems to buy crates, items, and permanent upgrades.
The game is fully command-driven, scalable, and designed for long-term user engagement with competitive leaderboards.

<br>

## Core Concepts

| Concept                | Description                                                                                          |
| ---------------------- | ---------------------------------------------------------------------------------------------------- |
| **Points**             | Temporary currency earned by gameplay (e.g., `/work`, `/daily`). Resets on prestige. Non-tradable.   |
| **Gems**               | Premium currency earned by prestiging, daily rewards, achievements, or events. Persistent. Tradable. |
| **Items**              | Loot obtained from crates. Items have **rarity tiers** affecting drop chance and value.              |
| **Crates**             | Containers bought with Gems, opened to yield items with rarity-weighted RNG.                         |
| **Upgrades**           | Permanent effects (e.g., multiplier, luck) bought with Gems in shop. Non-tradable. Capped.           |
| **Prestige**           | Reset Points (keep 5% as cushion), earn Gems based on Points total, increase prestige count.         |
| **Stats**              | Track lifetime player data (coins earned, crates opened, trades made, prestiges, etc.)               |
| **Inventory**          | Split into three categories: **Items**, **Upgrades**, **Crates**. Hard capacity limits apply.        |
| **Trading/Gifting**    | Only Gems and Items can be traded/gifted. Points and Upgrades cannot.                                |
| **Blacklist**          | Dev-only command to restrict bot access for certain users.                                           |
| **Developer Commands** | Prefix commands limited to developers (admin actions, manual item/currency grants).                  |
| **User Commands**      | Slash commands accessible to all non-blacklisted users.                                              |
| **Events**             | Limited-time crates available in shop. Crates/items persist after event, marked with event tag.      |

<br>

## Currency System

| Currency | Tradable | Reset on Prestige | Use Case                                       |
| -------- | -------- | ----------------- | ---------------------------------------------- |
| Points   | No       | Yes (keeps 5%)    | Earned through gameplay, prestige trigger only |
| Gems     | Yes      | No                | Buy crates, items, upgrades, trade, gift       |

### Gem Sources

To prevent hard paywall and keep engagement high, Gems are obtainable through:

- **Prestige** - Primary source, scales with total Points earned
- **Daily Rewards** - `/daily` command (5-10 Gems/day)
- **Login Streaks** - Consecutive daily claims multiply rewards
- **Achievements** - One-time milestone bonuses (first prestige, 100 crates opened, etc.)
- **Vote Rewards** - (Optional) If listed on bot directories later
- **Event Bonuses** - Special event participation rewards

### Point Sources

- `/work` - Primary grind command (5 min cooldown)
- `/daily` - Bonus Points once per 24h (larger than work, no cooldown conflicts)
- **Random Encounters** - 10% chance during `/work` to trigger mini-event (bonus points, rare gem drop, instant crate)
- **Streak Bonuses** - Daily login streaks multiply `/daily` and `/work` rewards

<br>

## Prestige System Overhaul

### Prestige Mechanics

- **Point Reset**: You keep 5% of your Points as a cushion post-prestige
- **Gem Reward Formula**: `Gems = floor(sqrt(TotalPoints / 1000))`
  - Example: 1M Points = ~31 Gems
  - Example: 10M Points = ~100 Gems
- **Prestige Preview**: `/prestige` shows exact Gem reward BEFORE confirming
- **Confirmation Required**: User must type `/prestige confirm` to finalize

### Next Prestige Tracker

- Always visible in `/balance` and `/stats`:
  - "Next Milestone: 2.5M Points (+50 Gems)"
  - Progress bar: `[████████░░] 80%`

### Prestige Milestones

Every 5 prestiges unlocks a bonus:

- Prestige 5: +10% `/work` earnings
- Prestige 10: +1 Gem per prestige permanently
- Prestige 25: Unlock exclusive prestige-only crate in shop
- Prestige 50: Custom profile badge
- Prestige 100: ???

<br>

## Inventory Structure

Inventory is globally stored and split into:

| Section  | Contents                                                                   | Capacity Limit          |
| -------- | -------------------------------------------------------------------------- | ----------------------- |
| Items    | Crate rewards. Used to increase gameplay stats like multiplier, luck.      | 500 (upgradable in shop) |
| Upgrades | Permanent shop-only bonuses (multiplier, luck, etc). Non-tradable. Capped. | 50 (fixed)              |
| Crates   | Openable containers containing random items.                               | 100 (upgradable in shop) |

- Items are only obtained through **crates** (or trading).
- Upgrades are only bought using **Gems** from the **Prestige Shop**.
- Crates are obtained by spending Gems or through event drops.
- **Capacity Overflow Handling**: If inventory is full, opening crates or receiving trades is blocked with error message.

### Inventory Capacity Upgrades

Sold in Prestige Shop:

- **Item Storage Upgrade** - +50 item slots per purchase (Max 10 purchases = 1000 total)
- **Crate Storage Upgrade** - +25 crate slots per purchase (Max 8 purchases = 300 total)

### Upgrade System (Detailed)

| Upgrade Name            | Effect                          | Max Level | Cost Scaling         |
| ----------------------- | ------------------------------- | --------- | -------------------- |
| Points Multiplier       | +10% Points per level           | 10        | `50 * level^2` Gems  |
| Gem Bonus (Prestige)    | +1 Gem per prestige per level   | 5         | `100 * level^2` Gems |
| Work Cooldown Reduction | -30s cooldown per level         | 5         | `75 * level^2` Gems  |
| Luck Boost              | +5% rare drop chance per level  | 10        | `60 * level^2` Gems  |
| Daily Bonus Multiplier  | +20% `/daily` rewards per level | 5         | `80 * level^2` Gems  |

- Upgrades use **exponential cost scaling** to balance late-game
- All upgrades are **permanent** and **account-bound**
- Displayed in `/inventory upgrades` with current level and next cost

<br>

## Item Rarity System

| Rarity    | Drop Chance Weight | Color Code (Embed) |
| --------- | ------------------ | ------------------ |
| Common    | 50%                | Gray (#95a5a6)     |
| Uncommon  | 30%                | Green (#2ecc71)    |
| Rare      | 12%                | Blue (#3498db)     |
| Epic      | 5%                 | Purple (#9b59b6)   |
| Legendary | 2.5%               | Orange (#e67e22)   |
| Mythic    | 0.5%               | Red (#e74c3c)      |

- Each crate defines its own loot table with rarity weights
- RNG uses **weighted random selection** based on rarity percentages
- Drop rates are **per-crate configurable** in JSON (some crates may have 0% Mythic, others boosted Legendary, etc.)

### Item Showcase Feature

- `/showcase [item_id]` - Set an item as your "featured pull"
- Displayed in `/profile` command with rarity color and image
- Can showcase your rarest/favorite item publicly
- Only one item showcased at a time

<br>

## Number Formatting System

To ensure readability, all large numerical values (Points, Gems, etc.) will be shortened in output.

| Raw Value       | Displayed As |
| --------------- | ------------ |
| `1,000`         | `1K`         |
| `100,000`       | `100K`       |
| `1,000,000`     | `1M`         |
| `1,000,000,000` | `1B`         |
| `1e12`          | `1T`         |
| `1e15`          | `1Qa`        |
| `1e18`          | `1Qi`        |
| `1e21`          | `1Sx`        |
| `1e24`          | `1Sp`        |
| `1e27`          | `1Oc`        |
| `1e30`          | `1No`        |
| `1e33`          | `1Dc`        |
| `1e36`+         | `999Qa+` (cap display, show "MAX" badge) |

- Abbreviate any number ≥ 1000
- Use suffixes: K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc
- **Above 1e36**: Display as `999Qa+` with a golden "MAX" badge/emoji
- Stored values remain unshortened in the database
- Admin/developer commands display full raw values by default

Applies to:

- `/balance`, `/stats`, `/top`
- Inventory displays
- Shop and prestige messages
- Trade/gift confirmations

<br>

## Feature Breakdown

### Crate Opening

- `/opencrate [crate_name]` - Opens a crate from inventory
- Displays rolling result with suspense:
  ```
  🎲 Opening Starter Crate...
  Rolling... Common... Uncommon... Rare...
  ✨ You got: **Golden Sword** (Rare)
  ```
- Use setTimeout delays (1s between rarity reveals) for text-based animation
- Final result shown in embed with item image, rarity color, and stats

### Crate Crafting / Fusion

- `/craft [crate_type] [amount]` - Combine lower-tier crates into higher-tier
- Example: 5x Starter Crates → 1x Advanced Crate
- Fusion recipes defined in static JSON
- Crafting costs no Gems, only crate consumption

### Prestige & Prestige Shop

- `/prestige` - Shows preview of Gems earned and confirmation prompt
- `/prestige confirm` - Finalizes prestige, resets Points (keeps 5%), awards Gems
- `/shop prestige` - Lists all upgrades with costs, levels, and effects
- `/buy upgrade [name]` - Purchase upgrade if you have enough Gems and haven't hit cap

### Leaderboards

- `/top [category]` - Display top 10 users by:
  - `points` - Current Points balance
  - `gems` - Current Gems balance
  - `prestiges` - Total prestige count
  - `crates` - Total crates opened
  - `trades` - Total successful trades
  - `networth` - Combined value of Points + Gems + Items (calculated on-demand)

- **Leaderboard Caching**: Results cached for 5 minutes to prevent spam queries
- **Time Period Filters** (Future): `/top [category] daily/weekly/alltime`

### Event System

- **Event Crates** appear in `/shop crates` during limited-time events
- Crates remain purchasable with Gems only during event window (e.g., 7 days)
- After event ends:
  - Crates disappear from shop
  - Existing crates in user inventory remain usable
  - Items obtained are marked with event tag: `[Halloween 2024]`, `[Winter Event]`
- Event items are **fully tradable** forever
- Event tag displayed in item name/description for collection tracking

### Trading System

- `/trade @user` - Initiates trade menu
- Interactive menu (buttons/select menus):
  - Both users add items/gems to trade
  - Both users must click "Confirm" button
  - 30 second timeout if no response
- **Trade Cooldown**: 60 seconds between trades (per user)
- **Fraud Prevention**:
  - Both parties must confirm trade explicitly
  - Display full trade contents before confirmation
  - Log all trades to webhook + local file
  - No takebacks after confirmation

### Gifting System

- `/gift @user [item_id or gems] [amount]` - One-way transfer
- No confirmation from receiver needed
- Cooldown: 5 minutes between gifts
- Cannot gift Points or Upgrades
- Logged to webhook for audit trail

### Logging & Auditing

- **Webhook Logging** (Discord webhook URL in config):
  - Admin commands executed (grant, blacklist, etc.)
  - Trade completions (both parties, items/gems exchanged)
  - Prestige events (user, Points reset, Gems earned)
  - Crate openings with rare drops (Epic+)
  - Error events (command failures, database errors)

- **Local File Logging** (rotating daily logs):
  - All command usage (user, command, timestamp)
  - Database write operations
  - Trade/gift history (detailed)
  - Error stack traces

### Stats Tracking

- Lifetime counters stored per user:
  - `total_points_earned` - Sum of all Points ever earned (not current balance)
  - `total_gems_earned` - Sum of all Gems ever earned
  - `total_gems_spent` - Sum of all Gems spent
  - `crates_opened` - Total crates opened
  - `trades_completed` - Successful trades
  - `gifts_sent` - Gifts sent
  - `gifts_received` - Gifts received
  - `prestiges` - Total prestige count
  - `commands_used` - Total command invocations
  - `daily_streak` - Consecutive days using `/daily`
  - `account_created` - Timestamp of first command usage
  - `last_seen` - Timestamp of last command

- Displayed in `/stats` command with formatted numbers

### Achievements System

One-time milestone rewards:

| Achievement                   | Requirement          | Reward         |
| ----------------------------- | -------------------- | -------------- |
| First Steps                   | Use `/work` 1 time   | 10 Gems        |
| Lucky Opener                  | Open 10 crates       | 25 Gems        |
| Crate Addict                  | Open 100 crates      | 100 Gems       |
| Prestige Beginner             | Prestige 1 time      | 50 Gems        |
| Prestige Veteran              | Prestige 10 times    | 250 Gems       |
| Trader                        | Complete 10 trades   | 50 Gems        |
| Rare Collector                | Own 50 Rare+ items   | 100 Gems       |
| Legendary Luck                | Unbox a Legendary    | 200 Gems       |
| Mythic Hunter                 | Unbox a Mythic       | 500 Gems       |
| Generous                      | Gift 10 times        | 75 Gems        |
| Streak Master                 | 30 day daily streak  | 300 Gems       |

- Achievements tracked in `user_achievements` table
- `/achievements` command lists all achievements with progress bars
- Pop-up notification when achievement unlocked

<br>

## Developer Commands (Prefix)

All developer commands use prefix (e.g., `!`) and require developer role check.

| Command                                      | Description                                      |
| -------------------------------------------- | ------------------------------------------------ |
| `!blacklist add <user_id>`                   | Prevent user from using bot                      |
| `!blacklist remove <user_id>`                | Remove user from blacklist                       |
| `!blacklist list`                            | Show all blacklisted users                       |
| `!grant points <user_id> <amount>`           | Add Points to user                               |
| `!grant gems <user_id> <amount>`             | Add Gems to user                                 |
| `!grant item <user_id> <item_id> <amount>`   | Add item to user inventory                       |
| `!grant crate <user_id> <crate_id> <amount>` | Add crate to user inventory                      |
| `!reset <user_id>`                           | Wipe user data completely                        |
| `!stats <user_id>`                           | View raw user data (full numbers, no formatting) |
| `!reload`                                    | Reload JSON data files without restart           |
| `!backup`                                    | Manually trigger database backup                 |
| `!announce <message>`                        | Send announcement to all guilds (bot is in)      |

<br>

## User Commands (Slash)

All user commands use Discord slash commands (`/`).

| Command                              | Description                                        | Cooldown |
| ------------------------------------ | -------------------------------------------------- | -------- |
| `/work`                              | Earn Points (main grind command)                   | 5 min    |
| `/daily`                             | Claim daily Points + Gem bonus                     | 24 hours |
| `/balance`                           | View your Points, Gems, next prestige milestone    | None     |
| `/stats [user]`                      | View lifetime stats (your own or another user)     | None     |
| `/inventory [items/upgrades/crates]` | View your inventory sections                       | None     |
| `/opencrate <crate_name>`            | Open a crate from your inventory                   | None     |
| `/shop [crates/upgrades]`            | Browse available crates or upgrades                | None     |
| `/buy crate <crate_name> [amount]`   | Purchase crate(s) from shop                        | None     |
| `/buy upgrade <upgrade_name>`        | Purchase upgrade from prestige shop                | None     |
| `/craft <crate_type> <amount>`       | Fuse lower-tier crates into higher-tier            | None     |
| `/prestige`                          | View prestige preview and confirmation prompt      | None     |
| `/prestige confirm`                  | Finalize prestige (reset Points, earn Gems)        | None     |
| `/trade <user>`                      | Initiate trade with another user                   | 60s      |
| `/gift <user> <item/gems> <amount>`  | Gift items or Gems to another user                 | 5 min    |
| `/top <category>`                    | View leaderboard (points, gems, prestiges, etc.)   | None     |
| `/achievements`                      | View your achievement progress                     | None     |
| `/showcase <item_id>`                | Set an item as your featured showcase item         | None     |
| `/profile [user]`                    | View profile card (stats, showcase item, badges)   | None     |

<br>

## Static Game Data Storage

All crates, items, rarities, and images are defined in **static JSON files** stored in `/data/` directory.

### File Structure (Examples)

```
/data
├── crates.json          # All crate definitions
├── items.json           # All item definitions
├── crafting.json        # Crate fusion recipes
├── achievements.json    # Achievement definitions
└── events.json          # Event configurations
```

### Example: `crates.json`

```json
{
  "crates": [
    {
      "id": "starter_crate",
      "name": "Starter Crate",
      "description": "A basic crate for beginners.",
      "cost": 50,
      "image": "https://cdn.example.com/crates/starter.png",
      "loot_table": [
        { "item_id": "iron_sword", "weight": 50 },
        { "item_id": "steel_helmet", "weight": 30 },
        { "item_id": "golden_sword", "weight": 15 },
        { "item_id": "diamond_axe", "weight": 4 },
        { "item_id": "legendary_bow", "weight": 0.9 },
        { "item_id": "mythic_blade", "weight": 0.1 }
      ],
      "event": null
    },
    {
      "id": "halloween_crate",
      "name": "Halloween Crate",
      "description": "Spooky limited edition crate!",
      "cost": 150,
      "image": "https://cdn.example.com/crates/halloween.png",
      "loot_table": [
        { "item_id": "pumpkin_sword", "weight": 40 },
        { "item_id": "ghost_armor", "weight": 35 },
        { "item_id": "cursed_staff", "weight": 20 },
        { "item_id": "witch_hat", "weight": 4 },
        { "item_id": "vampire_blade", "weight": 1 }
      ],
      "event": "Halloween 2024"
    }
  ]
}
```

### Example: `items.json`

```json
{
  "items": [
    {
      "id": "iron_sword",
      "name": "Iron Sword",
      "rarity": "Common",
      "description": "A basic iron blade.",
      "image": "https://cdn.example.com/items/iron_sword.png",
      "stats": {
        "attack": 10
      }
    },
    {
      "id": "mythic_blade",
      "name": "Mythic Blade of Eternity",
      "rarity": "Mythic",
      "description": "A legendary weapon forged by ancient gods.",
      "image": "https://cdn.example.com/items/mythic_blade.png",
      "stats": {
        "attack": 500,
        "crit_chance": 25
      }
    },
    {
      "id": "pumpkin_sword",
      "name": "Pumpkin Sword",
      "rarity": "Rare",
      "description": "A blade carved from haunted pumpkins.",
      "image": "https://cdn.example.com/items/pumpkin_sword.png",
      "event": "Halloween 2024",
      "stats": {
        "attack": 75,
        "spooky": 100
      }
    }
  ]
}
```

### Example: `crafting.json`

```json
{
  "recipes": [
    {
      "input": {
        "crate_id": "starter_crate",
        "amount": 5
      },
      "output": {
        "crate_id": "advanced_crate",
        "amount": 1
      }
    },
    {
      "input": {
        "crate_id": "advanced_crate",
        "amount": 3
      },
      "output": {
        "crate_id": "elite_crate",
        "amount": 1
      }
    }
  ]
}
```

### JSON Schema Validation

On bot startup:
- Load all JSON files into memory
- Validate against JSON schemas (ensure no missing fields, invalid types)
- If validation fails, log error and prevent bot startup
- Use `ajv` library for schema validation in TypeScript

### Developer-Only Editing

- JSON files are manually edited by developers
- No in-bot commands to create/modify crates or items
- Use Git version control for JSON data (track changes, rollback if needed)
- After editing JSON, use `!reload` command to hot-reload data without bot restart

<br>

## Rate Limiting & Cooldowns

### Per-User Cooldowns

Applied to specific commands to prevent spam:

- `/work` - 5 minutes
- `/daily` - 24 hours (resets at midnight UTC)
- `/trade` - 60 seconds
- `/gift` - 5 minutes

### Global Rate Limits

Applied bot-wide to prevent abuse:

- Max 10 commands per user per 10 seconds (sliding window)
- If exceeded, respond with: `⏳ Slow down! Try again in X seconds.`

### Implementation

- Store cooldowns in SQLite `user_cooldowns` table
- On command execution, check timestamp of last usage
- If within cooldown window, deny with error message + remaining time
- Use `Map<user_id, timestamp>` in memory for rate limit tracking (faster than DB)

<br>

## Database Schema (SQLite)

### Core Tables

#### `users`
```
id (TEXT PRIMARY KEY) - Discord user ID
points (INTEGER DEFAULT 0) - Current Points balance
gems (INTEGER DEFAULT 0) - Current Gems balance
prestiges (INTEGER DEFAULT 0) - Total prestige count
created_at (INTEGER) - Unix timestamp
last_seen (INTEGER) - Unix timestamp
daily_streak (INTEGER DEFAULT 0) - Consecutive daily claims
```

#### `user_stats`
```
user_id (TEXT PRIMARY KEY) - Discord user ID
total_points_earned (INTEGER DEFAULT 0)
total_gems_earned (INTEGER DEFAULT 0)
total_gems_spent (INTEGER DEFAULT 0)
crates_opened (INTEGER DEFAULT 0)
trades_completed (INTEGER DEFAULT 0)
gifts_sent (INTEGER DEFAULT 0)
gifts_received (INTEGER DEFAULT 0)
commands_used (INTEGER DEFAULT 0)
```

#### `user_inventory`
```
id (INTEGER PRIMARY KEY AUTOINCREMENT)
user_id (TEXT) - Discord user ID
item_id (TEXT) - Item ID from items.json
amount (INTEGER DEFAULT 1) - Stack size
acquired_at (INTEGER) - Unix timestamp
UNIQUE(user_id, item_id)
```

#### `user_crates`
```
id (INTEGER PRIMARY KEY AUTOINCREMENT)
user_id (TEXT) - Discord user ID
crate_id (TEXT) - Crate ID from crates.json
amount (INTEGER DEFAULT 1)
acquired_at (INTEGER)
UNIQUE(user_id, crate_id)
```

#### `user_upgrades`
```
id (INTEGER PRIMARY KEY AUTOINCREMENT)
user_id (TEXT) - Discord user ID
upgrade_id (TEXT) - Upgrade identifier (e.g., "points_multiplier")
level (INTEGER DEFAULT 1) - Current upgrade level
UNIQUE(user_id, upgrade_id)
```

#### `user_achievements`
```
id (INTEGER PRIMARY KEY AUTOINCREMENT)
user_id (TEXT) - Discord user ID
achievement_id (TEXT) - Achievement identifier
unlocked_at (INTEGER) - Unix timestamp
UNIQUE(user_id, achievement_id)
```

#### `user_cooldowns`
```
user_id (TEXT PRIMARY KEY)
work_last_used (INTEGER DEFAULT 0)
daily_last_used (INTEGER DEFAULT 0)
trade_last_used (INTEGER DEFAULT 0)
gift_last_used (INTEGER DEFAULT 0)
```

#### `blacklist`
```
user_id (TEXT PRIMARY KEY) - Discord user ID
reason (TEXT) - Why user was blacklisted
blacklisted_at (INTEGER) - Unix timestamp
blacklisted_by (TEXT) - Developer who issued blacklist
```

#### `trade_log`
```
id (INTEGER PRIMARY KEY AUTOINCREMENT)
user1_id (TEXT) - First trader
user2_id (TEXT) - Second trader
user1_gave (TEXT) - JSON string of items/gems given
user2_gave (TEXT) - JSON string of items/gems given
completed_at (INTEGER) - Unix timestamp
```

#### `showcase`
```
user_id (TEXT PRIMARY KEY) - Discord user ID
item_id (TEXT) - Item being showcased
set_at (INTEGER) - Unix timestamp
```

### Indexes

```sql
CREATE INDEX idx_users_points ON users(points DESC);
CREATE INDEX idx_users_gems ON users(gems DESC);
CREATE INDEX idx_users_prestiges ON users(prestiges DESC);
CREATE INDEX idx_user_stats_crates ON user_stats(crates_opened DESC);
CREATE INDEX idx_user_inventory_user ON user_inventory(user_id);
CREATE INDEX idx_user_crates_user ON user_crates(user_id);
CREATE INDEX idx_trade_log_users ON trade_log(user1_id, user2_id);
```

<br>

## Modular Development Plan

### Potential Project Structure

```
cratecore-bot/
├── src/
│   ├── commands/
│   │   ├── user/              # Slash commands
│   │   │   ├── work.ts
│   │   │   ├── daily.ts
│   │   │   ├── balance.ts
│   │   │   ├── stats.ts
│   │   │   ├── inventory.ts
│   │   │   ├── opencrate.ts
│   │   │   ├── shop.ts
│   │   │   ├── buy.ts
│   │   │   ├── craft.ts
│   │   │   ├── prestige.ts
│   │   │   ├── trade.ts
│   │   │   ├── gift.ts
│   │   │   ├── top.ts
│   │   │   ├── achievements.ts
│   │   │   ├── showcase.ts
│   │   │   └── profile.ts
│   │   └── developer/         # Prefix commands
│   │       ├── blacklist.ts
│   │       ├── grant.ts
│   │       ├── reset.ts
│   │       ├── reload.ts
│   │       ├── backup.ts
│   │       └── announce.ts
│   ├── managers/
│   │   ├── UserManager.ts       # User CRUD operations
│   │   ├── InventoryManager.ts  # Item/crate inventory logic
│   │   ├── CrateManager.ts      # Crate opening, RNG, loot tables
│   │   ├── PrestigeManager.ts   # Prestige calculations, resets
│   │   ├── UpgradeManager.ts    # Upgrade purchases, level tracking
│   │   ├── TradeManager.ts      # Trade sessions, confirmations
│   │   ├── StatsManager.ts      # Lifetime stat tracking
│   │   ├── AchievementManager.ts # Achievement checks, unlocks
│   │   ├── BlacklistManager.ts  # Blacklist CRUD
│   │   └── EventManager.ts      # Event crate availability logic
│   ├── utils/
│   │   ├── database.ts          # SQLite connection, query helpers
│   │   ├── logger.ts            # Webhook + file logging
│   │   ├── formatNumber.ts      # Number abbreviation (1K, 1M, etc.)
│   │   ├── cooldown.ts          # Cooldown + rate limit checks
│   │   ├── rarityRNG.ts         # Weighted random item selection
│   │   ├── embedBuilder.ts      # Reusable embed templates
│   │   └── validation.ts        # JSON schema validation
│   ├── data/
│   │   ├── crates.json
│   │   ├── items.json
│   │   ├── crafting.json
│   │   ├── achievements.json
│   │   └── events.json
│   ├── schemas/                 # JSON schemas for validation
│   │   ├── crate.schema.json
│   │   ├── item.schema.json
│   │   └── crafting.schema.json
│   ├── index.ts                 # Bot entry point
│   └── config.ts                # Bot token, webhook URLs, dev IDs
├── database/
│   └── cratecore.db             # SQLite database file
├── logs/                        # Local log files
│   └── 2024-10-15.log
├── backups/                     # Automated database backups
│   └── cratecore_2024-10-15.db
├── package.json
├── tsconfig.json
└── README.md
```

### Module Responsibilities

#### `UserManager.ts`
- Create new user on first command usage
- Get/update Points, Gems, prestige count
- Handle daily streak tracking
- Query user balance and stats
- Manage user cooldowns

#### `InventoryManager.ts`
- Add/remove items from user inventory
- Check inventory capacity before adding
- Get user items by rarity filter
- Stack duplicate items (increase amount)
- Item transfer between users (for trades/gifts)
- Crate storage management

#### `CrateManager.ts`
- Load crate definitions from JSON
- Weighted RNG for item selection based on loot table
- Handle crate opening animation text
- Add opened item to user inventory
- Deduct crate from user inventory
- Track crate opening stats

#### `PrestigeManager.ts`
- Calculate Gem reward based on Points formula
- Preview prestige rewards before confirmation
- Reset Points (keep 5%), award Gems
- Increment prestige count
- Update next milestone display
- Check prestige milestone bonuses

#### `UpgradeManager.ts`
- Load upgrade definitions (hardcoded or JSON)
- Check if user can afford upgrade
- Validate upgrade level cap
- Calculate exponential cost scaling
- Apply upgrade purchase (deduct Gems, increment level)
- Get active upgrade effects for calculations

#### `TradeManager.ts`
- Create trade session between two users
- Store pending trade data in memory (Map)
- Handle item/gem additions to trade
- Both-party confirmation logic
- Execute trade (transfer items/gems atomically)
- Log completed trades
- Handle timeouts and cancellations

#### `StatsManager.ts`
- Increment lifetime stat counters
- Query user stats for `/stats` command
- Calculate networth (Points + Gems + item values)
- Track command usage per user
- Update last_seen timestamp

#### `AchievementManager.ts`
- Load achievement definitions from JSON
- Check if user meets achievement requirements
- Award achievement rewards (Gems)
- Mark achievement as unlocked in DB
- Send achievement unlock notification
- Progress tracking for incremental achievements

#### `BlacklistManager.ts`
- Add user to blacklist
- Remove user from blacklist
- Check if user is blacklisted (middleware)
- List all blacklisted users

#### `EventManager.ts`
- Load active events from JSON
- Check if event crates should appear in shop
- Filter crates by event status (active/expired)
- Mark event items with event tag in embeds

<br>

## Implementation Guide (For Solo Student Developer - Me)

### Phase 1: Foundation (Week 1-2)

**Goal**: Get basic bot running with database and command handler.

1. **Setup**:
   - Initialize TypeScript project with `discord.js` v14+
   - Install dependencies: `better-sqlite3`, `ajv` (JSON validation)
   - Create `config.ts` with bot token, developer IDs
   - Setup SQLite database with `database.ts` utility

2. **Database**:
   - Write SQL schema creation scripts
   - Create `users`, `user_stats`, `user_cooldowns` tables first
   - Test basic CRUD operations (insert user, update points)

3. **Command Handler**:
   - Build slash command loader (scan `/commands/user/` folder)
   - Build prefix command loader (scan `/commands/developer/` folder)
   - Implement developer ID check middleware
   - Implement blacklist check middleware

4. **Basic Commands**:
   - `/work` - Award random Points (50-150), save to DB
   - `/balance` - Show Points and Gems from DB
   - `!grant points <user> <amount>` - Admin command to add Points

**Testing**: Verify commands work, database updates correctly, developer commands restricted.

---

### Phase 2: Core Economy (Week 3-4)

**Goal**: Implement full currency system with prestige.

1. **UserManager**:
   - Create/get user functions
   - Add/deduct Points/Gems with validation
   - Update prestige count

2. **PrestigeManager**:
   - Implement `sqrt(Points / 1000)` formula for Gems
   - `/prestige` preview with embed showing reward
   - `/prestige confirm` reset Points (keep 5%), award Gems
   - Track next milestone calculation

3. **Number Formatting**:
   - Write `formatNumber()` utility (K, M, B, T, Qa, etc.)
   - Apply to all balance/stats displays
   - Handle edge cases (negative numbers, decimals)

4. **Cooldown System**:
   - Write `cooldown.ts` utility using DB timestamps
   - Apply to `/work` (5 min)
   - Show remaining time in error messages

5. **Daily Rewards**:
   - `/daily` command with 24h cooldown
   - Award Points + Gems (5-10)
   - Track daily streak in DB
   - Multiply rewards by streak (max 2x at 7 days)

**Testing**: Prestige multiple times, verify Points reset correctly, Gems accumulate, cooldowns work.

---

### Phase 3: Item System (Week 5-6)

**Goal**: Load items/crates from JSON, implement RNG opening.

1. **JSON Data**:
   - Create `crates.json`, `items.json` with examples
   - Write JSON schemas in `/schemas/`
   - Load JSON on bot startup with `ajv` validation

2. **CrateManager**:
   - Parse loot tables from `crates.json`
   - Implement weighted RNG using item weights
   - Test with sample crate (verify drop rates match weights)

3. **InventoryManager**:
   - Create `user_inventory`, `user_crates` tables
   - Add/remove item functions with capacity checks
   - Stack items (increment amount if duplicate)

4. **Commands**:
   - `/inventory items` - List user items with rarity colors
   - `/inventory crates` - List user crates with counts
   - `/opencrate <crate_name>` - Roll RNG, add item, remove crate
   - `/shop crates` - List available crates with costs

5. **Crate Opening Animation**:
   - Use `setTimeout` to edit message 3 times:
     - "Opening crate..."
     - "Rolling... Common... Uncommon..."
     - "You got: **Item Name** (Rarity)"
   - Embed with item image and stats

**Testing**: Buy crate, open it, verify item added to inventory, crate removed, drop rates feel correct.

---

### Phase 4: Shop & Upgrades (Week 7)

**Goal**: Implement upgrade shop with exponential scaling.

1. **UpgradeManager**:
   - Define upgrade configs (name, max level, cost formula, effect)
   - Create `user_upgrades` table
   - Purchase upgrade function (validate level cap, deduct Gems)
   - Get active upgrades for user

2. **Commands**:
   - `/shop upgrades` - List all upgrades with costs and levels
   - `/buy upgrade <name>` - Purchase upgrade
   - `/inventory upgrades` - Show owned upgrades with levels

3. **Apply Upgrade Effects**:
   - Modify `/work` to use Points Multiplier upgrade
   - Modify `/daily` to use Daily Bonus Multiplier
   - Apply Luck Boost to crate RNG (increase rare drop chance)

4. **Inventory Capacity**:
   - Default limits: 500 items, 100 crates
   - Create capacity upgrades in shop
   - Block item/crate additions if at capacity

**Testing**: Buy upgrades, verify effects apply, cost scaling works, level caps enforced.

---

### Phase 5: Trading & Gifting (Week 8)

**Goal**: Secure player-to-player economy.

1. **TradeManager**:
   - Create `Map<trade_id, TradeSession>` in memory
   - Trade session stores: users, items offered, gems offered, confirmations
   - 30 second timeout auto-cancels trade

2. **Trade Flow**:
   - `/trade @user` creates session
   - Interactive buttons: "Add Item", "Add Gems", "Confirm", "Cancel"
   - Both users see live trade contents
   - Both must click "Confirm" to execute
   - Atomically transfer items/gems in DB transaction

3. **Fraud Prevention**:
   - Check both users have items/gems they're offering
   - Prevent trading if either inventory at capacity
   - 60s cooldown between trades
   - Log to webhook + `trade_log` table

4. **Gifting**:
   - `/gift @user <item/gems> <amount>` - One-way transfer
   - No confirmation from receiver
   - 5 min cooldown
   - Same capacity checks as trading
   - Log to webhook

**Testing**: Trade items between test accounts, verify both sides update correctly, try exploit scenarios (capacity overflow, duplicate trades).

---

### Phase 6: Leaderboards & Stats (Week 9)

**Goal**: Competitive features and data tracking.

1. **StatsManager**:
   - Increment lifetime counters on every action:
     - Points earned after `/work`, `/daily`
     - Gems earned after prestige
     - Gems spent after shop purchases
     - Crates opened after `/opencrate`
     - Trades/gifts completed
   - `/stats [user]` command displays formatted stats

2. **Leaderboards**:
   - `/top <category>` queries DB sorted by stat
   - Cache results in memory for 5 minutes
   - Display top 10 users with formatted numbers
   - Show user's own rank if not in top 10

3. **Profile Command**:
   - `/profile [user]` shows:
     - Balance (Points, Gems)
     - Total prestiges
     - Daily streak
     - Showcased item (if set)
     - Account age

4. **Showcase System**:
   - `/showcase <item_id>` sets featured item
   - Store in `showcase` table
   - Display in profile with item image and rarity

**Testing**: Generate fake data, verify leaderboards update correctly, caching works.

---

### Phase 7: Achievements & Events (Week 10)

**Goal**: Add progression milestones and limited-time content.

1. **AchievementManager**:
   - Create `achievements.json` with requirements and rewards
   - Check requirements after every command
   - Award Gems when unlocked
   - Store in `user_achievements` table
   - `/achievements` command shows progress

2. **EventManager**:
   - Create `events.json` with start/end timestamps
   - Filter shop crates by active events
   - Mark event items with tag in item display
   - Event crates disappear from shop after event ends

3. **Crafting System**:
   - Load `crafting.json` with fusion recipes
   - `/craft <crate_type> <amount>` checks recipe
   - Deduct input crates, award output crate
   - No Gem cost for crafting

**Testing**: Unlock achievements, verify rewards, test event crate visibility before/during/after event.

---

### Phase 8: Logging & Admin Tools (Week 11)

**Goal**: Production-ready monitoring and moderation.

1. **Logger**:
   - Webhook logging for important events (trades, prestiges, errors)
   - Local file logging (rotating daily)
   - Include timestamps, user IDs, action details

2. **Developer Commands**:
   - `!blacklist add/remove/list`
   - `!grant` for all currency/items
   - `!reset <user>` wipes user data
   - `!reload` hot-reloads JSON files
   - `!backup` manually triggers DB backup
   - `!announce` sends message to all guilds

3. **Automated Backups**:
   - Daily SQLite file copy to `/backups/` folder
   - Use `setInterval` or cron job
   - Keep last 7 days of backups

4. **Error Handling**:
   - Try-catch all DB operations
   - Log stack traces to file
   - Send user-friendly error messages
   - Webhook alert on critical errors

**Testing**: Trigger errors intentionally, verify logs created, backups work, admin commands restricted.

---

### Phase 9: Polish & Optimization (Week 12)

**Goal**: Performance tuning and UX improvements.

1. **Rate Limiting**:
   - Implement global rate limiter (10 commands per 10s per user)
   - Use in-memory `Map` for speed
   - Sliding window algorithm

2. **Embed Templates**:
   - Create reusable embed builder in `embedBuilder.ts`
   - Consistent colors, footers, timestamps
   - Thumbnail images from JSON data

3. **Random Encounters**:
   - 10% chance during `/work` to trigger:
     - Bonus Points (2x multiplier)
     - Rare Gem drop (1-5 Gems)
     - Instant free crate
   - Announce in message with special emoji

4. **UI Improvements**:
   - Add progress bars to prestige preview
   - Use Discord buttons/select menus where possible
   - Pagination for long lists (inventory, leaderboards)

5. **Performance**:
   - Add DB indexes for common queries
   - Cache frequently accessed data (upgrade configs, JSON)
   - Batch DB writes where possible

**Testing**: Load test with multiple users, verify performance, check embed rendering, test edge cases.

---

### Phase 10: Deployment & Maintenance (Week 13+)

**Goal**: Go live and iterate based on feedback.

1. **Hosting**:
   - Deploy to VPS, cloud provider, or hosting service
   - Use PM2 or systemd for process management
   - Enable automatic restarts on crash

2. **Monitoring**:
   - Setup uptime monitoring (UptimeRobot, etc.)
   - Discord webhook for bot status alerts
   - Track DB size growth

3. **Community Feedback**:
   - Create feedback channel in support server
   - Track feature requests in GitHub issues
   - Monitor for economy exploits or bugs

4. **Balancing**:
   - Adjust prestige formula if too easy/hard
   - Tweak drop rates based on player feedback
   - Add new crates/items regularly to keep interest

5. **Future Expansion**:
   - Migrate to PostgreSQL when user base grows
   - Add more event types (double XP weekends, special boss fights)
   - Implement seasons with leaderboard resets
