# 07 — Web and Launcher Integration

Archipelago has a web interface (WebHost) for generating seeds and displaying game
information, as well as a desktop Launcher. This chapter explains how to integrate your
game with both.

## WebWorld Configuration

The `WebWorld` class controls how your game appears on the Archipelago website.

```python
# worlds/your_game/__init__.py

from worlds.AutoWorld import WebWorld
from BaseClasses import Tutorial


class YourGameWeb(WebWorld):
    # ── Theme ───────────────────────────────────────────────────
    # Choose from: dirt, grass, grassFlowers, ice, jungle, ocean,
    #              partyTime, stone
    theme = "jungle"

    # ── Rich Text ───────────────────────────────────────────────
    # Set to True to render option docs with reStructuredText
    rich_text_options_doc = True

    # ── Bug Report Page ─────────────────────────────────────────
    bug_report_page = "https://github.com/YourUser/Archipelago/issues"

    # ── Game Info Languages ─────────────────────────────────────
    # Default is ["en"]. Add more if you provide translated docs.
    game_info_languages = ["en"]

    # ── Tutorials ───────────────────────────────────────────────
    tutorials = [
        Tutorial(
            tutorial_name="Setup Guide",
            description="How to set up Your Game for Archipelago multiworld.",
            language="English",
            file_name="setup_en.md",
            link="guide/en",
            authors=["YourName"],
        ),
    ]

    # ── Option Presets ──────────────────────────────────────────
    options_presets = {
        "Beginner": {
            "difficulty": "easy",
            "boss_hp": 50,
            "extra_locations": False,
        },
        "Challenge": {
            "difficulty": "hard",
            "boss_hp": 200,
            "extra_locations": True,
        },
    }

    # ── Option Groups ───────────────────────────────────────────
    # Organize options into collapsible sections on the web page.
    # "Game Options" and "Item & Location Options" are auto-created.
    # Add custom groups for better organization:
    from Options import OptionGroup
    option_groups = [
        OptionGroup("Difficulty Settings", [
            # reference your option classes here
        ]),
    ]

    # ── Item / Location Descriptions ────────────────────────────
    item_descriptions = {
        "Sword": "The hero's trusty blade. Required to defeat bosses.",
        "Boss Key": "Opens the door to the dungeon boss room.",
    }
    location_descriptions = {
        "Dungeon - Boss Drop": "Dropped by the dungeon boss after defeat.",
    }
```

## Game Info Page

The game info page is displayed on the website. It must be in `docs/` inside your world
folder.

**Naming convention:** `{language_code}_{Game Name}.md`

Example: `docs/en_Your Game.md`

```markdown
# Your Game

Your Game is a classic action-adventure where you explore dungeons and collect treasures.

## What does randomization do?

All key items and treasures are shuffled across the multiworld. You may find items from
other players' games in your chests, and they may find your items in theirs.

## What is the goal?

Collect the Sword and defeat the Dungeon Boss.

## What items can appear in other players' worlds?

- Sword
- Shield
- Boss Key
- Health Potions
```

## Setup / Tutorial Documents

Setup docs guide players through installing and configuring the client.

**Naming convention:** `setup_{language_code}.md`

Example: `docs/setup_en.md`

These are referenced in the `tutorials` list of your `WebWorld`.

## Launcher Component

To add your client to the Archipelago desktop launcher:

```python
# worlds/your_game/__init__.py (at module level, outside classes)

from worlds.LauncherComponents import Component, components, Type, launch_subprocess


def launch_client(*args):
    from .client import run_client
    launch_subprocess(run_client, name="YourGameClient", args=args)


components.append(
    Component(
        display_name="Your Game Client",
        func=launch_client,
        component_type=Type.CLIENT,
        game_name="Your Game",
        # Optional:
        # icon="your_game",          # 48x48 icon name
        # supports_uri=True,         # enable webhost slot links
        # file_identifier=...,       # launch by file association
    )
)
```

### Client Icon

If you define an `icon`, place a 48×48 pixel PNG in the `data/` directory. Smaller or
larger images will be scaled to 48×48.

### URI Support

Setting `supports_uri=True` enables "click to launch" links on the webhost when a player
generates a room with your game. The launcher will open your client and pass the connection
info.

## Testing the Web Interface

1. Run the web host locally:
   ```bash
   python WebHost.py
   ```
2. Open `http://localhost:52881` in your browser.
3. Navigate to your game's page.
4. Verify:
   - Game info page renders correctly.
   - Options page shows all options with correct descriptions.
   - Option presets work.
   - Setup guide is linked and renders.
   - Theme looks good.

## Testing the Launcher

1. Run the launcher:
   ```bash
   python Launcher.py
   ```
2. Verify your client appears in the launcher list.
3. Click it and verify it launches correctly.

## Next Step

Proceed to [08 - Packaging and Distribution](08_packaging_and_distribution.md) to learn
how to build and share your `.apworld` file.
