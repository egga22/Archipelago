# Archipelago Custom Game Integration Guide

Welcome! This guide walks you through every step of creating a custom game integration
(called an "APWorld") for [Archipelago](https://archipelago.gg/), the popular multiworld
randomizer framework.

## Who This Guide Is For

- Game modders who want to add multiworld randomizer support to their favourite game.
- Developers comfortable with Python 3.11+ who want to learn the Archipelago framework.
- Anyone looking for AI-assisted prompts to speed up the development process.

## Guide Contents

| # | Document | Description |
|---|----------|-------------|
| 1 | [01 - Overview and Concepts](01_overview_and_concepts.md) | What Archipelago is, key terminology, and how integrations work at a high level. |
| 2 | [02 - Prerequisites and Environment Setup](02_prerequisites_and_environment_setup.md) | Tools, Python version, repository fork/clone, and first-time build. |
| 3 | [03 - Creating the World Package](03_creating_the_world_package.md) | Folder structure, items, locations, regions, options, and the World class. |
| 4 | [04 - Implementing Generation Logic](04_implementing_generation_logic.md) | The generation pipeline hooks (`generate_early` → `generate_output`) with code examples. |
| 5 | [05 - Building the Game Client](05_building_the_game_client.md) | How to build a client that talks to the Archipelago server via WebSocket. |
| 6 | [06 - Testing Your Integration](06_testing_your_integration.md) | Writing unit tests, running the test suite, and manual end-to-end testing. |
| 7 | [07 - Web and Launcher Integration](07_web_and_launcher_integration.md) | WebWorld setup, tutorials, themes, option presets, and Launcher components. |
| 8 | [08 - Packaging and Distribution](08_packaging_and_distribution.md) | Building `.apworld` files, metadata, and distribution. |
| 9 | [09 - Submitting to Archipelago](09_submitting_to_archipelago.md) | Pull request workflow, code review expectations, and maintainer responsibilities. |
| 10 | [10 - Troubleshooting](10_troubleshooting.md) | Common errors, debugging techniques, and FAQ. |
| 11 | [11 - AI Prompts for Development](11_ai_prompts_for_development.md) | Ready-to-use AI prompts for every stage of development. |

## Quick-Start Path

If you want the fastest path to a working prototype:

1. Read **01 – Overview** to understand the concepts.
2. Follow **02 – Prerequisites** to set up your environment.
3. Copy the skeleton in **03 – Creating the World Package** into `worlds/your_game/`.
4. Implement the generation hooks from **04 – Generation Logic**.
5. Use **06 – Testing** to verify everything works.
6. Build your game client using **05 – Building the Game Client**.
7. Package and share using **08 – Packaging**.

## Conventions Used in This Guide

- `your_game` — placeholder for your game's folder/module name (e.g., `my_cool_game`).
- `Your Game` — placeholder for the human-readable game name (e.g., `My Cool Game`).
- `YourGame` — placeholder for your PascalCase class prefix (e.g., `MyCoolGame`).
- Code blocks prefixed with `#` comments explain what each section does.
- File paths are relative to the Archipelago repository root unless stated otherwise.

## Additional Resources

- [Official Archipelago Docs](/docs/) — The canonical documentation.
- [Archipelago Discord](https://archipelago.gg/discord) — `#ap-world-dev` and `#ap-modding-help` channels.
- [AutoWorld.py](/worlds/AutoWorld.py) — Base `World` and `WebWorld` class definitions.
- [BaseClasses.py](/BaseClasses.py) — Core classes (`Region`, `Location`, `Item`, `Entrance`).
- [Network Protocol](/docs/network%20protocol.md) — Full packet specification for clients.
