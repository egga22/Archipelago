# 08 — Packaging and Distribution

Once your integration is working, you can package it as an `.apworld` file for easy
distribution or submit it for inclusion in the main Archipelago repository.

## What Is an .apworld File?

An `.apworld` is a **ZIP archive** (with the `.apworld` extension) containing your world
folder. It allows players to install your game support without modifying the core
Archipelago installation.

**Structure inside the ZIP:**

```
your_game.apworld          ← the ZIP file (all lowercase)
└── your_game/             ← folder with the same name (case-sensitive)
    ├── __init__.py
    ├── items.py
    ├── locations.py
    ├── options.py
    ├── rules.py
    ├── regions.py
    ├── archipelago.json
    ├── docs/
    │   ├── en_Your Game.md
    │   └── setup_en.md
    └── test/
        ├── __init__.py
        └── test_access.py
```

> **Warning:** The `.apworld` filename must be **all lowercase**. Mixed case causes
> import errors on frozen Python 3.10+.

## The archipelago.json Manifest

```json
{
    "game": "Your Game",
    "world_version": "1.0.0",
    "authors": ["YourName"],
    "minimum_ap_version": "0.6.0"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `game` | Yes | Human-readable game name. Must match `World.game`. |
| `world_version` | No | Semantic version of your world (`major.minor.build`). |
| `authors` | No | List of author names. |
| `minimum_ap_version` | No | Minimum Archipelago version required. |
| `maximum_ap_version` | No | Maximum Archipelago version supported. |

> `version` and `compatible_version` (APContainer packaging fields) are **automatically
> added** by the build tool. Do not write them manually.

## Building with the Launcher

The recommended way to build `.apworld` files:

1. Open the Archipelago Launcher.
2. Click **"Build APWorlds"**.
3. Find the output in `build/apworlds/`.

Or from the command line:

```bash
python Launcher.py "Build APWorlds" -- "Your Game"
```

This automatically:
- Packages your world folder into a ZIP.
- Adds `version` and `compatible_version` to `archipelago.json`.
- Respects `.apignore` exclusions.

## .apignore File

Similar to `.gitignore`, this file excludes unnecessary files from the `.apworld` package.
Place it in your world folder root.

```gitignore
# worlds/your_game/.apignore

# Exclude development files
*.pyc
__pycache__/
.mypy_cache/
*.egg-info/

# Exclude test data
test_data/

# Exclude scripts not needed at runtime
scripts/
dev_notes.md
```

Some exclusions are made globally (like `__pycache__/`) via `data/GLOBAL.apignore`.

## Installing an .apworld

Players install your world by placing the `.apworld` file into:

- **Source install:** `worlds/` directory.
- **Binary install:** `<install_dir>/lib/worlds/` directory.

Archipelago will automatically discover and load it.

## Distribution Options

### Option A: Unofficial .apworld Release

1. Build your `.apworld` file.
2. Upload it to GitHub Releases, a Discord channel, or a hosting service.
3. Provide installation instructions.

**Pros:** Quick to distribute, no review process.  
**Cons:** Players must manually install, won't be in the official game list.

### Option B: Submit to Main Repository

1. Fork the main Archipelago repo.
2. Add your world to `worlds/`.
3. Open a Pull Request.
4. Go through the review process (see next chapter).

**Pros:** Official support, automatic inclusion in releases, wider audience.  
**Cons:** Must meet all quality standards, ongoing maintenance responsibility.

## Import Rules for .apworld Compatibility

- **Within your world:** Use **relative imports**  
  ```python
  from .items import YourGameItem
  from .options import YourGameOptions
  ```

- **From AP core:** Use **absolute imports**  
  ```python
  from BaseClasses import Item, Location, Region
  from worlds.AutoWorld import World
  from Options import Toggle, Range
  ```

If you use absolute imports within your world package, the `.apworld` ZIP loader will
fail to resolve them.

## Version Management

When releasing updates:

1. Bump `world_version` in `archipelago.json`.
2. Rebuild the `.apworld`.
3. Archipelago will load the highest valid version if multiple are present.

## Next Step

Proceed to [09 - Submitting to Archipelago](09_submitting_to_archipelago.md) to learn the
pull request workflow and review expectations.
