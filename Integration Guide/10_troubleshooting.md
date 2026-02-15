# 10 — Troubleshooting

This chapter covers common errors and issues you may encounter when developing an
Archipelago game integration, along with their solutions.

---

## Generation Errors

### `Fill algorithm could not place all items`

**Cause:** The fill algorithm cannot find valid placements for all items given your rules.

**Solutions:**
- Check that the number of items equals the number of locations.
- Verify your rules are not too restrictive.
- Ensure you have enough sphere-1 locations (reachable with no items).
- Use `local_early_items` to hint that certain items should be in sphere 1:
  ```python
  def generate_early(self):
      self.local_early_items["Boss Key"] = 1
  ```
- Add more locations that are freely accessible.
- Use `push_precollected` to give players starting items that unlock initial locations.

### `KeyError: 'Item Name'` in `create_item`

**Cause:** AP is trying to create an item by name (for plando, start inventory, or item
links) but your `create_item` doesn't handle it.

**Solution:** Make sure `create_item` can create **any** item in `item_name_to_id`:
```python
def create_item(self, name: str) -> YourGameItem:
    data = item_table[name]
    return YourGameItem(name, data.classification, data.code, self.player)
```

### `No Region named "Menu" for player X`

**Cause:** Your world doesn't create a region with the origin name (default: `"Menu"`).

**Solution:** Ensure you create a `"Menu"` region, or set `origin_region_name` on your
World class:
```python
class YourGameWorld(World):
    origin_region_name = "Start Area"  # if you don't want to use "Menu"
```

### `Locations left unfilled`

**Cause:** More locations than items.

**Solution:** Add filler items to balance:
```python
def create_items(self):
    pool = [...]
    locations_count = len(self.multiworld.get_unfilled_locations(self.player))
    while len(pool) < locations_count:
        pool.append(self.create_filler())
    self.multiworld.itempool += pool
```

### `Items left over after fill`

**Cause:** More items than locations.

**Solution:** Remove excess items or add more locations.

---

## Import Errors

### `ModuleNotFoundError` when loading as .apworld

**Cause:** Using absolute imports within your world package.

**Fix:** Use relative imports for all intra-world references:
```python
# Wrong:
from worlds.your_game.items import YourGameItem

# Right:
from .items import YourGameItem
```

### `ImportError: cannot import name 'X' from 'Y'`

**Cause:** Circular imports between your modules.

**Fix:** Move the import inside the function that needs it, or restructure to avoid
cycles.

---

## Test Failures

### Generic test `test_ids` fails

**Cause:** Item or location IDs collide with another world, are out of range, or are
missing.

**Fix:**
- Choose IDs in the range 1 to 2^31 - 1.
- Make sure every item/location with a non-None code has a unique ID within your game.
- Check that `item_name_to_id` and `location_name_to_id` are populated correctly.

### `test_all_state_can_reach_everything` fails

**Cause:** Some location is unreachable even with all items collected.

**Fix:**
- Check your rules: is there a rule that can never be satisfied?
- Ensure all regions are connected to the origin region via entrances.
- Use `visualize_regions` to debug the region graph.

### `test_fill` fails

**Cause:** Item count ≠ location count, or rules are too restrictive.

**Fix:** See the "Fill algorithm could not place all items" section above.

---

## Client Issues

### Client connects but receives no items

**Possible causes:**
- `items_handling` is set to 0 (should be `0b111` to receive all items).
- The slot name doesn't match the generated YAML.
- The game name doesn't match `World.game`.

### Items are received but not applied in-game

**Cause:** The `on_package` handler for `ReceivedItems` isn't implementing the game-side
logic.

**Fix:** Ensure you iterate through `args["items"]` and apply each one.

### Connection refused with `InvalidGame`

**Cause:** The game name in your `Connect` packet doesn't match the generated slot.

**Fix:** Ensure `ctx.game` (or the `game` field in the Connect packet) exactly matches
your `World.game` string.

### Connection refused with `IncompatibleVersion`

**Cause:** Version mismatch between client and server.

**Fix:** Update your client to use the same network protocol version as the server.

---

## WebHost Issues

### Game doesn't appear on the website

**Cause:** The world failed to load (import error, syntax error, etc.).

**Fix:**
1. Run `python WebHost.py` and check the console for errors.
2. Run `python -c "import worlds.your_game"` to test the import.

### Options page is blank or broken

**Cause:** `options_dataclass` is not set, or option classes have errors.

**Fix:** Verify your options dataclass inherits from `PerGameCommonOptions` and all fields
are valid option types.

### Game info page shows raw markdown

**Cause:** The markdown file has syntax issues or is not properly linked.

**Fix:** Check the filename matches the pattern `{lang}_{Game Name}.md` exactly.

---

## Performance Issues

### Generation takes too long

**Possible causes:**
- Too many locations or items.
- Overly complex rules that trigger excessive re-evaluation.
- Using `can_reach` in entrance rules without indirect conditions.

**Solutions:**
- Set `explicit_indirect_conditions = False` if you have many cross-region dependencies
  (trades performance for correctness).
- Simplify rules where possible.
- Use events to consolidate repeated rule checks.

### Tests take too long

**Fix:**
- Each test should complete in under 1 second.
- Avoid creating multiple multiworlds per test.
- Use `WorldTestBase` which reuses a single multiworld.

---

## Debugging Tips

### Visualize the Region Graph

```python
from Utils import visualize_regions
visualize_regions(
    self.multiworld.get_region("Menu", self.player),
    "my_world.puml",
)
```

Open the `.puml` file with a PlantUML viewer to see your region layout.

### Enable Network Logging

```bash
python MultiServer.py output/your_file.archipelago --log_network
```

This logs all packets to help debug client/server communication.

### Print the Spoiler Log

After generation, check `output/<seed>_Spoiler.txt` for:
- Item placements.
- Playthrough (expected item acquisition order).
- Unreachable locations (which shouldn't exist).

### Check State at a Breakpoint

In a test, you can inspect the state:
```python
def test_debug(self):
    self.collect_by_name("Sword")
    print(f"Can reach Dungeon: {self.can_reach_region('Dungeon')}")
    print(f"Collected items: {[i.name for i in self.multiworld.state.prog_items]}")
```

---

## FAQ

### Can I have 0 items and 0 locations?

Technically yes (the "Archipelago" meta-game does this), but it's extremely atypical.
Your world would just be a spectator.

### Can items be received multiple times?

Yes. Your client **must** handle receiving any item any number of times.

### Can I use external Python packages?

Yes. List them in `worlds/your_game/requirements.txt`. Run `ModuleUpdate.py` to install.

### Can I store persistent data between generations?

No. Each generation is independent. Use `fill_slot_data` to pass data to the client.

### What if my game has missable locations?

Archipelago assumes all locations stay reachable forever. Options:
- Modify the game to make them repeatable.
- Don't include missable locations in the pool.
- Warn players about save management.
- Only include logic for repeatable methods of access.

See the [APWorld Dev FAQ](/docs/apworld_dev_faq.md) for more details.

## Next Step

Proceed to [11 - AI Prompts for Development](11_ai_prompts_for_development.md) for
ready-to-use AI prompts to accelerate your development.
