# CrateCore Discord Bot - Feature Brainstorm

<br>

## Overview

CrateCore is a text-based Discord bot simulating a crate/item economy game inspired by clicker/rebirth simulators.
Users earn **Points**, prestige to earn **Gems**, and spend Gems to buy crates, items, and permanent upgrades.
The game is fully command-driven, scalable, and designed for long-term user engagement.

<br>

## Core Concepts

| Concept                | Description                                                                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------- |
| **Points**             | Temporary currency earned by gameplay (e.g., `/work`). Resets on prestige. Non-tradable.       |
| **Gems**               | Premium currency earned by prestiging or special events. Persistent across sessions. Tradable. |
| **Items**              | Loot obtained from crates. Items have **rarity tiers** affecting drop chance and value.        |
| **Crates**             | Containers bought with Gems, opened to yield items with rarity-weighted RNG.                   |
| **Upgrades**           | Permanent effects (e.g., multiplier, luck) bought with Gems in shop. Non-tradable. Capped.     |
| **Prestige**           | Reset Points, earn Gems, increase leaderboard prestige count.                                  |
| **Stats**              | Track lifetime player data (coins earned, crates opened, trades made, prestiges, etc.)         |
| **Inventory**          | Split into three categories: **Items**, **Upgrades**, **Crates**.                              |
| **Trading/Gifting**    | Only Gems and Items can be traded/gifted. Points and Upgrades cannot.                          |
| **Blacklist**          | Dev-only command to restrict bot access for certain users.                                     |
| **Developer Commands** | Prefix commands limited to developers (admin actions, manual item/currency grants).            |
| **User Commands**      | Slash commands accessible to all non-blacklisted users.                                        |

<br>

## Currency System

| Currency | Tradable | Reset on Prestige | Use Case                                       |
| -------- | -------- | ----------------- | ---------------------------------------------- |
| Points   | No       | Yes               | Earned through gameplay, prestige trigger only |
| Gems     | Yes      | No                | Buy crates, items, upgrades, trade, gift       |

<br>

## Inventory Structure

Inventory is globally stored and split into:

| Section  | Contents                                                                   |
| -------- | -------------------------------------------------------------------------- |
| Items    | Crate rewards. Used to increase gameplay stats like multiplier, luck.      |
| Upgrades | Permanent shop-only bonuses (multiplier, luck, etc). Non-tradable. Capped. |
| Crates   | Openable containers containing random items.                               |

* Items are only obtained through **crates**.
* Upgrades are only bought using **Gems** from the **Prestige Shop**.
* Crates are obtained by spending Gems or through special event/reward drops.

### Upgrade System (Summary)

* Purchasable only via Gems
* Hard capped (e.g., Multiplier Upgrade → Max Level 10)
* Not tradable
* Stored separately from items

<br>

## Item Rarity System

| Rarity    | Drop Chance Impact | Value Impact   |
| --------- | ------------------ | -------------- |
| Common    | High               | Low            |
| Uncommon  | Moderate           | Moderate       |
| Rare      | Low                | High           |
| Epic      | Very Low           | Very High      |
| Legendary | Extremely Low      | Extremely High |
| Mythic    | Ultra Rare         | Ultra High     |

* Each crate defines drop chances per item rarity.
* RNG weighted by rarity for crate openings.

<br>

## Number Formatting System

To ensure readability, all large numerical values (Points, Gems, etc.) will be shortened in output.

| Raw Value       | Displayed As |
| --------------- | ------------ |
| `1,000`         | `1K`         |
| `100,000`       | `100K`       |
| `1,000,000`     | `1M`         |
| `1,000,000,000` | `1B`         |
| `1e15`          | `1Q`         |
| `1e50`          | `1e50`       |

* Abbreviate any number ≥ 1000.
* Use suffixes: K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc, etc.
* Scientific notation used if value > `1e36`.
* Stored values remain unshortened in the database.
* Admin/developer commands will display full values by default.

Applies to:

* `/balance`, `/stats`, `/top`
* Inventory displays
* Shop and prestige messages

<br>

## Feature Breakdown

### Inventory

* Split into three subcategories:

  * **Items**: Equipables from crates (affect stats)
  * **Upgrades**: Bought from shop with Gems, persistent, capped
  * **Crates**: Consumables to obtain items

### Crate Crafting / Fusion

* Combine multiple crates into higher-tier crates (optional enhancement).

### Prestige & Prestige Shop

* Prestiging resets Points but awards Gems and increases prestige count.
* Prestige Shop sells permanent upgrades purchasable only by Gems:

  * Inventory capacity
  * Multiplier increases
  * Extra gems per prestige
  * Shorter cooldowns
  * Higher crate drop chance

### Leaderboards

* Display top users by various metrics: Points, Gems, Prestiges, Crates opened, Trades made, etc.

### Event System

* Time-limited crates or bonuses.

### Logging & Auditing

* Global logging via webhook for all important events (admin commands, errors, trades, crate openings).
* Logs stored in files locally for audit.

### Stats Tracking

* Lifetime counters for user actions: total Points earned, Gems earned/spent, crates opened, trades made, prestiges, commands used.

### Developer Commands (Prefix)

* Blacklist users
* Grant items, crates, points, or gems
* Manual override systems
* Configuration reload

### User Commands (Slash)

* `/work` - earn Points
* `/opencrate`
* `/buycrate`
* `/shop`
* `/prestige`
* `/stats`
* `/inventory`
* `/trade`
* `/gift`
* `/balance`
* `/top`
* More TBA

<br>

## Constraints & Rules

| Action / Feature | Allowed | Notes                                               |
| ---------------- | ------- | --------------------------------------------------- |
| Points Trading   | X       | Points are non-tradable and reset on prestige       |
| Gems Trading     | ✓       | Gems can be traded or gifted to other users         |
| Item Trading     | ✓       | Items can be traded or gifted between users         |
| Upgrade Trading  | X       | Upgrades are locked to user and non-tradable        |
| Shop Purchases   | ✓       | Only Gems used to buy crates and upgrades           |
| Crates from Shop | ✓       | All crates are Gem-purchasable or event-earned only |

* Bot settings, economy, and database are **global**, not per-guild.
* Developer commands are **prefix-only**.
* User commands are **slash-only**.

<br>

## Static Game Data Storage

* All existing crates, items, rarities, and images are defined in **static JSON files**.
* These files are **developer-only editable**; new content is added manually, not through commands.
* On bot startup, the bot loads all JSON data into memory for fast RNG pulls.
* Example structure:

```json
{
  "crates": [
    {
      "id": "starter_crate",
      "name": "Starter Crate",
      "image": "https://cdn.example.com/crates/starter.png",
      "items": [
        {
          "id": "iron_sword",
          "name": "Iron Sword",
          "rarity": "Common",
          "image": "https://cdn.example.com/items/iron_sword.png"
        },
        {
          "id": "golden_sword",
          "name": "Golden Sword",
          "rarity": "Rare",
          "image": "https://cdn.example.com/items/golden_sword.png"
        }
      ]
    }
  ]
}
```

* Images are referenced by URL (CDN, GitHub repo, etc.).
* Data includes rarity, flavor text, and release event if desired.
* User inventories only store **item IDs**; metadata comes from static JSON.
* Use JSON Schema validation at startup to ensure no malformed crate/item definitions.

<br>

## Optional (Future Considerations)

* Item Collection Tracker: Track % of items obtained per crate.
* Flex Titles: Unlockable user tags for status display.
* Item Skins: Cosmetic-only item variants.
* Booster system: Temporary buffs purchasable with Gems.
* Referral system: Gems awarded for bringing new users (requires abuse control).
* Limited Edition Crates/Items per Event

<br>

## Modular Development Plan

* `CommandHandler`
* `Database (SQLite)`
* `UserManager`
* `InventoryManager`
* `CrateManager`
* `PrestigeManager`
* `UpgradeManager`
* `TradeManager`
* `StatsManager`
* `BlacklistManager`
* `EventManager`
* `Logger` (Webhook + file)
* `Utils` (number formatting, cooldowns, rarity weight calc, etc.)

<br>

## Final Notes

The bot is designed for infinite progression, competitive grinding, and strategic resource use.
No bloated features. Every system supports retention and replayability.
Core is stable. Development can begin.
