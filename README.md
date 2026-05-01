# CustomEvents — Config Reference

Everything you can do lives in `config.yml`. No code changes needed.  
This file is the single source of truth for all fields, values, and placeholders.

---

## Table of Contents

1. [Global Settings](#1-global-settings)
2. [Drop Tables](#2-drop-tables)
3. [Events](#3-events)
   - [Top-Level Event Fields](#31-top-level-event-fields)
   - [Triggers](#32-triggers)
   - [Duration Format](#33-duration-format)
   - [Conditions](#34-conditions)
   - [Rewards](#35-rewards)
   - [Announcements](#36-announcements)
   - [Scoring & Leaderboard (Individual)](#37-scoring--leaderboard-individual-events)
   - [Target, Milestones & Auto-Multiplier (Global)](#38-target-milestones--auto-multiplier-global-events)
   - [Scheduling](#39-scheduling)
   - [Event Chaining](#310-event-chaining)
   - [Participation Cooldown](#311-participation-cooldown)
4. [Placeholders](#4-placeholders)
5. [Persistence & Crash Safety](#5-persistence--crash-safety)
6. [Admin Commands](#6-admin-commands)
7. [Permissions](#7-permissions)
8. [Discord Webhook](#8-discord-webhook)

---

## 1. Global Settings

```yml
settings:
  glow_enabled: false
  discord_webhook: "https://discord.com/api/webhooks/YOUR_ID/YOUR_TOKEN"
```

| Field | Type | Default | Description |
|---|---|---|---|
| `glow_enabled` | boolean | `false` | Adds a hidden Unbreaking enchant to reward items so they visually shimmer. **Keep `false` if you run an anti-illegal-items plugin** — it will delete those items on sight. |
| `discord_webhook` | string | none | Global Discord webhook URL. When set, the plugin posts an embed to this channel on every event start, end, or goal completion. Can be overridden per event. See [Discord Webhook](#8-discord-webhook). |

---

## 2. Drop Tables

Reusable lists of items that any event can reference by name via `rewards.drop_table`.

```yml
drop_tables:
  my_table_name:
    - item: diamond
      amount: 1
      chance: 5
    - item: gold_ingot
      amount: "2-5"
      chance: 40
```

Each entry uses the same fields as a reward drop item — see [Item Reward Fields](#item-reward-fields).

---

## 3. Events

All events go under the `events:` key. The event name is the key you use in commands.

```yml
events:
  my_event_name:
    type: individual
    trigger: block_break
    duration: 30min
    conditions: ...
    rewards: ...
    announcements: ...
```

---

### 3.1 Top-Level Event Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | string | yes | `individual` or `global_target` |
| `trigger` | string | yes | What gameplay action fires this event. See [Triggers](#32-triggers). |
| `duration` | duration | yes | How long the event runs. See [Duration Format](#33-duration-format). |
| `conditions` | section | no | Filters that must pass before a player earns a reward. |
| `rewards` | section | no | What the player receives when the trigger fires. |
| `announcements` | section | no | Messages, titles, sounds, and bossbars. |
| `scoring` | section | no | Enables per-player score tracking (individual events only). |
| `leaderboard` | section | no | End-of-event rank prizes (individual events only). |
| `target` | section | no | Goal and count-per-break (global events only). |
| `milestones` | section | no | Rewards at specific count thresholds (global events only). |
| `auto_multiplier` | list | no | Automatic multiplier boosts as time runs low (global events only). |
| `completion_rewards` | section | no | Reward given to all players when goal is reached (global events only). |
| `schedule` | section | no | Auto start/stop times. |
| `on_end.start_event` | string | no | Name of event to auto-start when this one ends (chaining). |
| `cooldown_between_runs` | duration | none | Minimum time that must pass after this event ends before it can start again. Applies to both manual `/event start` and the scheduler. See [Participation Cooldown](#311-participation-cooldown). |
| `discord_webhook` | string | none | Per-event Discord webhook URL. Overrides `settings.discord_webhook` for this event only. |

---

### 3.2 Triggers

The `trigger` field controls which in-game action activates the event for a player.

| Value | Fires when… |
|---|---|
| `block_break` | A player breaks a block |
| `block_place` | A player places a block |
| `entity_kill` | A player kills any entity |
| `player_fish` | A player successfully catches a fish |
| `item_craft` | A player crafts an item |
| `item_smelt` | A player extracts a smelted item from a furnace |
| `player_damage` | A player deals melee damage to an entity |
| `timed` | *(Scheduled start/stop only — no per-player trigger)* |

---

### 3.3 Duration Format

Used in `duration`, `cooldown`, `every`, and multiplier `time_left` fields.

| Suffix | Example | Meaning |
|---|---|---|
| `s` | `10s` | 10 seconds |
| `min` or `m` | `30min` / `30m` | 30 minutes |
| `h` | `2h` | 2 hours |
| `ms` | `500ms` | 500 milliseconds |

---

### 3.4 Conditions

All condition fields are optional. Omitting a field means "no restriction".

```yml
conditions:
  blocks: [gold_ore, deepslate_gold_ore]
  worlds: [world, world_nether]
  biomes: [badlands, desert]
  gamemodes: [survival]
  y_level: "below:60"
  time: "13000-23000"
  held_item: [diamond_pickaxe]
  permission: "events.participate"
  cooldown: 10s
  max_triggers: 1000
  max_triggers_per_player: 50
  natural_blocks_only: true
```

| Field | Type | Description |
|---|---|---|
| `blocks` | string list | Block or entity types that must match. Accepts Material names (`gold_ore`) or the wildcard `any_ore`. |
| `worlds` | string list | World names the player must be in. |
| `biomes` | string list | Biome names the block must be in. Use Bukkit biome names, e.g. `badlands`, `jagged_peaks`. |
| `gamemodes` | string list | Player's gamemode. Values: `survival`, `creative`, `adventure`, `spectator`. |
| `y_level` | string | Y-coordinate rule. See [Y-Level Format](#y-level-format). |
| `time` | string | World time range in ticks. Format: `"start-end"`. See [Time Range](#time-range). |
| `held_item` | string list | Material name of the item the player must be holding in their main hand. |
| `permission` | string | Permission node the player must have to earn rewards. |
| `cooldown` | duration | Per-player cooldown between trigger rewards. |
| `max_triggers` | integer | Server-wide total trigger cap. Event stops awarding rewards after this. |
| `max_triggers_per_player` | integer | Per-player cap on how many times they can be rewarded. |
| `natural_blocks_only` | boolean | If `true`, blocks placed by players are flagged and will **not** earn rewards (anti-abuse). |

#### Block Wildcards

| Wildcard | Expands to |
|---|---|
| `any_ore` | All ore types: coal, iron, copper, gold, redstone, emerald, lapis, diamond, nether gold, nether quartz, ancient debris (including deepslate variants) |

#### Y-Level Format

| Format | Example | Meaning |
|---|---|---|
| `below:N` | `below:60` | Y < 60 |
| `above:N` | `above:100` | Y > 100 |
| `min-max` | `10-50` | 10 ≤ Y ≤ 50 |
| Exact | `64` | Y == 64 |

#### Time Range

Minecraft world time is in ticks (0–24000).

| Ticks | In-game time |
|---|---|
| `0` | 6:00 AM (sunrise) |
| `6000` | 12:00 PM (noon) |
| `12000` | 6:00 PM (sunset) |
| `13000` | Night begins |
| `18000` | Midnight |
| `23000` | Just before dawn |

Example — night only: `time: "13000-23000"`

---

### 3.5 Rewards

```yml
rewards:
  cancel_natural_drops: false
  drops:
    - item: diamond
      amount: "1-3"
      chance: 15
      custom_name: "&bEvent Diamond"
      lore:
        - "&7Found during an event!"
      enchants:
        - sharpness:3
      glow: true
  give:
    - item: gold_ingot
      amount: 5
      chance: 100
  commands:
    - console: "eco give {player} 500"
    - player: "kit claim event"
  xp:
    amount: "5-20"
    chance: 60
  effects:
    - type: speed
      duration: 10s
      level: 2
  drop_table: my_table_name
```

| Field | Type | Description |
|---|---|---|
| `cancel_natural_drops` | boolean | If `true`, suppresses the block's normal item drops. Default: `false`. |
| `drops` | item list | Items spawned at the player's feet as a dropped item entity. |
| `give` | item list | Items added directly to the player's inventory (no floor drop). Overflow drops at feet. |
| `commands` | list | Console or player commands to run on reward. |
| `xp` | section | Experience points to give. |
| `effects` | list | Potion effects to apply to the player. |
| `drop_table` | string | Name of a reusable drop table defined under `drop_tables:`. Merged with `drops`. |

#### Item Reward Fields

Applies to entries in `drops`, `give`, and `drop_tables`.

| Field | Type | Default | Description |
|---|---|---|---|
| `item` | string | required | Bukkit Material name (e.g. `diamond`, `iron_ingot`, `oak_log`). Case-insensitive. |
| `amount` | string or int | `1` | Fixed (`5`) or random range (`"2-8"`). |
| `chance` | integer | `100` | Percentage chance (1–100) this item is included in the reward. |
| `custom_name` | string | none | Display name with `&` color codes. |
| `lore` | string list | none | Lore lines with `&` color codes. |
| `enchants` | string list | none | Format: `enchantment_name:level` (e.g. `sharpness:3`, `fortune:2`). |
| `glow` | boolean | `false` | Visual glow effect. **Only works if `settings.glow_enabled: true`.** Ignored otherwise. |

#### Commands

```yml
commands:
  - console: "eco give {player} 500"   # run by the server console
  - player: "kit claim event"           # run as the player
```

Supports all [placeholders](#4-placeholders).

#### XP

```yml
xp:
  amount: "5-20"   # fixed or range
  chance: 60       # percentage
```

#### Potion Effects

```yml
effects:
  - type: speed         # Bukkit PotionEffectType name
    duration: 10s       # duration format
    level: 2            # amplifier level (1 = level I)
```

Common effect types: `speed`, `strength`, `regeneration`, `resistance`, `haste`, `jump_boost`, `saturation`, `absorption`, `night_vision`, `invisibility`, `poison`, `slowness`, `weakness`, `blindness`.

---

### 3.6 Announcements

```yml
announcements:
  on_start:
    broadcast: "&6Gold Rush started!"
    title: "&6GOLD RUSH"
    subtitle: "&eMine ores!"
    sound: "ui.toast.challenge_complete"
    bossbar:
      text: "&6⛏ Gold Rush | Rank #{rank} | {score} pts | {time_left} left"
      color: YELLOW
      style: SOLID
  on_trigger:
    actionbar: "&6+{reward_amount}x Gold!"
    sound: "entity.experience_orb.pickup"
  on_end:
    broadcast: "&cGold Rush ended! Top: {top_player} — {top_score}"
    title: "&cEVENT OVER"
    subtitle: "&7Gold Rush has ended"
    sound: "entity.wither.death"
  on_end_incomplete:          # global events only
    broadcast: "&7Time's up! Reached {current}/6,000."
  interval_broadcast:
    every: 5min
    message: "&6[Gold Rush] Active! {time_left} left!"
```

#### on_start

| Field | Type | Description |
|---|---|---|
| `broadcast` | string | Chat message sent to all players. Supports `&` colors and [placeholders](#4-placeholders). |
| `title` | string | Large title shown on screen. |
| `subtitle` | string | Smaller subtitle shown below the title. |
| `sound` | string | Sound key played to all players. See [Sound Names](#sound-names). |
| `bossbar` | section | Persistent bar shown while the event is running. |

#### Bossbar

| Field | Options | Description |
|---|---|---|
| `text` | string | Text on the bar. Supports colors and placeholders including `{rank}` and `{score}`. |
| `color` | `PINK` `BLUE` `RED` `GREEN` `YELLOW` `PURPLE` `WHITE` | Bar color. |
| `style` | `SOLID` `SEGMENTED_6` `SEGMENTED_10` `SEGMENTED_12` `SEGMENTED_20` | Bar segment style. |

> **Individual events** — each player gets their own bossbar showing their personal `{rank}` and `{score}`. Updates every second.  
> **Global events** — one shared bossbar for all players showing `{current}`, `{goal}`, `{percent}`.

#### on_trigger

Fires every time a player earns a reward.

| Field | Type | Description |
|---|---|---|
| `actionbar` | string | Text shown in the player's action bar (above hotbar). |
| `sound` | string | *(Legacy)* Single sound played to the triggering player. Still works. |
| `sounds` | list | Layered sound sequence played to the triggering player. Replaces `sound`. |

**Layered sounds** let you stack multiple sounds with individual `volume`, `pitch`, and `delay` to create a cascade that feels far more satisfying than a single note.

```yml
on_trigger:
  actionbar: "&6+{reward_amount}x Gold!"
  sounds:
    - sound: "block.note_block.bell"
      volume: 1.0
      pitch: 1.3
    - sound: "entity.experience_orb.pickup"
      volume: 0.9
      pitch: 1.8
      delay: 80ms
    - sound: "block.note_block.harp"
      volume: 0.7
      pitch: 1.5
      delay: 160ms
```

| Field | Type | Default | Description |
|---|---|---|---|
| `sound` | string | required | Minecraft sound key (e.g. `block.note_block.bell`). |
| `volume` | float | `1.0` | Loudness. `1.0` = full volume. Values above `1.0` increase range, not loudness. |
| `pitch` | float | `1.0` | Speed/pitch of the sound. Range: `0.5` (low) → `2.0` (high). |
| `delay` | duration | none | How long after the trigger to play this sound. Use `ms` for precision (e.g. `80ms`, `160ms`). |

**Recommended pitch ranges by feel:**

| Range | Feel |
|---|---|
| `0.5 – 0.8` | Deep, heavy, impactful |
| `0.9 – 1.2` | Natural, grounded |
| `1.3 – 1.6` | Bright, crisp, satisfying |
| `1.7 – 2.0` | High, sparkly, celebratory |

**Recommended sounds for trigger effects:**

| Sound key | Character |
|---|---|
| `block.note_block.bell` | Clear bell — warm and satisfying |
| `block.note_block.harp` | Soft pluck — gentle resolution |
| `block.note_block.pling` | Electronic ping — energetic |
| `block.note_block.bass` | Deep thump — punchy impact |
| `block.amethyst_cluster.break` | Crystal crack — bright and crisp |
| `entity.experience_orb.pickup` | XP sparkle — reward feel |
| `entity.player.attack.crit` | Crit swoosh — combat punch |
| `entity.fishing_bobber.retrieve` | Reel — satisfying catch feel |
| `entity.player.levelup` | Level-up sweep — triumphant |

#### on_end

| Field | Type | Description |
|---|---|---|
| `broadcast` | string | Chat message to all players. |
| `title` | string | Title on screen. |
| `subtitle` | string | Subtitle on screen. |
| `sound` | string | Sound to all players. |

#### on_end_incomplete *(global events only)*

Fires when the event timer expires without reaching the goal.

| Field | Type | Description |
|---|---|---|
| `broadcast` | string | Chat message explaining the goal was not reached. |

#### on_player_join

Sent privately to a player who joins the server while the event is already running — so they immediately know what's happening and can participate.

```yml
announcements:
  on_player_join: "&6[Gold Rush] An event is running! Mine gold ores — Rank #{rank} with {score} pts. {time_left} left!"
```

| Field | Type | Description |
|---|---|---|
| `on_player_join` | string | Private message to the joining player. Supports `{rank}`, `{score}`, `{time_left}`, and all other placeholders. |

> The message is sent only once, right after the player finishes loading in. If the event ended before they fully joined they won't receive it.

#### interval_broadcast

Repeating reminder message while the event is active.

| Field | Type | Description |
|---|---|---|
| `every` | duration | How often to send the message. |
| `message` | string | Chat message. Supports placeholders. |

#### Sound Names

Use Minecraft sound keys exactly as they appear in-game. Examples:

| Key | Description |
|---|---|
| `ui.toast.challenge_complete` | Advancement toast sound |
| `entity.experience_orb.pickup` | XP pickup |
| `entity.player.levelup` | Level-up sound |
| `block.beacon.activate` | Beacon activation |
| `entity.wither.spawn` | Wither spawn |
| `entity.wither.death` | Wither death |
| `entity.villager.celebrate` | Villager celebrating |
| `entity.fishing_bobber.splash` | Fishing bobber splash |
| `entity.fishing_bobber.retrieve` | Fishing reel-in |
| `block.note_block.pling` | Note block pling |
| `ui.toast.challenge_complete` | Challenge complete toast |

Full list: [minecraft.wiki/w/Sounds.json](https://minecraft.wiki/w/Sounds.json)

---

### 3.6b Per-Type Rewards (`rewards_by_type`)

When each target type (block, mob, fish catch, crafted item) should award **different points and different loot**, use `rewards_by_type`. Works with every trigger — the key format just changes depending on the trigger type.

```yml
rewards_by_type:
  diamond_ore:        # exact Bukkit Material or EntityType name, lower-case
    points: 10
    rewards:
      give:
        - item: diamond
          amount: 1
          chance: 100
      commands:
        - console: "eco give {player} 500"

  coal_ore:
    points: 1
    rewards:
      give:
        - item: coal
          amount: "1-3"
          chance: 100

  _default:           # catch-all for anything not explicitly listed
    points: 1
    rewards:
      give:
        - item: iron_nugget
          amount: 1
          chance: 100
```

| Field | Description |
|---|---|
| Key | The name to match against — see the table below for what each trigger uses. Case-insensitive. |
| `any_ore` | Wildcard matching all ore types. Only for `block_break` and `block_place` triggers. |
| `_default` | Fallback when no exact key matches. |
| `points` | Points added to the player's leaderboard score for this specific type. Default: `1`. |
| `rewards` | Full reward section — same fields as the global `rewards`. Fires **in addition** to the top-level `rewards`. |

**What the key resolves to per trigger:**

| Trigger | Key is… | Example keys |
|---|---|---|
| `block_break` | Bukkit Material name of the broken block | `diamond_ore`, `deepslate_iron_ore`, `ancient_debris` |
| `block_place` | Bukkit Material name of the placed block | `tnt`, `chest`, `oak_log` |
| `entity_kill` | Bukkit EntityType name of the killed entity | `zombie`, `skeleton`, `wither`, `ender_dragon` |
| `player_damage` | Bukkit EntityType name of the damaged entity | `creeper`, `blaze`, `player` |
| `player_fish` | Bukkit Material name of the item caught | `cod`, `salmon`, `tropical_fish`, `pufferfish`, `enchanted_book`, `saddle` |
| `item_craft` | Bukkit Material name of the crafted item | `diamond_sword`, `iron_pickaxe`, `cake` |
| `item_smelt` | Bukkit Material name of the smelted output | `iron_ingot`, `gold_ingot`, `cooked_beef` |

**How global `rewards` and `rewards_by_type` combine:**  
Top-level `rewards` (XP, effects, global commands) fires for **every** successful trigger. The matching `rewards_by_type` entry fires on **top** of that with the type-specific loot. This lets you give base XP to every kill while giving mob-specific drops separately.

**Filtering behaviour:**  
When `rewards_by_type` is defined, conditions filtering by block/entity type uses the keys of `rewards_by_type` instead of `conditions.blocks`. A trigger is rejected if no key matches (and no `_default` exists).

**Example point values** (Over Miner defaults — change freely):

| Ore | Points |
|---|---|
| `ancient_debris` | 15 |
| `diamond_ore` / `deepslate_diamond_ore` | 10 |
| `emerald_ore` / `deepslate_emerald_ore` | 7 |
| `gold_ore` / `deepslate_gold_ore` | 5 |
| `nether_gold_ore` | 4 |
| `lapis_ore` / `deepslate_lapis_ore` | 3 |
| `iron_ore` / `deepslate_iron_ore` | 2 |
| `redstone_ore` / `deepslate_redstone_ore` | 2 |
| `nether_quartz_ore` | 2 |
| `copper_ore` / `deepslate_copper_ore` | 1 |
| `coal_ore` / `deepslate_coal_ore` | 1 |

> These are just the defaults in the example config. Change `points` to whatever fits your server's economy.

---

### 3.7 Scoring & Leaderboard (Individual Events)

```yml
scoring:
  enabled: true
  metric: triggers     # what counts as a point

leaderboard:
  top_size: 3
  end_rewards:
    "1":
      commands:
        - console: "eco give {player} 10000"
        - console: "broadcast &6{player} won with {score}!"
    "2":
      commands:
        - console: "eco give {player} 5000"
    "3":
      commands:
        - console: "eco give {player} 2500"
```

#### Scoring

| Field | Type | Description |
|---|---|---|
| `enabled` | boolean | Enables score tracking. Required for leaderboard rewards to work. |
| `metric` | string | What the score represents. See table below. |

| Metric | Counts |
|---|---|
| `triggers` | Each successful reward trigger = 1 point |
| `items_collected` | Number of items received (uses reward amount) |
| `mobs_killed` | Each entity kill = 1 point |
| `damage_dealt` | Damage dealt per hit |

#### Leaderboard

| Field | Type | Description |
|---|---|---|
| `top_size` | integer | How many top players are tracked and rewarded. |
| `end_rewards` | map | Keys are rank numbers (`"1"`, `"2"`, `"3"`). Each supports `commands`, `drops`, `give`, `xp`, `effects`. |

#### Offline Player Rewards

Players who disconnect before the event ends **still receive their leaderboard rank**. The plugin caches every participant's name during the event, so when the event finishes:

- **Online players** — receive full rewards (items, drops, XP, effects, commands).
- **Offline players** — receive **console commands only**. Item drops and effects cannot be applied to offline players, but economy commands (e.g. `eco give {player} 10000`) work fine because most economy plugins accept offline player names.

A log entry is written to console for every offline reward:
```
[CustomEvents] Offline leaderboard reward for Steve (rank #1, score 47) — commands only.
```

> **Tip:** Use economy commands for rank 1 prizes — they're offline-safe. Save physical items for participation drops that fire when players are actively playing.

---

### 3.8 Target, Milestones & Auto-Multiplier (Global Events)

#### Target

```yml
target:
  goal: 6000          # total count needed to complete the event
  count_per_break: 1  # how much each trigger adds to the global counter
```

#### Milestones

Fires once when the global counter crosses a threshold. Keys are the threshold numbers.

```yml
milestones:
  1000:
    broadcast: "&e1,000 reached!"
    sound: "ui.toast.challenge_complete"
    commands:
      - console: "eco give {player} 500"
  3000:
    broadcast: "&eHalfway!"
```

| Field | Type | Description |
|---|---|---|
| `broadcast` | string | Chat message to all players. |
| `sound` | string | Sound to all players. |
| `commands` | list | Console commands. `{player}` resolves to the player who triggered the milestone. |

#### Auto-Multiplier

Automatically boosts the `count_per_break` as the clock runs down.

```yml
auto_multiplier:
  - time_left: "30min"
    multiplier: 2
    announce: "&e2x MULTIPLIER ACTIVE!"
  - time_left: "10min"
    multiplier: 5
    announce: "&65x — FINAL PUSH!"
  - time_left: "5min"
    multiplier: 10
    announce: "&410x — LAST 5 MINUTES!"
```

| Field | Type | Description |
|---|---|---|
| `time_left` | duration | When remaining time drops to this value, the multiplier fires. |
| `multiplier` | integer | Replaces the current multiplier. Applied to `count_per_break`. |
| `announce` | string | Broadcast to all players when the multiplier kicks in. |

#### Completion Rewards

Runs when the global goal is reached. Given to **all online players**.

```yml
completion_rewards:
  broadcast: "&l[COMPLETE!] Goal reached!"
  title: "&lGOAL REACHED!"
  subtitle: "&fAll players rewarded!"
  commands:
    - console: "eco give {player} 2000"
    - console: "coins give {player} 500"
```

Supports all reward fields: `commands`, `drops`, `give`, `xp`, `effects`.

---

### 3.9 Scheduling

Events start and stop automatically based on the schedule. The scheduler polls every 30 seconds.

#### Timezone

```yml
schedule:
  timezone: "Asia/Karachi"   # IANA timezone name
```

Uses any valid [IANA timezone ID](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).  
Examples: `Asia/Karachi`, `America/New_York`, `Europe/London`, `UTC`.

#### Recurring Weekly

Repeats every week on the specified days and times.

```yml
schedule:
  timezone: "Asia/Karachi"
  recurring:
    - day: "SATURDAY"
      start: "20:00"
      duration: 2h
    - day: "SUNDAY"
      start: "15:00"
      duration: 2h
```

| Field | Values | Description |
|---|---|---|
| `day` | `MONDAY` `TUESDAY` `WEDNESDAY` `THURSDAY` `FRIDAY` `SATURDAY` `SUNDAY` | Day of the week. |
| `start` | `HH:mm` | 24-hour start time in the configured timezone. |
| `duration` | duration | How long the event runs. |

#### Fixed One-Time Window

```yml
schedule:
  timezone: "Asia/Karachi"
  start: "2026-05-03 20:00"
  end:   "2026-05-03 22:00"
```

#### Multiple Fixed Windows

Counter persists across windows when `persist_count: true`.

```yml
schedule:
  timezone: "Asia/Karachi"
  persist_count: true
  windows:
    - start: "2026-05-03 20:00"
      end:   "2026-05-03 22:00"
    - start: "2026-05-04 18:00"
      end:   "2026-05-04 20:00"
```

---

### 3.10 Event Chaining

Automatically starts another event when this one ends.

```yml
on_end:
  start_event: ore_frenzy_phase2
```

Use this to build multi-phase events. The chained event starts within ~1 second of the first ending.

---

### 3.11 Participation Cooldown

Prevents an event from restarting too soon after it ends — whether by the scheduler or an admin.

```yml
  gold_rush:
    cooldown_between_runs: 1h    # can't restart for 1 hour after it ends
```

| Field | Type | Default | Description |
|---|---|---|---|
| `cooldown_between_runs` | duration | none | Minimum time between the end of one run and the start of the next. Uses standard [duration format](#33-duration-format). |

**Behaviour:**

- If an admin runs `/event start <name>` while on cooldown, they receive: `"Event 'gold_rush' is on cooldown: 58m 12s remaining."` — and the event does not start.
- The scheduler silently skips the event until the cooldown clears.
- The cooldown starts from the moment the event ends (any reason: completed, time expired, or manual stop).
- To disable, remove the field or set `cooldown_between_runs: 0s`.

**Global events + early completion:**  
If a global event finishes its goal before the time window closes, the scheduler will **not** restart it within the same window — even without a cooldown set. This is tracked automatically per schedule window. The `cooldown_between_runs` field adds an additional time-based lock on top of this.

---

## 4. Placeholders

Available in all text fields: `broadcast`, `title`, `subtitle`, `actionbar`, `bossbar.text`, `commands`, `interval_broadcast.message`, `announce`, `milestone.broadcast`.

| Placeholder | Resolves to | Where |
|---|---|---|
| `{player}` | Triggering player's name | Per-player contexts |
| `{score}` | Player's current score | Individual events |
| `{rank}` | Player's current leaderboard rank (1 = first) | Individual events, bossbar |
| `{top_player}` | Name of the player in 1st place | End announcements |
| `{top_score}` | Score of the player in 1st place | End announcements |
| `{reward_amount}` | Amount from the last reward given | `on_trigger` only |
| `{current}` | Current global counter | Global events |
| `{goal}` | Target goal number | Global events |
| `{percent}` | `{current} / {goal}` as a percentage (one decimal) | Global events |
| `{multiplier}` | Current active multiplier | Global events |
| `{time_left}` | Human-readable time remaining (`12m 30s`) | All |
| `{trigger_count}` | Total server-wide trigger count | All |
| `{next_start}` | Next scheduled start time | `on_end_incomplete` |

#### Text Formatting

All text fields support two formatting systems — you can mix them freely within a single config.

**Legacy `&` codes** — simple and familiar:
```
&6Gold Rush &ehas &lstarted!
```

**MiniMessage** — rich gradient, hover, click, and more. Use `<` tags:
```
<gold><bold>Gold Rush</bold></gold> <yellow>has started!</yellow>
<gradient:gold:yellow>Mine ores for rewards!</gradient>
```

The plugin auto-detects which format you're using: if the text contains `<`, it's parsed as MiniMessage; otherwise `&` codes are used. You can mix both systems across different fields, but not within a single field value.

**Common MiniMessage tags:**

| Tag | Effect |
|---|---|
| `<gold>` | Gold color (works for all named colors) |
| `<#FF5733>` | Hex color |
| `<bold>` / `<b>` | Bold |
| `<italic>` / `<i>` | Italic |
| `<underlined>` | Underline |
| `<gradient:color1:color2>` | Gradient between two colors |
| `<rainbow>` | Rainbow gradient |
| `<reset>` | Reset all formatting |

Full MiniMessage reference: [docs.advntr.dev/minimessage](https://docs.advntr.dev/minimessage/format.html)

#### Color Codes

Use `&` followed by a code anywhere text is accepted.

| Code | Color | Code | Format |
|---|---|---|---|
| `&0` | Black | `&l` | **Bold** |
| `&1` | Dark Blue | `&o` | *Italic* |
| `&2` | Dark Green | `&n` | Underline |
| `&3` | Dark Aqua | `&m` | Strikethrough |
| `&4` | Dark Red | `&k` | Obfuscated |
| `&5` | Dark Purple | `&r` | Reset |
| `&6` | Gold | | |
| `&7` | Gray | | |
| `&8` | Dark Gray | | |
| `&9` | Blue | | |
| `&a` | Green | | |
| `&b` | Aqua | | |
| `&c` | Red | | |
| `&d` | Light Purple | | |
| `&e` | Yellow | | |
| `&f` | White | | |

---

## 5. Persistence & Crash Safety

CustomEvents uses a single **SQLite database** (`plugins/CustomEvents/custom_events.db`) as its persistence layer. No external database server needed — it's embedded in the jar.

### What is stored

| Data | Table | Notes |
|---|---|---|
| Running event state (scores, counters, multipliers) | `active_states` | Auto-saved every 30 s |
| Player name cache | `player_names` | Used for offline reward commands |
| Event run history | `event_history` | Permanent log — viewable via `/event history` |
| Cooldown timestamps | `event_meta` | `last_ended_at` per event |
| Early-completion window tracking | `event_meta` | Prevents global events restarting in same window |
| Per-player cooldowns | *(memory only)* | Resets on restart |

### How crash recovery works

- **Auto-save** — every 30 seconds, all running events write their full state (scores, global counter, multiplier, milestones, player names) to the database. Worst-case data loss on a crash: 30 seconds.
- **Clean shutdown** — on `/stop` or `/reload`, a final save runs before bossbars and tasks are torn down. Zero data lost on a clean stop.
- **Restore on startup** — the plugin reads `active_states` on enable and recreates all events that haven't expired. Bossbars reappear when players rejoin. Events that expired while the server was offline are discarded automatically.

### Database file

`plugins/CustomEvents/custom_events.db` — a standard SQLite file. You can open it with [DB Browser for SQLite](https://sqlitebrowser.org/) to inspect or clear history records. Do not edit `active_states` while the server is running.

---

## 6. Admin Commands

All commands require the `customevents.admin` permission.

| Command | Description |
|---|---|
| `/event start <name>` | Manually start an event |
| `/event stop <name>` | Manually stop a running event |
| `/event list` | List all events and their running status |
| `/event info <name>` | Show live stats for a running event (time left, count, multiplier) |
| `/event reload` | Stop all events and reload `config.yml` |
| `/event multiplier <name> <value>` | Override the multiplier on a global event live |
| `/event setcount <name> <value>` | Set the global counter to an exact number |
| `/event addcount <name> <value>` | Add to the global counter |
| `/event subcount <name> <value>` | Subtract from the global counter |
| `/event leaderboard <name>` | Show the current top players for an individual event |
| `/event reset <name>` | Reset all scores, counts, cooldowns, and multiplier for a running event |
| `/event simulate <name> <player>` | Fire a test trigger for a player (useful for testing reward configs) |
| `/event history [name] [page]` | View past event runs. Omit `name` for all events. Page defaults to 1, 5 entries per page. |

#### History output example

```
=== Event History (page 1) ===
#12  gold_rush    — ✓ completed   — May 01 20:47  [scheduled]
     Participants: 8   Triggers: 312
     Top: #1 Steve (47), #2 Alex (32), #3 Bob (18)
#11  redstone_rally — ⏰ time expired — May 01 15:02  [scheduled]
     Participants: 5   Triggers: 4821
#10  diamond_rush — ✗ stopped     — Apr 30 12:00  [manual]
     Participants: 1   Triggers: 7
```

| Column | Meaning |
|---|---|
| `✓ completed` | Global event reached its goal before time ran out |
| `⏰ time expired` | Timer ran out (goal not reached, or individual event ended naturally) |
| `✗ stopped` | Admin called `/event stop` |
| `[scheduled]` / `[manual]` | Whether the event was started by the scheduler or by a command |

---

## 7. Permissions

| Node | Default | Description |
|---|---|---|
| `customevents.admin` | OP | Access to all `/event` commands |
| `customevents.participate` | Everyone | Required to receive rewards when `conditions.permission` is set to this node |

> To restrict participation in a specific event, set `conditions.permission: customevents.participate` and revoke the node from players who should be excluded, or use a custom permission node like `events.vip` and grant it only to certain groups.

---

## 8. Discord Webhook

Automatically posts embed messages to a Discord channel when events start, end, or reach their goal.

### Setup

**Option A — Global (all events post to the same channel):**
```yml
settings:
  discord_webhook: "https://discord.com/api/webhooks/1234567890/abcdef..."
```

**Option B — Per event (each event has its own channel):**
```yml
events:
  gold_rush:
    discord_webhook: "https://discord.com/api/webhooks/..."   # overrides global

  redstone_rally:
    discord_webhook: "https://discord.com/api/webhooks/..."   # different channel
```

Per-event webhook takes priority over the global one. If neither is set, Discord notifications are disabled.

### How to get a webhook URL

1. Open your Discord server → **Server Settings → Integrations → Webhooks**.
2. Click **New Webhook**, choose the channel, copy the URL.
3. Paste it into `config.yml`.

### What gets posted

| Event | Title | Color | Fields |
|---|---|---|---|
| Event starts | `🎉 gold_rush — Started` | Gold | Type, Trigger, Duration |
| Event ends (completed goal) | `✅ gold_rush — Completed!` | Green | Top Player, Participants |
| Event ends (time expired) | `⏰ gold_rush — Time's Up` | Grey | Top Player, Participants |
| Event manually stopped | `🛑 gold_rush — Stopped` | Red | Top Player, Participants |

### Example embed

```
🎉 gold_rush — Started
─────────────────────────────
Type          Trigger       Duration
Individual    block_break   30m
─────────────────────────────
CustomEvents
```

### Notes

- Posts are sent **asynchronously** — they never lag the server.
- If Discord is down or the URL is wrong, a `WARNING` is logged but the plugin continues normally.
- Delivery is **fire-and-forget** — the plugin does not retry failed posts.
- The top player shown at event end includes **offline players** — names are resolved from the in-event cache.
