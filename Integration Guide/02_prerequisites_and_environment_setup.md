# 02 — Prerequisites and Environment Setup

## Required Tools

| Tool | Version | Notes |
|------|---------|-------|
| **Python** | 3.11.9 – 3.13.x | Do **not** use the Windows Store version. Download from [python.org](https://www.python.org/downloads/). |
| **pip** | Latest | Ships with Python. |
| **Git** | Latest | For cloning and contributing. |
| **A code editor** | — | [PyCharm Community](https://www.jetbrains.com/pycharm/download/) (recommended), VS Code, or any editor with Python support. |
| **C compiler** (optional) | — | Only needed if a dependency requires compilation. On Windows: Visual Studio Build Tools. On Linux: `build-essential`. |

## Step 1 — Fork and Clone the Repository

1. Go to the Archipelago repository on GitHub (the upstream or your fork source).
2. Click **Fork** to create your own copy.
3. Clone your fork locally:

```bash
git clone https://github.com/<your-username>/Archipelago.git
cd Archipelago
```

4. Add the upstream remote so you can pull updates later:

```bash
git remote add upstream https://github.com/ArchipelagoMW/Archipelago.git
```

## Step 2 — Create a Feature Branch

Always work on a dedicated branch:

```bash
git checkout -b world/your-game-name
```

## Step 3 — Install Python Dependencies

Run the module updater, which reads every `requirements.txt` in the project:

```bash
python ModuleUpdate.py
```

> **Tip:** If you add a `worlds/your_game/requirements.txt` with extra packages, re-run
> `ModuleUpdate.py` to install them.

### Alternative: Virtual Environment (Recommended on Linux/macOS)

```bash
python -m venv venv
source venv/bin/activate      # Linux/macOS
# venv\Scripts\activate       # Windows
python ModuleUpdate.py
```

## Step 4 — Verify the Setup

Run the launcher to make sure everything loads:

```bash
python Launcher.py
```

Or generate a test seed using one of the built-in games:

```bash
# Place a minimal YAML in the Players/ folder first, then:
python Generate.py
```

Run the test suite to confirm nothing is broken:

```bash
pip install pytest pytest-subtests
pytest
```

## Step 5 — (Optional) Set Up Your Editor

### PyCharm

1. Open the Archipelago folder as a project.
2. Set the Python interpreter to the one you installed (or your venv).
3. Right-click the `test/` folder → **Mark Directory as → Test Sources Root**.
4. Run `ModuleUpdate.py` once from PyCharm's terminal.
5. Run `WebHost.py` once to build web assets.

### VS Code

1. Open the folder.
2. Install the **Python** extension.
3. Select the interpreter (Ctrl+Shift+P → "Python: Select Interpreter").
4. Configure `pytest` as the test runner in `.vscode/settings.json`:

```json
{
    "python.testing.pytestEnabled": true,
    "python.testing.pytestArgs": ["test"]
}
```

## Step 6 — Understand the Repository Layout

```
Archipelago/
├── BaseClasses.py          # Region, Location, Item, Entrance, MultiWorld
├── CommonClient.py         # Reusable Python client base class
├── Fill.py                 # Item placement algorithm
├── Generate.py             # Seed generation entry point
├── Launcher.py             # GUI launcher
├── Main.py                 # Main entry point
├── MultiServer.py          # Server for hosting multiworld sessions
├── Options.py              # Option base classes
├── WebHost.py              # Web server for hosting
├── worlds/
│   ├── AutoWorld.py        # World and WebWorld base classes
│   ├── generic/            # Built-in "Archipelago" meta-world
│   ├── <game_name>/        # Each game is a package here
│   └── ...
├── docs/                   # Official documentation
├── test/                   # Global test suite
│   ├── general/            # Tests that run against every world
│   └── ...
└── data/                   # Static data files
```

## GitHub Actions (CI)

If you enable GitHub Actions on your fork, every push will automatically run:

- The full test suite across supported Python versions.
- Linting and style checks.

To enable: Go to your fork → **Actions** tab → click **Enable workflows**.

This catches regressions early and is *strongly recommended*.

## Next Step

Proceed to [03 - Creating the World Package](03_creating_the_world_package.md) to start
building your game's world.
