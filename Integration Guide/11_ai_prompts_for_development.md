# 11 — AI Prompts for Development

This chapter provides ready-to-use prompts you can give to an AI assistant (like GitHub
Copilot, ChatGPT, or Claude) at each stage of developing your Archipelago integration.
Copy, paste, and customize the bracketed sections.

---

## Phase 1 — Planning and Research

### Prompt: Analyze My Game for Randomization

```
I want to create an Archipelago multiworld randomizer integration for [GAME NAME].

The game is a [GENRE] game where the player [BRIEF DESCRIPTION OF GAMEPLAY].

Key progression items in the game include:
- [ITEM 1]: [what it does]
- [ITEM 2]: [what it does]
- [ITEM 3]: [what it does]

Key locations where items are found:
- [LOCATION 1]: [how to access it]
- [LOCATION 2]: [how to access it]
- [LOCATION 3]: [how to access it]

The game is completed when the player [GOAL DESCRIPTION].

Please analyze this game and suggest:
1. Which items should be classified as "progression" vs "useful" vs "filler".
2. How to divide the game world into logical regions.
3. What access rules would be needed between regions.
4. What player options would be good to offer.
5. Any potential issues with randomization (missable locations, softlocks, etc.).
```

### Prompt: Research Existing Similar Integrations

```
I'm developing an Archipelago game integration for [GAME NAME], which is a
[GENRE] game similar to [SIMILAR GAME].

Please look at how existing Archipelago worlds handle similar game types.
Specifically, I'd like to know:
1. Common patterns for [SPECIFIC ASPECT, e.g., "RPG inventory systems"].
2. How other worlds handle [SPECIFIC CHALLENGE, e.g., "missable locations"].
3. Best practices for [SPECIFIC FEATURE, e.g., "entrance randomization"].
```

---

## Phase 2 — World Package Structure

### Prompt: Generate Item Definitions

```
I'm creating an Archipelago world for [GAME NAME]. Here are all the items in my game:

Progression items (required for logic):
- [ITEM]: [description]
- [ITEM]: [description]

Useful items (helpful but not required):
- [ITEM]: [description]

Filler items:
- [ITEM]: [description]

Trap items:
- [ITEM]: [description]

Please generate a complete items.py file following Archipelago conventions:
- Use a base ID of [YOUR_BASE_ID].
- Include an Item subclass with game = "[GAME NAME]".
- Include an item_table dictionary mapping names to ItemData(code, classification).
- Include item_groups for hint convenience.
- Follow PEP 8 with 120-char line limit and double-quoted strings.
```

### Prompt: Generate Location Definitions

```
I'm creating an Archipelago world for [GAME NAME]. Here are all the locations:

[REGION 1] locations:
- [LOCATION NAME]: [description, how to access]
- [LOCATION NAME]: [description, how to access]

[REGION 2] locations:
- [LOCATION NAME]: [description, how to access]

Please generate a complete locations.py file following Archipelago conventions:
- Use a base ID of [YOUR_BASE_ID].
- Include a Location subclass with game = "[GAME NAME]".
- Include a location_table mapping names to LocationData(code, region).
- Follow PEP 8 with 120-char line limit and double-quoted strings.
```

### Prompt: Generate Options

```
I'm creating an Archipelago world for [GAME NAME]. I want the following player options:

1. [OPTION NAME] - [description] - Type: [Choice/Toggle/Range] - Values: [list values]
2. [OPTION NAME] - [description] - Type: [Choice/Toggle/Range] - Default: [value]
3. [OPTION NAME] - [description] - Type: Range - Min: [X], Max: [Y], Default: [Z]

Please generate a complete options.py file following Archipelago conventions:
- Create a dataclass inheriting from PerGameCommonOptions.
- Include docstrings for each option (these appear on the web).
- Use snake_case for field names.
- Follow PEP 8 with 120-char line limit and double-quoted strings.
```

### Prompt: Generate the World Class

```
I'm creating an Archipelago world for [GAME NAME].

I have already created:
- items.py with item_table and YourGameItem
- locations.py with location_table and YourGameLocation
- options.py with YourGameOptions
- regions.py with region_table and entrance_table

The game regions connect as follows:
- Menu → [REGION 1] (always accessible)
- [REGION 1] → [REGION 2] (requires [ITEM])
- [REGION 2] → [REGION 3] (requires [ITEM 1] and [ITEM 2])

The goal is to [COMPLETION CONDITION].

Please generate a complete __init__.py with:
- A WebWorld subclass with theme, tutorials, and option presets.
- A World subclass with all required hooks (create_regions, create_items,
  set_rules, create_item, fill_slot_data, get_filler_item_name).
- Proper event handling for the completion condition.
- Follow Archipelago conventions and style guide.
```

---

## Phase 3 — Logic and Rules

### Prompt: Generate Access Rules

```
I'm creating Archipelago access rules for [GAME NAME]. Here is my region graph
and the items required to traverse each connection:

Regions: [Menu, Overworld, Forest, Mountain, Castle, Boss Room]

Connections:
- Menu → Overworld: always accessible
- Overworld → Forest: requires Axe
- Overworld → Mountain: requires Climbing Boots
- Forest → Castle: requires Forest Key AND Sword
- Mountain → Castle: requires Mountain Key
- Castle → Boss Room: requires all 3 Boss Medals

Location-specific rules:
- "Forest - Hidden Chest": requires Bomb
- "Mountain - Peak Treasure": requires Climbing Boots AND Rope
- "Boss Room - Final Drop": requires Sword AND Shield

Please generate a complete rules.py file with a set_rules function.
Use lambda functions and the Archipelago Rules API (set_rule, add_rule).
Also set up the completion condition using an event.
```

### Prompt: Verify Logic Completeness

```
Here are my Archipelago access rules for [GAME NAME]:

[PASTE YOUR RULES CODE]

And here are my items classified as progression:
[LIST PROGRESSION ITEMS]

Please verify:
1. Can the game always be completed? Is there a valid path to the goal?
2. Are there any items marked as progression that are never checked in rules?
3. Are there any items checked in rules but NOT marked as progression?
4. Are there potential softlocks or unreachable states?
5. Is sphere-1 access reasonable (enough locations reachable with no items)?
```

---

## Phase 4 — Client Development

### Prompt: Generate a Python Client

```
I'm creating an Archipelago client for [GAME NAME] in Python.

The client needs to:
- Connect to the Archipelago server via WebSocket.
- Interface with the game via [METHOD: memory reading / file watching / mod API / etc.].
- Send location checks when the player [DESCRIPTION OF CHECK TRIGGER].
- Grant items by [DESCRIPTION OF HOW ITEMS ARE APPLIED].
- Detect goal completion when [GOAL DETECTION METHOD].

My slot_data contains: [LIST SLOT DATA FIELDS]

Please generate a client.py using CommonClient as the base class, with:
- A YourGameContext class inheriting from CommonContext.
- server_auth, on_package handlers.
- A game-reading loop that detects location checks.
- Item granting logic.
- Goal completion detection and StatusUpdate.
- Launcher integration component registration.
```

### Prompt: Generate a Non-Python Client Skeleton

```
I'm creating an Archipelago client for [GAME NAME] in [LANGUAGE: C# / Lua / JavaScript].

I need a WebSocket client that implements the Archipelago network protocol:
- Connect to ws://host:port
- Handle RoomInfo, Connected, ReceivedItems, PrintJSON packets
- Send Connect, LocationChecks, StatusUpdate packets
- Maintain a receive index for item synchronization
- Auto-reconnect on disconnect

Please generate a skeleton client with:
- Connection management
- Packet parsing (JSON)
- Handlers for each server packet type
- Methods for sending each client packet type
- A main loop structure
```

---

## Phase 5 — Testing

### Prompt: Generate Unit Tests

```
I'm creating Archipelago unit tests for [GAME NAME].

My world has:
- Regions: [list regions]
- Key progression items: [list items and what they unlock]
- Completion condition: [describe]
- Options: [list options that affect logic]

Please generate a test file using WorldTestBase with:
1. A base test class with game = "[GAME NAME]"
2. Tests verifying each progression item is required for its region/location
3. Tests verifying the game is completable with all items
4. Tests for different option combinations
5. Edge case tests (e.g., minimum items, maximum difficulty)

Follow Archipelago test conventions:
- Use assertAccessDependency, collect_all_but, collect_by_name
- Each test under 1 second
- Descriptive docstrings
```

---

## Phase 6 — Debugging and Troubleshooting

### Prompt: Debug a Fill Error

```
I'm getting this error when generating a seed for my Archipelago world [GAME NAME]:

[PASTE FULL ERROR TRACEBACK]

My world has:
- [X] items and [Y] locations
- Regions: [list]
- Key rules: [describe]

Please help me diagnose and fix this fill error.
```

### Prompt: Debug a Logic Issue

```
My Archipelago world [GAME NAME] generates successfully, but the logic seems wrong:

[DESCRIBE THE ISSUE, e.g., "The Sword is placed behind the boss door, but the boss
requires the Sword to defeat."]

Here are my relevant rules:
[PASTE RULES CODE]

And my item classifications:
[PASTE ITEM TABLE]

Please identify what's wrong and suggest a fix.
```

---

## Phase 7 — Polish and Documentation

### Prompt: Generate Game Info Page

```
Please write an Archipelago game info page (en_[GAME NAME].md) for [GAME NAME].

The game is: [DESCRIPTION]
Randomization affects: [WHAT IS SHUFFLED]
The goal is: [COMPLETION CONDITION]
Items that can appear in other worlds: [LIST]
Unique features of this integration: [LIST]

Follow the format used by other Archipelago game info pages: a brief intro,
sections for "What does randomization do?", "What is the goal?",
"What items can appear in other players' worlds?", and any game-specific notes.
```

### Prompt: Generate Setup Guide

```
Please write an Archipelago setup guide (setup_en.md) for [GAME NAME].

Required software:
- [LIST REQUIRED SOFTWARE AND VERSIONS]

Installation steps:
- [DESCRIBE INSTALLATION PROCESS]

Connection steps:
- [HOW TO CONNECT TO A MULTIWORLD]

The client is: [DESCRIPTION OF CLIENT TYPE - standalone app, game mod, etc.]

Follow the format used by other Archipelago setup guides.
```

---

## General Tips for Using AI with Archipelago Development

1. **Provide context:** Always mention you're working with Archipelago and link to
   relevant docs or existing worlds for the AI to reference.

2. **Iterate:** Don't expect perfect code on the first try. Use the AI to generate a
   skeleton, then refine through conversation.

3. **Verify logic:** AI can generate plausible-looking rules that have subtle bugs.
   Always test with actual generation.

4. **Use AI for boilerplate:** Item tables, location tables, and option definitions are
   great candidates for AI generation.

5. **Review style:** Ask the AI to follow the Archipelago style guide (120-char lines,
   double quotes, PEP 8, type annotations).

6. **Cross-reference:** When the AI generates code, compare it against a working world
   like `worlds/yachtdice/` or `worlds/apsudoku/` to verify patterns.
