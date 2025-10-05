# CrateCore Discord Bot - Feature Brainstorm v3

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
| **Gems**               | Premium currency earned by prestiging, daily rewards, achievements, or events. Persistent. Non-tradable. |
| **Items**              | Loot obtained from crates. Items have **rarity tiers** affecting drop chance and value.              |
| **Crates**             | Containers bought with Gems, opened to yield items with rarity-weighted RNG.                         |
| **Upgrades**           | Permanent effects (e.g., multiplier, luck) bought with Gems in shop. Non-tradable. Capped.           |
| **Prestige**           | Reset Points (keep 5% as cushion), earn Gems based on Points total, increase prestige count.         |
| **Stats**              | Track lifetime player data (coins earned, crates opened, prestiges, etc.)               |
| **Inventory**          | Split into three categories: **Items**, **Upgrades**, **Crates**. Hard capacity limits apply.        |
| **Gifting**    | Only Gems and Items can be gifted. Points and Upgrades cannot.                                |
| **Blacklist**          | Dev-only command to restrict bot access for certain users.                                           |
| **Developer Commands** | Prefix commands limited to developers (admin actions, manual item/currency grants).                  |
| **User Commands**      | Slash commands accessible to all non-blacklisted users.                                              |
| **Events**             | Limited-time crates available in shop. Crates/items persist after event, marked with event tag.      |

<br>

## Currency System

| Currency | Tradable | Reset on Prestige | Use Case                                       |
| -------- | -------- | ----------------- | ---------------------------------------------- |
| Points   | No       | Yes (keeps 5%)    | Earned through gameplay, prestige trigger only |
| Gems     | No      | No                | Buy crates, items, upgrades, gift       |

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

- Items are only obtained through **crates** (or gifting).
- Upgrades are only bought using **Gems** from the **Prestige Shop**.
- Crates are obtained by spending Gems or through event drops.
- **Capacity Overflow Handling**: If inventory is full, opening crates or receiving gifts is blocked with error message.

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
- Gift confirmations

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
- Event items are **fully giftable** forever
- Event tag displayed in item name/description for collection tracking

### Gifting System

- `/gift @user [item_id or gems] [amount]` - One-way transfer
- No confirmation from receiver needed
- Cooldown: 5 minutes between gifts
- Cannot gift Points or Upgrades
- Logged to webhook for audit trail

### Logging & Auditing

- **Webhook Logging** (Discord webhook URL in config):
  - Admin commands executed (grant, blacklist, etc.)
  - Gift completions (both parties, items/gems exchanged)
  - Prestige events (user, Points reset, Gems earned)
  - Crate openings with rare drops (Epic+)
  - Error events (command failures, database errors)

- **Local File Logging** (rotating daily logs):
  - All command usage (user, command, timestamp)
  - Database write operations
  - Gift history (detailed)
  - Error stack traces

### Stats Tracking

- Lifetime counters stored per user:
  - `total_points_earned` - Sum of all Points ever earned (not current balance)
  - `total_gems_earned` - Sum of all Gems ever earned
  - `total_gems_spent` - Sum of all Gems spent
  - `crates_opened` - Total crates opened
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
