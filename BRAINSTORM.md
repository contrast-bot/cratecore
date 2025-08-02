# CrateCore Discord Bot — Final Design & Feature Brainstorm

---

## Overview

CrateCore is a text-based Discord bot simulating a crate/item economy game inspired by clicker/rebirth simulators.  
Users earn **Points**, prestige to earn **Gems**, and spend Gems to buy crates, items, and permanent upgrades.  
The game is fully command-driven, scalable, and designed for long-term user engagement.

---

## Core Concepts

| Concept           | Description                                                                                  |
|-------------------|----------------------------------------------------------------------------------------------|
| **Points**        | Temporary currency earned by gameplay (e.g., `/work`). Resets on prestige. Non-tradable.     |
| **Gems**          | Premium currency earned by prestiging or special events. Persistent across sessions. Tradable.|
| **Items**         | Loot obtained from crates. Items have **rarity tiers** affecting drop chance and value.      |
| **Crates**        | Containers bought with Gems, opened to yield items with rarity-weighted RNG.                 |
| **Prestige**      | Reset Points, earn Gems, increase leaderboard prestige count.                               |
| **Upgrades**      | Permanent buffs bought with Gems in the Prestige Shop (e.g., inventory expansion).           |
| **Stats**         | Track lifetime player data (coins earned, crates opened, trades made, prestiges, etc.)       |
| **Inventory**     | Split into two categories: Items and Crates.                                                |
| **Trading/Gifting**| Only Gems and Items can be traded/gifted. Points cannot.                                    |
| **Blacklist**     | Dev-only command to restrict bot access for certain users.                                  |
| **Developer Commands** | Prefix commands limited to developers (admin actions, manual item/currency grants).       |
| **User Commands** | Slash commands accessible to all non-blacklisted users.                                    |

---

## Currency System

| Currency  | Tradable | Reset on Prestige | Use Case                         |
|-----------|----------|-------------------|---------------------------------|
| Points    | No       | Yes               | Earned through gameplay, prestige trigger only |
| Gems      | Yes      | No                | Buy crates, items, upgrades, trade, gift |

---

## Item Rarity System

| Rarity    | Drop Chance Impact       | Value Impact          |
|-----------|-------------------------|----------------------|
| Common    | High                    | Low                  |
| Uncommon  | Moderate                | Moderate             |
| Rare      | Low                     | High                 |
| Epic      | Very Low                | Very High            |
| Legendary | Extremely Low           | Extremely High       |
| Mythic    | Ultra Rare              | Ultra High           |

- Each crate defines drop chances per item rarity.
- RNG weighted by rarity for crate openings.

---

## Number Formatting System

To ensure readability, all large numerical values (Points, Gems, etc.) will be shortened in output.

| Raw Value     | Displayed As |
|---------------|--------------|
| `1,000`       | `1K`         |
| `100,000`     | `100K`       |
| `1,000,000`   | `1M`         |
| `1,000,000,000`| `1B`        |
| `1e15`        | `1Q`         |
| `1e50`        | `1e50`       |

- Abbreviate any number ≥ 1000.
- Use suffixes: K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc, etc.
- Scientific notation used if value > `1e36`.
- Stored values remain unshortened in the database.
- Admin/developer commands will display full values by default.

Applies to:
- `/balance`, `/stats`, `/top`
- Inventory displays
- Shop and prestige messages

---

## Feature Breakdown

### Inventory  
- Items and crates stored separately.  
- Users can hold multiple crates and items.

### Crate Crafting / Fusion  
- Combine multiple crates into higher-tier crates (optional enhancement).

### Prestige & Prestige Shop  
- Prestiging resets Points but awards Gems and increases prestige count.  
- Prestige Shop sells permanent upgrades purchasable only by Gems (inventory expansions, gem bonuses, cooldown reductions, luck boosts).

### Leaderboards  
- Display top users by various metrics: Points, Gems, Prestiges, Crates opened, Trades made, etc.

### Event System  
- Time-limited crates or bonuses.

### Logging & Auditing  
- Global logging via webhook for all important events (admin commands, errors, trades, crate openings).  
- Logs stored in files locally for audit.

### Stats Tracking  
- Lifetime counters for user actions: total Points earned, Gems earned/spent, crates opened, trades made, prestiges, commands used.

### Developer Commands (Prefix)  
- Blacklist users, grant items/currency, test/debug commands.

### User Commands (Slash)  
- Gameplay commands like `/work`, `/opencrate`, `/prestige`, `/trade`, `/stats`.

---

## Constraints & Rules

- Points **cannot** be traded or gifted.  
- Gems and Items **can** be traded or gifted.  
- All crates and items cost Gems to buy.  
- Points serve only as progression score and prestige trigger.  
- Bot settings, economy, and database are **global**, not guild-specific.  
- Developer commands are prefix-only. User commands are slash-only.

---

## Optional (Future Considerations)

- Item Collection Tracker: Track % of items obtained per crate.  
- Flex Titles: Unlockable user tags for status display.  
- Item Skins: Cosmetic-only item variants.  
- Booster system: Temporary buffs purchasable with Gems.  
- Referral system: Gems awarded for bringing new users (requires abuse control).

---

## Modular Development Plan

- CommandHandler  
- Database (SQLite)  
- UserManager  
- InventoryManager  
- CrateManager  
- PrestigeManager  
- UpgradeManager  
- TradeManager  
- StatsManager  
- BlacklistManager  
- EventManager  
- Logger (Webhook + file)  
- Utils (number formatting, cooldowns, etc.)

---

## Final Notes

The bot is designed for infinite progression, competitive grinding, and strategic resource use.  
No bloated features. Every system supports retention and replayability.
