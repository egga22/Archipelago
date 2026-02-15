# 03 — Creating the World Package

This is the core of your integration. By the end of this chapter you will have a complete
(minimal) world package that the Archipelago generator can load.

## Folder Structure

Create the following files inside `worlds/your_game/`:

```
worlds/your_game/
├── __init__.py             # World class (required)
├── items.py                # Item definitions
├── locations.py            # Location definitions
├── regions.py              # Region and entrance definitions
├── rules.py                # Access rules
├── options.py              # Player-configurable options
├── archipelago.json        # APWorld metadata (optional but recommended)
├── docs/
│   ├── en_Your Game.md     # Game info page (required, at least one)
│   └── setup_en.md         # Setup/tutorial doc (required, at least one)
└── test/
    ├── __init__.py         # Required for test discovery
    └── test_access.py      # Your unit tests
```

> **Important:** Every subfolder containing `.py` files must have an `__init__.py` for
> frozen-build compatibility.

---

## Step 1 — Define Your Items (`items.py`)

```python
# worlds/your_game/items.py

from BaseClasses import Item, ItemClassification
from typing import NamedTuple, Optional


class YourGameItem(Item):
    """An item from Your Game."""
    game: str = "Your Game"


class ItemData(NamedTuple):
    code: Optional[int]
    classification: ItemClassification


# Choose a unique base ID. Pick a large number unlikely to collide.
# Check existing worlds to avoid overlaps.
BASE_ID = 0xYG0000  # replace with a real hex or decimal number

item_table: dict[str, ItemData] = {
    "Sword":          ItemData(BASE_ID + 0, ItemClassification.progression),
    "Shield":         ItemData(BASE_ID + 1, ItemClassification.useful),
    "Health Potion":  ItemData(BASE_ID + 2, ItemClassification.filler),
    "Boss Key":       ItemData(BASE_ID + 3, ItemClassification.progression),
    "Trap Bomb":      ItemData(BASE_ID + 4, ItemClassification.trap),
}

# Optional: group items for hint convenience
item_groups: dict[str, set[str]] = {
    "weapons": {"Sword"},
    "healing": {"Health Potion"},
}
```

### Item Classification Quick Reference

| Classification | Meaning |
|----------------|---------|
| `progression` | May be required to reach new locations. **If in doubt, use this.** |
| `useful` | Helpful but never logically required. Won't be placed on excluded locations. |
| `filler` | Junk / padding. |
| `trap` | Negative effect on the receiver. |
| `progression_skip_balancing` | Progression but should not be moved to earlier spheres (e.g., currency). |

---

## Step 2 — Define Your Locations (`locations.py`)

```python
# worlds/your_game/locations.py

from BaseClasses import Location
from typing import NamedTuple, Optional


class YourGameLocation(Location):
    """A location in Your Game."""
    game: str = "Your Game"


class LocationData(NamedTuple):
    code: Optional[int]
    region: str


BASE_ID = 0xYG0000  # use the same base as items, or a different one — IDs are independent

location_table: dict[str, LocationData] = {
    "Overworld - Chest 1":    LocationData(BASE_ID + 0, "Overworld"),
    "Overworld - Chest 2":    LocationData(BASE_ID + 1, "Overworld"),
    "Dungeon - Boss Drop":    LocationData(BASE_ID + 2, "Dungeon"),
    "Dungeon - Hidden Chest": LocationData(BASE_ID + 3, "Dungeon"),
}
```

> **Rule of thumb:** The number of items (excluding events) should equal the number of
> locations. If you have fewer items than locations, add filler. If you have more items,
> add more locations or remove items from the pool.

---

## Step 3 — Define Your Regions (`regions.py`)

```python
# worlds/your_game/regions.py

# This file is optional — you can also create regions directly in __init__.py.
# But separating them improves readability.

region_table: dict[str, list[str]] = {
    "Menu":      [],                          # origin region, no locations
    "Overworld": ["Overworld - Chest 1",
                  "Overworld - Chest 2"],
    "Dungeon":   ["Dungeon - Boss Drop",
                  "Dungeon - Hidden Chest"],
}

# Connections: (source, destination)
entrance_table: list[tuple[str, str]] = [
    ("Menu", "Overworld"),
    ("Overworld", "Dungeon"),
]
```

---

## Step 4 — Define Player Options (`options.py`)

```python
# worlds/your_game/options.py

from dataclasses import dataclass
from Options import (
    Toggle, DefaultOnToggle, Choice, Range,
    PerGameCommonOptions,
)


class Difficulty(Choice):
    """How difficult should the logic be?"""
    display_name = "Difficulty"
    option_easy = 0
    option_normal = 1
    option_hard = 2
    default = 1


class ExtraLocations(Toggle):
    """Include bonus/hidden locations in the randomization pool."""
    display_name = "Extra Locations"


class BossHP(Range):
    """Multiplier for boss hit points (percentage)."""
    display_name = "Boss HP %"
    range_start = 50
    range_end = 200
    default = 100


@dataclass
class YourGameOptions(PerGameCommonOptions):
    difficulty: Difficulty
    extra_locations: ExtraLocations
    boss_hp: BossHP
```

---

## Step 5 — Write Access Rules (`rules.py`)

```python
# worlds/your_game/rules.py

from worlds.generic.Rules import set_rule
from BaseClasses import CollectionState


def set_rules(world) -> None:
    multiworld = world.multiworld
    player = world.player

    # Dungeon requires the Boss Key
    set_rule(
        multiworld.get_entrance("Overworld -> Dungeon", player),
        lambda state: state.has("Boss Key", player),
    )

    # Boss Drop requires the Sword
    set_rule(
        multiworld.get_location("Dungeon - Boss Drop", player),
        lambda state: state.has("Sword", player),
    )
```

---

## Step 6 — The World Class (`__init__.py`)

This is where everything comes together.

```python
# worlds/your_game/__init__.py

from typing import ClassVar
from worlds.AutoWorld import World, WebWorld
from BaseClasses import Region, ItemClassification, Tutorial

from .items import YourGameItem, item_table, item_groups
from .locations import YourGameLocation, location_table
from .regions import region_table, entrance_table
from .options import YourGameOptions
from .rules import set_rules as _set_rules


class YourGameWeb(WebWorld):
    theme = "grass"
    tutorials = [
        Tutorial(
            tutorial_name="Setup Guide",
            description="A guide to setting up Your Game for Archipelago.",
            language="English",
            file_name="setup_en.md",
            link="guide/en",
            authors=["YourName"],
        )
    ]


class YourGameWorld(World):
    """
    Your Game is an exciting adventure where you explore dungeons and collect
    treasures. This integration randomises all items across the multiworld.
    """

    game = "Your Game"
    options_dataclass = YourGameOptions
    options: YourGameOptions                    # typing hint
    web = YourGameWeb()
    topology_present = True

    # ── ID mappings (required) ──────────────────────────────────────
    item_name_to_id = {name: data.code for name, data in item_table.items()
                       if data.code is not None}
    location_name_to_id = {name: data.code for name, data in location_table.items()
                           if data.code is not None}

    # ── Optional groups ─────────────────────────────────────────────
    item_name_groups = item_groups

    # ── Generation hooks ────────────────────────────────────────────

    def create_item(self, name: str) -> YourGameItem:
        data = item_table[name]
        return YourGameItem(name, data.classification, data.code, self.player)

    def create_regions(self) -> None:
        regions: dict[str, Region] = {}
        for region_name, location_names in region_table.items():
            region = Region(region_name, self.player, self.multiworld)
            for loc_name in location_names:
                loc_data = location_table[loc_name]
                region.locations.append(
                    YourGameLocation(self.player, loc_name, loc_data.code, region)
                )
            regions[region_name] = region

        # Connect regions via entrances
        for source_name, dest_name in entrance_table:
            regions[source_name].connect(regions[dest_name])

        self.multiworld.regions += list(regions.values())

    def create_items(self) -> None:
        pool = []
        for name, data in item_table.items():
            if data.code is not None:  # skip events
                pool.append(self.create_item(name))

        # Pad with filler if needed
        while len(pool) < len(self.multiworld.get_unfilled_locations(self.player)):
            pool.append(self.create_item("Health Potion"))

        self.multiworld.itempool += pool

    def set_rules(self) -> None:
        _set_rules(self)

        # Completion condition
        self.multiworld.completion_condition[self.player] = (
            lambda state: state.has("Sword", self.player)
            and state.can_reach_region("Dungeon", self.player)
        )

    def fill_slot_data(self) -> dict:
        return self.options.as_dict("difficulty", "boss_hp")

    def get_filler_item_name(self) -> str:
        return "Health Potion"
```

---

## Step 7 — Game Info and Setup Docs

### `docs/en_Your Game.md`

```markdown
# Your Game

Your Game is a classic action-adventure title. This Archipelago integration
randomises all key items and treasure across the multiworld.

## Where is the options page?

Visit the [player options page](/games/Your%20Game/player-options) to configure
your YAML.
```

### `docs/setup_en.md`

```markdown
# Your Game Setup Guide

## Required Software

- Archipelago from the [releases page](https://github.com/ArchipelagoMW/Archipelago/releases)
- Your Game (original copy)
- The Your Game Archipelago client

## Installation

1. Install Archipelago.
2. Place the Your Game client in the Archipelago directory.
3. Generate a seed with a YAML that includes Your Game.
4. Launch the client and connect to the multiworld server.
```

---

## Step 8 — APWorld Metadata (`archipelago.json`)

```json
{
    "game": "Your Game",
    "world_version": "0.1.0",
    "authors": ["YourName"]
}
```

---

## Checklist

Before moving on, confirm:

- [ ] `worlds/your_game/__init__.py` exists and contains a `World` subclass.
- [ ] `item_name_to_id` and `location_name_to_id` are defined and non-empty.
- [ ] At least one region is named `"Menu"` (the origin region).
- [ ] The number of items equals the number of locations.
- [ ] A completion condition is set.
- [ ] At least one game-info doc (`en_Your Game.md`) exists in `docs/`.
- [ ] At least one setup doc exists in `docs/`.
- [ ] All subfolders with `.py` files have an `__init__.py`.

## Next Step

Proceed to [04 - Implementing Generation Logic](04_implementing_generation_logic.md) for a
deep dive into each generation hook.
