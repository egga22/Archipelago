# 06 — Testing Your Integration

Automated tests catch regressions early and are **required** for contributing to the main
Archipelago repository. This chapter covers the testing framework and how to write tests
for your world.

## Test Infrastructure Overview

Archipelago uses Python's `unittest` framework with `pytest` as the test runner.

```
test/
├── general/            # Generic tests that run against EVERY world
│   ├── test_ids.py     # Verifies item/location IDs are unique and valid
│   ├── test_memory.py  # Memory and performance tests
│   └── ...
├── webhost/            # WebHost-specific tests
└── ...

worlds/your_game/
└── test/
    ├── __init__.py     # Required (can be empty)
    └── test_access.py  # Your game-specific tests
```

> **Important:** Generic tests in `test/general/` will automatically run against your world.
> You do not need to do anything special for this to happen.

## Setting Up Your Test Package

Create the test directory:

```bash
mkdir -p worlds/your_game/test
touch worlds/your_game/test/__init__.py
```

## Writing Tests with WorldTestBase

Archipelago provides `WorldTestBase` — a purpose-built base class that creates a
pre-configured multiworld for your game.

```python
# worlds/your_game/test/test_access.py

from worlds.AutoWorld import World
from test.bases import WorldTestBase


class YourGameTestBase(WorldTestBase):
    game = "Your Game"
    # Optional: set options for this test class
    # options = {"difficulty": 2, "boss_hp": 150}


class TestAccess(YourGameTestBase):
    """Test that logic rules work correctly."""

    def test_sword_required_for_boss(self):
        """The boss drop should not be reachable without the Sword."""
        self.assertAccessDependency(
            ["Dungeon - Boss Drop"],
            [["Sword"]],
        )

    def test_boss_key_required_for_dungeon(self):
        """The dungeon region should not be accessible without the Boss Key."""
        self.collect_all_but(["Boss Key"])
        self.assertFalse(
            self.can_reach_region("Dungeon"),
            "Dungeon should not be reachable without the Boss Key.",
        )
        self.collect_by_name("Boss Key")
        self.assertTrue(
            self.can_reach_region("Dungeon"),
            "Dungeon should be reachable with the Boss Key.",
        )


class TestGoal(YourGameTestBase):
    """Test that the game can be completed."""

    def test_all_state_can_reach_everything(self):
        """With all items, every location should be reachable."""
        # This test is inherited from WorldTestBase automatically,
        # but you can override it if needed.
        super().test_all_state_can_reach_everything()

    def test_completion(self):
        """The game should be completable with all items."""
        self.collect_all_but([])
        self.assertTrue(
            self.multiworld.completion_condition[self.player](self.multiworld.state),
            "The game should be completable with all items collected.",
        )
```

## WorldTestBase Built-in Tests

These tests run automatically for every `WorldTestBase` subclass:

| Test | What it verifies |
|------|-----------------|
| `test_all_state_can_reach_everything` | With all items collected, every location and region is reachable. |
| `test_empty_state_can_reach_something` | With no items, at least something is reachable (typically the Menu). |
| `test_fill` | The fill algorithm can successfully place all items. |

## Useful Helper Methods

| Method | Purpose |
|--------|---------|
| `self.collect_all_but(names)` | Collect all items except those listed. |
| `self.collect_by_name(name)` | Collect a specific item. |
| `self.can_reach_region(name)` | Check if a region is reachable in current state. |
| `self.can_reach_location(name)` | Check if a location is reachable. |
| `self.assertAccessDependency(locs, items)` | Assert that locations require specific items. |
| `self.get_items_by_name(name)` | Get all items with a given name. |
| `self.count(name)` | Count how many of an item are collected. |

## Testing with Different Options

```python
class TestHardMode(YourGameTestBase):
    options = {"difficulty": 2}  # Hard mode

    def test_hard_mode_specific_logic(self):
        # Test something specific to hard mode
        ...


class TestEasyMode(YourGameTestBase):
    options = {"difficulty": 0}  # Easy mode

    def test_easy_mode_has_more_items(self):
        # Verify easy mode behavior
        ...
```

## Running Tests

### Run All Tests

```bash
pip install pytest pytest-subtests
pytest
```

### Run Only Your World's Tests

```bash
pytest worlds/your_game/test/ -v
```

### Run a Specific Test

```bash
pytest worlds/your_game/test/test_access.py::TestAccess::test_sword_required_for_boss -v
```

### Run Generic Tests Against Your World Only

```bash
pytest test/general/ -k "Your Game" -v
```

### Multithreaded Testing (Faster)

```bash
pip install pytest-xdist
pytest -n12
```

### PyCharm

Right-click on `worlds/your_game/test/` → **Run 'Unittests in test'**.

## Performance Guidelines

- Each individual test should take **less than 1 second**.
- Avoid creating multiple multiworlds in a single test.
- Focus tests on **logic correctness**, not on fill randomness.
- Use `WorldTestBase` instead of raw `unittest.TestCase` when possible.

## Manual End-to-End Testing

Automated tests cover generation logic, but you should also test the full pipeline:

1. **Create a test YAML:**
   ```yaml
   # Players/YourGame.yaml
   name: TestPlayer
   game: Your Game
   Your Game:
     difficulty: normal
     boss_hp: 100
   ```

2. **Generate a seed:**
   ```bash
   python Generate.py
   ```

3. **Host locally:**
   ```bash
   python MultiServer.py output/<generated_file>.archipelago
   ```

4. **Connect your client** and verify:
   - Items are received correctly.
   - Location checks are sent and acknowledged.
   - Goal completion triggers properly.
   - Reconnection works after disconnect.

5. **Check the spoiler log** to verify item placements make logical sense.

## Test Checklist

- [ ] All generic tests pass (`test/general/`).
- [ ] Your custom access tests pass.
- [ ] Tests with different option combinations pass.
- [ ] The fill algorithm completes without errors.
- [ ] Manual end-to-end test succeeds.

## Next Step

Proceed to [07 - Web and Launcher Integration](07_web_and_launcher_integration.md) to set
up the web interface and launcher components.
