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
4. [Placeholders](#4-placeholders)
5. [Admin Commands](#5-admin-commands)
6. [Permissions](#6-permissions)

---

## 1. Global Settings

```yml
settings:
  glow_enabled: false
```

| Field | Type | Default | Description |
|---|---|---|---|
| `glow_enabled` | boolean | `false` | Adds a hidden Unbreaking enchant to reward items so they visually shimmer. **Keep `false` if you run an anti-illegal-items plugin** — it will delete those items on sight. |

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

## 5. Admin Commands

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

---

## 6. Permissions

| Node | Default | Description |
|---|---|---|
| `customevents.admin` | OP | Access to all `/event` commands |
| `customevents.participate` | Everyone | Required to receive rewards when `conditions.permission` is set to this node |

> To restrict participation in a specific event, set `conditions.permission: customevents.participate` and revoke the node from players who should be excluded, or use a custom permission node like `events.vip` and grant it only to certain groups.
