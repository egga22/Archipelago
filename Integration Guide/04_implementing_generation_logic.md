# 04 — Implementing Generation Logic

The Archipelago generator calls a series of hooks on your `World` class in a fixed order.
This chapter explains each hook, when to use it, and provides practical examples.

## The Generation Pipeline

```
stage_assert_generate (class method)    ← verify prerequisites
         │
    generate_early                      ← read options, set up world state
         │
    create_regions                      ← build region graph and place locations
         │
    create_items                        ← populate the item pool
         │
    set_rules                           ← attach access and item rules
         │
    connect_entrances                   ← finalize entrances (entrance randomization)
         │
    generate_basic                      ← non-logic randomization
         │
    pre_fill                            ← manual item placement before fill
         │
    ─── Fill algorithm runs ───         ← AP places items into locations
         │
    post_fill                           ← adjustments after fill
         │
    generate_output                     ← write output files (ROM patches, etc.)
         │
    fill_slot_data                      ← data sent to game client
    modify_multidata                    ← modify server-hosted data
```

> Each instance method may also have a corresponding `stage_` class method that runs
> after **all** players' instance methods complete (e.g., `stage_generate_early`).

---

## Hook-by-Hook Reference

### `stage_assert_generate(cls, multiworld)`

**When:** Before any world is created.  
**Purpose:** Verify prerequisites (e.g., ROM file exists).

```python
@classmethod
def stage_assert_generate(cls, multiworld) -> None:
    # Example: require a ROM file
    if not cls.settings.rom_file.exists():
        raise FileNotFoundError("Your Game ROM not found. "
                                "Check host.yaml for the correct path.")
```

### `generate_early(self)`

**When:** After options are parsed, before items/locations are created.  
**Purpose:** Read options, compute derived values, set up instance variables.

```python
def generate_early(self) -> None:
    self.boss_hp_multiplier = self.options.boss_hp.value / 100.0

    # You can modify local_items / non_local_items here (last chance).
    if self.options.difficulty == "hard":
        self.hard_mode = True
    else:
        self.hard_mode = False
```

### `create_regions(self)`

**When:** After `generate_early`.  
**Purpose:** Add regions with locations to `self.multiworld.regions`.

```python
def create_regions(self) -> None:
    menu = Region("Menu", self.player, self.multiworld)
    overworld = Region("Overworld", self.player, self.multiworld)
    dungeon = Region("Dungeon", self.player, self.multiworld)

    # Add locations to regions
    for name, data in location_table.items():
        region = {"Overworld": overworld, "Dungeon": dungeon}[data.region]
        loc = YourGameLocation(self.player, name, data.code, region)
        region.locations.append(loc)

    # Connect regions
    menu.connect(overworld)
    overworld.connect(dungeon)

    # Add all regions (use append/extend/+=, NEVER `=`)
    self.multiworld.regions += [menu, overworld, dungeon]
```

### `create_items(self)`

**When:** After `create_regions`.  
**Purpose:** Add items to `self.multiworld.itempool`.

```python
def create_items(self) -> None:
    pool = []
    for name, data in item_table.items():
        if data.code is not None:
            pool.append(self.create_item(name))

    # Balance: items must equal unfilled locations
    locations_count = len(self.multiworld.get_unfilled_locations(self.player))
    while len(pool) < locations_count:
        pool.append(self.create_filler())

    # Remove excess if somehow there are too many
    while len(pool) > locations_count:
        pool.pop()

    self.multiworld.itempool += pool
```

### `create_item(self, name: str)`

**When:** Called by `create_items` and by AP itself (plando, start inventory, item links).  
**Purpose:** Create a single item by name. **Must** be implemented.

```python
def create_item(self, name: str) -> YourGameItem:
    data = item_table[name]
    return YourGameItem(name, data.classification, data.code, self.player)
```

### `set_rules(self)`

**When:** After `create_items`.  
**Purpose:** Attach access rules to locations and entrances.

```python
from worlds.generic.Rules import set_rule, add_rule, forbid_item

def set_rules(self) -> None:
    player = self.player
    mw = self.multiworld

    # Region access
    set_rule(mw.get_entrance("Overworld -> Dungeon", player),
             lambda state: state.has("Boss Key", player))

    # Location access
    set_rule(mw.get_location("Dungeon - Boss Drop", player),
             lambda state: state.has("Sword", player))

    # Require multiple items
    set_rule(mw.get_location("Dungeon - Hidden Chest", player),
             lambda state: state.has("Sword", player) and
                           state.has("Shield", player))

    # Require N copies of an item
    set_rule(mw.get_location("Overworld - Chest 2", player),
             lambda state: state.has("Health Potion", player, 3))

    # Completion condition
    mw.completion_condition[player] = (
        lambda state: state.has("Sword", player)
        and state.can_reach_region("Dungeon", player)
    )
```

### `connect_entrances(self)`

**When:** After `set_rules`.  
**Purpose:** Finalize entrance connections. Used primarily for entrance randomization.

If you are not doing entrance randomization, you can skip this hook and connect regions
in `create_regions` instead.

### `generate_basic(self)`

**When:** After `connect_entrances`.  
**Purpose:** Player-specific randomization that does not affect logic (e.g., cosmetic
shuffles, random NPC names).

### `pre_fill(self)`

**When:** Before the fill algorithm runs.  
**Purpose:** Manually place items that must go in specific locations.

```python
def pre_fill(self) -> None:
    # Place a specific item at a specific location
    loc = self.multiworld.get_location("Dungeon - Boss Drop", self.player)
    loc.place_locked_item(self.create_item("Boss Key"))
```

> Items placed during `pre_fill` should **not** be in `self.multiworld.itempool`.

### `generate_output(self, output_directory: str)`

**When:** After fill is complete.  
**Purpose:** Generate output files (ROM patches, data files, etc.).

```python
import os

def generate_output(self, output_directory: str) -> None:
    # At this point, every location has an item assigned.
    output = {}
    for loc in self.multiworld.get_locations(self.player):
        if loc.address is not None:
            output[loc.name] = {
                "item": loc.item.name,
                "player": loc.item.player,
            }

    # Write to a file
    filename = os.path.join(
        output_directory,
        f"{self.multiworld.get_out_file_name_base(self.player)}.json"
    )
    with open(filename, "w") as f:
        import json
        json.dump(output, f, indent=2)
```

### `fill_slot_data(self)`

**When:** After output generation.  
**Purpose:** Return a dictionary of data that will be sent to the client on connect.

```python
def fill_slot_data(self) -> dict:
    return {
        "difficulty": self.options.difficulty.value,
        "boss_hp": self.options.boss_hp.value,
        "seed": self.random.randint(0, 2**32 - 1),
    }
```

---

## Events (Generation-Only Items/Locations)

Events model in-game triggers that affect logic but are not real AP locations.

```python
from BaseClasses import ItemClassification

# In create_regions or set_rules:
boss_region = self.multiworld.get_region("Dungeon", self.player)
event_loc = YourGameLocation(self.player, "Defeat Boss", None, boss_region)
event_loc.place_locked_item(
    YourGameItem("Boss Defeated", ItemClassification.progression, None, self.player)
)
boss_region.locations.append(event_loc)

# Use the event in rules:
set_rule(some_other_location,
         lambda state: state.has("Boss Defeated", player))

# Use the event as completion condition:
self.multiworld.completion_condition[self.player] = (
    lambda state: state.has("Boss Defeated", self.player)
)
```

## Indirect Conditions

If an entrance rule uses `state.can_reach_region`, you **must** register an indirect
condition:

```python
self.multiworld.register_indirect_condition(
    self.multiworld.get_region("Some Region", self.player),
    self.multiworld.get_entrance("Gate to Area", self.player),
)
```

Or set `explicit_indirect_conditions = False` on your World class (at a performance cost).

---

## Debugging the Region Graph

Uncomment this in `set_rules` to generate a PlantUML diagram:

```python
from Utils import visualize_regions
visualize_regions(
    self.multiworld.get_region("Menu", self.player),
    "my_world.puml",
)
```

## Next Step

Proceed to [05 - Building the Game Client](05_building_the_game_client.md) to learn how
to connect your game to the Archipelago server.
