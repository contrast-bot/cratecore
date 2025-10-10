# CrateCore Discord Bot - Feature Brainstorm v4

<br>

## Overview

CrateCore is a text-based Discord bot simulating a crate/item economy game inspired by CS2/CSGO case opening.
Users earn **Cash**, spend **Gems** to buy crates, items, and permanent upgrades.
The game is fully command-driven, scalable, and designed for long-term user engagement with competitive leaderboards.

<br>

## Core Concepts

| Concept                | Description                                                                                              |
| ---------------------- | -------------------------------------------------------------------------------------------------------- |
| **Cash**               | Primary currency earned by gameplay (e.g., `/work`, `/daily`). Persistent. Non-tradable.                 |
| **Gems**               | Premium currency earned through daily rewards, achievements, or events. Persistent. Non-tradable.        |
| **Items**              | Loot obtained from crates. Items have **rarity tiers** affecting drop chance and value.                  |
| **Collections**        | Each item belongs to a collection (which crate it originates from).                                      |
| **Crates**             | Containers bought with Gems, opened to yield items with rarity-based RNG.                                |
| **Upgrades**           | Permanent effects (e.g., multiplier, luck) bought with Gems in shop. Non-tradable. Capped.               |
| **Stats**              | Track lifetime player data (cash earned, crates opened, etc.)                                            |
| **Inventory**          | Split into three categories: **Items**, **Upgrades**, **Crates**. Hard capacity limits apply.            |
| **Gifting**            | Only Gems and Items can be gifted. Cash and Upgrades cannot.                                             |
| **Blacklist**          | Dev-only command to restrict bot access for certain users.                                               |
| **Developer Commands** | Prefix commands limited to allowed user IDs (admin actions, manual item/currency grants).                |
| **User Commands**      | Slash commands accessible to all non-blacklisted users.                                                  |
| **Events**             | Limited-time crates available in shop. Crates/items persist after event, marked with event tag.          |

<br>

## Currency System

| Currency | Tradable | Use Case                                       |
| -------- | -------- | ---------------------------------------------- |
| Cash     | No       | Earned through gameplay, main currency         |
| Gems     | No       | Buy crates, items, upgrades, gift              |

### Gem Sources

- **Daily Rewards** - `/daily` command (5-10 Gems/day)
- **Login Streaks** - Consecutive daily claims multiply rewards
- **Achievements** - One-time milestone bonuses (100 crates opened, etc.)
- **Vote Rewards** - (Optional) If listed on bot directories later
- **Event Bonuses** - Special event participation rewards

### Cash Sources

- `/work` - Primary grind command (5 min cooldown)
- `/daily` - Bonus Cash once per 24h (larger than work, no cooldown conflicts)
- **Random Encounters** - 10% chance during `/work` to trigger mini-event (bonus cash, rare gem drop, instant crate)
- **Streak Bonuses** - Daily login streaks multiply `/daily` and `/work` rewards

<br>

## Inventory Structure

Inventory is globally stored and split into:

| Section  | Contents                                                                   | Capacity Limit           |
| -------- | -------------------------------------------------------------------------- | ------------------------ |
| Items    | Crate rewards. Collectible items with rarity and collection tags.          | 500 (upgradable in shop) |
| Upgrades | Permanent shop-only bonuses (multiplier, luck, etc). Non-tradable. Capped. | 50 (fixed)               |
| Crates   | Openable containers containing random items.                               | 100 (upgradable in shop) |

- Items are only obtained through **crates** (or gifting).
- Upgrades are only bought using **Gems** from the **Shop**.
- Crates are obtained by spending Gems or through event drops.
- **Capacity Overflow Handling**: If inventory is full, opening crates or receiving gifts is blocked with error message.

### Inventory Capacity Upgrades

Sold in Shop:

- **Item Storage Upgrade** - +50 item slots per purchase (Max 10 purchases = 1000 total)
- **Crate Storage Upgrade** - +25 crate slots per purchase (Max 8 purchases = 300 total)

### Upgrade System (Detailed)

| Upgrade Name            | Effect                          | Max Level | Cost Scaling         |
| ----------------------- | ------------------------------- | --------- | -------------------- |
| Cash Multiplier         | +10% Cash per level             | 10        | `50 * level^2` Gems  |
| Work Cooldown Reduction | -30s cooldown per level         | 5         | `75 * level^2` Gems  |
| Luck Boost              | +5% rare drop chance per level  | 10        | `60 * level^2` Gems  |
| Daily Bonus Multiplier  | +20% `/daily` rewards per level | 5         | `80 * level^2` Gems  |

- Upgrades use **exponential cost scaling** to balance late-game
- All upgrades are **permanent** and **account-bound**
- Displayed in `/inventory upgrades` with current level and next cost

<br>

## Item & Collection System

### Item Properties

Each item has:
- **ID** - Unique identifier
- **Name** - Display name
- **Rarity** - Common, Uncommon, Rare, Epic, Legendary, Mythic
- **Collection** - Which crate it originates from (e.g., "Starter Collection", "Halloween Collection")
- **Description** - Flavor text
- **Image URL** - Hosted on `contrast-bot.github.io/data/images/items/[item_id].png`
- **Event Tag** (optional) - If from limited-time event (e.g., "Halloween 2024")

Items have **no gameplay effects** - they are purely collectible for gambling/trading purposes.

### Collection Tracking

- Each item shows its collection in `/inventory items` view
- Collections are displayed in item embeds and showcase
- Users can filter inventory by collection (future feature)

### Item Showcase Feature

- `/showcase [item_id]` - Set an item as your "featured pull"
- Displayed in `/profile` command with rarity color and image
- Can showcase your rarest/favorite item publicly
- Only one item showcased at a time

<br>

## Rarity System

| Rarity    | Drop Chance | Color Code (Embed) |
| --------- | ----------- | ------------------ |
| Common    | 50%         | Gray (#95a5a6)     |
| Uncommon  | 30%         | Green (#2ecc71)    |
| Rare      | 12%         | Blue (#3498db)     |
| Epic      | 5%          | Purple (#9b59b6)   |
| Legendary | 2.5%        | Orange (#e67e22)   |
| Mythic    | 0.5%        | Red (#e74c3c)      |

- Each crate defines its own loot table with rarity percentages
- RNG uses **percentage-based random selection**
- Drop rates are **per-crate configurable** in JSON (some crates may have 0% Mythic, others boosted Legendary, etc.)

<br>

## Number Formatting System

To ensure readability, all large numerical values (Cash, Gems, etc.) will be shortened in output.

| Raw Value       | Displayed As                              |
| --------------- | ----------------------------------------- |
| `1,000`         | `1K`                                      |
| `100,000`       | `100K`                                    |
| `1,000,000`     | `1M`                                      |
| `1,000,000,000` | `1B`                                      |
| `1e12`          | `1T`                                      |
| `1e15`          | `1Qa`                                     |
| `1e18`          | `1Qi`                                     |
| `1e21`          | `1Sx`                                     |
| `1e24`          | `1Sp`                                     |
| `1e27`          | `1Oc`                                     |
| `1e30`          | `1No`                                     |
| `1e33`          | `1Dc`                                     |
| `1e36`+         | `999Qa+` (cap display, show "MAX" badge)  |

- Abbreviate any number ≥ 1000
- Use suffixes: K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc
- **Above 1e36**: Display as `999Qa+` with a golden "MAX" badge/emoji
- Stored values remain unshortened in the database
- Admin/developer commands display full raw values by default

Applies to:

- `/balance`, `/stats`, `/top`
- Inventory displays
- Shop messages
- Gift confirmations

<br>

## Feature Breakdown

### Crate Opening

- `/opencrate [crate_name]` - Opens a crate from inventory instantly
- Final result shown in embed with item image, rarity color, collection tag, and description
- No animation - immediate result display

### Crate Crafting / Fusion

- `/craft [crate_type] [amount]` - Combine lower-tier crates into higher-tier
- Example: 5x Starter Crates → 1x Advanced Crate
- Fusion recipes defined in static JSON
- Crafting costs no Gems, only crate consumption

### Shop

- `/shop [crates/upgrades]` - Lists all upgrades with costs, levels, and effects
- `/buy crate <crate_name> [amount]` - Purchase crate(s) from shop
- `/buy upgrade <upgrade_name>` - Purchase upgrade if you have enough Gems and haven't hit cap

### Leaderboards

- `/top [category]` - Display top 10 users by:
  - `cash` - Current Cash balance
  - `gems` - Current Gems balance
  - `crates` - Total crates opened
  - `networth` - Combined value of Cash + Gems + Items (calculated on-demand)

- **Leaderboard Caching**: Results cached for 5 minutes to prevent spam queries

### Event System

- **Event Crates** appear in `/shop crates` during limited-time events
- Crates remain purchasable with Gems only during event window (e.g., 7 days)
- After event ends:
  - Crates disappear from shop
  - Existing crates in user inventory remain usable
  - Items obtained are marked with event tag: `[Halloween 2024]`, `[Winter Event]`
- Event items are **fully giftable** forever
- Event tag displayed in item name/description for collection tracking

### Gifting System

- `/gift @user [item_id or gems] [amount]` - One-way transfer
- No confirmation from receiver needed
- Cooldown: 5 minutes between gifts
- Cannot gift Cash or Upgrades
- Logged to webhook for audit trail

### Feedback System

- `/feedback [message]` - Send feedback to developers
- Sends message via Discord webhook with:
  - User ID, username
  - Timestamp
  - Feedback content
  - Server ID (if applicable)
- Cooldown: 10 minutes between feedback submissions

### Logging & Auditing

- **Webhook Logging** (Discord webhook URL in config):
  - **Developer commands executed** (alert with user ID, command, timestamp, parameters)
  - Gift completions (both parties, items/gems exchanged)
  - Crate openings with rare drops (Epic+)
  - Feedback submissions
  - Error events (command failures, database errors)

- **Local File Logging** (rotating daily logs):
  - All command usage (user, command, timestamp)
  - Database write operations
  - Gift history (detailed)
  - Error stack traces

### Stats Tracking

- Lifetime counters stored per user:
  - `total_cash_earned` - Sum of all Cash ever earned (not current balance)
  - `total_gems_earned` - Sum of all Gems ever earned
  - `total_gems_spent` - Sum of all Gems spent
  - `crates_opened` - Total crates opened
  - `gifts_sent` - Gifts sent
  - `gifts_received` - Gifts received
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

All developer commands use prefix (e.g., `!`) and require user ID to be in allowed list.

Developer commands **automatically send alerts to Discord webhook** when executed.

| Command                                      | Description                                      |
| -------------------------------------------- | ------------------------------------------------ |
| `!blacklist add <user_id>`                   | Prevent user from using bot                      |
| `!blacklist remove <user_id>`                | Remove user from blacklist                       |
| `!blacklist list`                            | Show all blacklisted users                       |
| `!grant cash <user_id> <amount>`             | Add Cash to user                                 |
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
| `/work`                              | Earn Cash (main grind command)                     | 5 min    |
| `/daily`                             | Claim daily Cash + Gem bonus                       | 24 hours |
| `/balance`                           | View your Cash, Gems                               | None     |
| `/stats [user]`                      | View lifetime stats (your own or another user)     | None     |
| `/inventory [items/upgrades/crates]` | View your inventory sections                       | None     |
| `/opencrate <crate_name>`            | Open a crate from your inventory                   | None     |
| `/shop [crates/upgrades]`            | Browse available crates or upgrades                | None     |
| `/buy crate <crate_name> [amount]`   | Purchase crate(s) from shop                        | None     |
| `/buy upgrade <upgrade_name>`        | Purchase upgrade from shop                         | None     |
| `/craft <crate_type> <amount>`       | Fuse lower-tier crates into higher-tier            | None     |
| `/gift <user> <item/gems> <amount>`  | Gift items or Gems to another user                 | 5 min    |
| `/top <category>`                    | View leaderboard (cash, gems, crates, etc.)        | None     |
| `/achievements`                      | View your achievement progress                     | None     |
| `/showcase <item_id>`                | Set an item as your featured showcase item         | None     |
| `/profile [user]`                    | View profile card (stats, showcase item, badges)   | None     |
| `/feedback <message>`                | Send feedback to developers                        | 10 min   |

<br>

## Static Game Data Storage

All crates, items, rarities, and images are defined in **static JSON files** stored in `/data/` directory.

All images are hosted on `contrast-bot.github.io/data/images/`:
- **Crates**: `contrast-bot.github.io/data/images/crates/[crate_id].png`
- **Items**: `contrast-bot.github.io/data/images/items/[item_id].png`
- **Upgrades**: `contrast-bot.github.io/data/images/upgrades/[upgrade_id].png`

### File Structure

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
      "image": "https://contrast-bot.github.io/data/images/crates/starter_crate.png",
      "loot_table": [
        { "item_id": "iron_sword", "chance": 50 },
        { "item_id": "steel_helmet", "chance": 30 },
        { "item_id": "golden_sword", "chance": 15 },
        { "item_id": "diamond_axe", "chance": 4 },
        { "item_id": "legendary_bow", "chance": 0.9 },
        { "item_id": "mythic_blade", "chance": 0.1 }
      ],
      "collection": "Starter Collection",
      "event": null
    },
    {
      "id": "halloween_crate",
      "name": "Halloween Crate",
      "description": "Spooky limited edition crate!",
      "cost": 150,
      "image": "https://contrast-bot.github.io/data/images/crates/halloween_crate.png",
      "loot_table": [
        { "item_id": "pumpkin_sword", "chance": 40 },
        { "item_id": "ghost_armor", "chance": 35 },
        { "item_id": "cursed_staff", "chance": 20 },
        { "item_id": "witch_hat", "chance": 4 },
        { "item_id": "vampire_blade", "chance": 1 }
      ],
      "collection": "Halloween Collection",
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
      "image": "https://contrast-bot.github.io/data/images/items/iron_sword.png",
      "collection": "Starter Collection"
    },
    {
      "id": "mythic_blade",
      "name": "Mythic Blade of Eternity",
      "rarity": "Mythic",
      "description": "A legendary weapon forged by ancient gods.",
      "image": "https://contrast-bot.github.io/data/images/items/mythic_blade.png",
      "collection": "Starter Collection"
    },
    {
      "id": "pumpkin_sword",
      "name": "Pumpkin Sword",
      "rarity": "Rare",
      "description": "A blade carved from haunted pumpkins.",
      "image": "https://contrast-bot.github.io/data/images/items/pumpkin_sword.png",
      "collection": "Halloween Collection",
      "event": "Halloween 2024"
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
- `/gift` - 5 minutes
- `/feedback` - 10 minutes

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
