# 01 — Overview and Concepts

## What Is Archipelago?

Archipelago (AP) is an open-source **multiworld randomizer** framework. It lets players of
different games share a single randomized item pool so that finding an item in one game can
unlock progress in another player's game — even if they are playing a completely different
title.

## The Two Halves of an Integration

Every game integration consists of two cooperating pieces:

### 1. The World (server-side / generation-side)

A **Python package** that lives inside `worlds/your_game/`. It tells the Archipelago
generator:

- What **items** exist in your game (swords, keys, upgrades, etc.).
- What **locations** can hold items (chests, boss drops, quest rewards, etc.).
- What **regions** the game world is divided into and how they connect.
- What **rules** govern access (e.g., "the player needs the Hookshot to reach the Lake").
- What **options** the player can tweak (difficulty, goal type, etc.).
- What the **completion condition** (goal) is.

### 2. The Client (game-side)

A program (or game mod) that:

- Connects to the Archipelago server over a **WebSocket**.
- **Sends** location-check messages when the player picks up an item in-game.
- **Receives** item-grant messages when another player sends an item.
- Tracks a synchronization index so items are never lost or duplicated.
- Sends a **goal-complete** status update when the player finishes.

The client can be written in any language, but if you write it in Python you can integrate
it directly into the Archipelago Launcher.

## Key Terminology

| Term | Meaning |
|------|---------|
| **MultiWorld** | The combined state of all players' worlds during generation. |
| **World** | Your game-specific Python class that plugs into the generator. |
| **Region** | A logical container of locations that share access rules. |
| **Location** | A place where an item can be placed (chest, NPC, drop, etc.). |
| **Item** | Something the player can receive (weapon, key, ability, etc.). |
| **Entrance** | A one-way connection from one region to another, possibly gated by rules. |
| **Event** | A generation-only item/location pair used to model in-game triggers (e.g., flipping a switch). |
| **Slot** | A single player's position in a multiworld session. |
| **Slot Data** | Custom JSON data sent from the World to the client at connect time. |
| **APWorld / .apworld** | The packaged zip of your world folder for distribution. |
| **DataPackage** | The item/location ID mappings the server sends so all clients can decode each other's data. |
| **Filler** | Low-value items used to pad the item pool to match the location count. |
| **Progression** | An item that *may* be required to reach new locations. |
| **Trap** | An item with a negative effect on the receiving player. |

## How Generation Works (Simplified)

```
Player YAML configs
        │
        ▼
┌───────────────────┐
│   Generate.py     │  Reads all YAMLs, creates a MultiWorld
│                   │  Calls each World's hooks in order:
│  generate_early   │──▶  read options, set up state
│  create_regions   │──▶  build region/entrance graph
│  create_items     │──▶  populate item pool
│  set_rules        │──▶  attach access rules
│  connect_entrances│──▶  finalize entrances (ER if applicable)
│  generate_basic   │──▶  non-logic randomization
│  pre_fill / fill  │──▶  place items into locations
│  post_fill        │──▶  cleanup
│  generate_output  │──▶  write ROM patches / output files
└───────────────────┘
        │
        ▼
   .archipelago output file  →  uploaded to server  →  players connect
```

## How a Play Session Works

```
 Player A (Your Game)              Archipelago Server             Player B (Other Game)
 ───────────────────               ──────────────────             ────────────────────
        │                                  │                              │
   Connect via WS ──────────────▶  RoomInfo / Connected  ◀────── Connect via WS
        │                                  │                              │
   Check location  ─── LocationChecks ──▶  │                              │
        │                          Determine item owner                   │
        │                                  │── ReceivedItems ───────────▶ │
        │                                  │                       Grant item in-game
        │                                  │                              │
        │            ◀── ReceivedItems ────│   ◀── LocationChecks ────── Check location
   Grant item in-game                      │                              │
        │                                  │                              │
   Goal complete ─── StatusUpdate ───────▶ │                              │
```

## Next Step

Proceed to [02 - Prerequisites and Environment Setup](02_prerequisites_and_environment_setup.md)
to prepare your development environment.
