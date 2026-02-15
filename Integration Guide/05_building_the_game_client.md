# 05 — Building the Game Client

The game client is the bridge between the running game and the Archipelago server.
It can be a standalone program, a game mod, or a combination of both.

## Overview

```
┌──────────────┐      WebSocket       ┌─────────────────────┐
│  Your Game   │◄────────────────────►│  Archipelago Server  │
│  (or mod)    │   JSON packets       │  (MultiServer.py)    │
└──────────────┘                      └─────────────────────┘
       ▲
       │  Game memory / API / file I/O
       ▼
┌──────────────┐
│  AP Client   │  (Python, Lua, C#, JavaScript, etc.)
└──────────────┘
```

## Hard Requirements

Your client **must**:

1. Handle both **secure** (`wss://`) and **insecure** (`ws://`) WebSocket connections.
2. **Reconnect** automatically if the connection drops.
3. Allow the user to **change the port** for saved connection info.
4. Send a **StatusUpdate** packet when the player completes their goal.
5. **Send LocationChecks** when the player checks a location in-game.
6. **Receive and apply items** from `ReceivedItems` packets at any time.
7. Handle **duplicate items** — any item can be received any number of times.
8. Handle items received from **admin commands** (no player/location attribution).
9. Maintain a **receive index** to resync after reconnection.
10. Deliver items that were sent **while the player was offline**.

## Connection Flow

```
Client                          Server
  │                                │
  │──── WebSocket connect ────────►│
  │◄─── RoomInfo ─────────────────│
  │                                │
  │──── GetDataPackage (optional)─►│
  │◄─── DataPackage ──────────────│
  │                                │
  │──── Connect {game, name, ...}─►│
  │◄─── Connected / ConnectionRefused
  │                                │
  │◄─── ReceivedItems (queued) ───│
  │◄─── PrintJSON ────────────────│
  │                                │
  │──── LocationChecks ───────────►│  (player checks a location)
  │◄─── ReceivedItems ────────────│  (player receives an item)
  │                                │
  │──── StatusUpdate (goal) ──────►│  (player finishes)
```

## Writing a Python Client

Archipelago provides `CommonClient.py` as a base. Here is a minimal example:

```python
# worlds/your_game/client.py

import asyncio
from CommonClient import (
    CommonContext,
    server_loop,
    get_base_parser,
    gui_enabled,
)
import Utils


class YourGameContext(CommonContext):
    game = "Your Game"
    command_processor = ...  # optional
    items_handling = 0b111   # receive all items (local + remote)

    def __init__(self, server_address, password):
        super().__init__(server_address, password)
        # your custom state here
        self.received_items_count = 0

    async def server_auth(self, password_requested: bool = False):
        """Called when the server asks for authentication."""
        if password_requested and not self.password:
            await super().server_auth(password_requested)
        await self.get_username()
        await self.send_connect()

    def on_package(self, cmd: str, args: dict):
        """Handle incoming packets."""
        if cmd == "Connected":
            # slot_data is available in args["slot_data"]
            slot_data = args.get("slot_data", {})
            print(f"Connected! Difficulty: {slot_data.get('difficulty')}")

        elif cmd == "ReceivedItems":
            # Process items
            for item in args["items"]:
                item_id = item.item
                item_name = self.item_names.lookup_in_game(item_id)
                print(f"Received: {item_name}")
                # TODO: grant item in-game
                self.received_items_count += 1

    def run_gui(self):
        """Launch the Kivy-based tracker/GUI (optional)."""
        from kvui import GameManager
        class YourGameManager(GameManager):
            logging_pairs = [("Client", "Archipelago")]
            base_title = "Your Game Client"
        self.ui = YourGameManager(self)
        self.ui_task = asyncio.create_task(self.ui.async_run(), name="UI")


async def main():
    parser = get_base_parser()
    args = parser.parse_args()

    ctx = YourGameContext(args.connect, args.password)
    ctx.server_task = asyncio.create_task(server_loop(ctx), name="Server")

    if gui_enabled:
        ctx.run_gui()

    await ctx.exit_event.wait()
    await ctx.shutdown()


def run_client(*args: str):
    """Entry point for the Launcher."""
    Utils.init_logging("YourGameClient")
    asyncio.run(main())


if __name__ == "__main__":
    run_client()
```

## Sending Location Checks

When the player checks a location in-game, send the location IDs:

```python
# Inside your game-reading loop:
checked_location_ids = [loc_id for loc_id in newly_checked]
await ctx.send_msgs([{
    "cmd": "LocationChecks",
    "locations": checked_location_ids,
}])
```

## Sending Goal Completion

```python
from NetUtils import ClientStatus

await ctx.send_msgs([{
    "cmd": "StatusUpdate",
    "status": ClientStatus.CLIENT_GOAL,
}])
```

## Writing a Non-Python Client

You can write clients in any language with WebSocket support. Use the
[Network Protocol](/docs/network%20protocol.md) document as your reference.

Popular choices:
- **Lua** — for emulator-based mods (BizHawk, RetroArch).
- **C#** — for Unity game mods.
- **JavaScript** — for web-based games.
- **C / C++** — for native game hooks.

### Minimum Packet Handling

| Direction | Packet | Purpose |
|-----------|--------|---------|
| S→C | `RoomInfo` | Server version, seed info. |
| C→S | `Connect` | Authenticate with game name, slot name, password. |
| S→C | `Connected` | Player list, slot data, missing locations. |
| S→C | `ReceivedItems` | Items to grant. Track `index` for resync. |
| C→S | `LocationChecks` | Report checked locations. |
| C→S | `StatusUpdate` | Report goal completion. |
| S→C | `PrintJSON` | Chat and notification messages. |

## Launcher Integration

Register your client in `worlds/your_game/__init__.py`:

```python
from worlds.LauncherComponents import Component, components, Type, launch_subprocess

def launch_client(*args):
    from .client import run_client
    launch_subprocess(run_client, name="YourGameClient", args=args)

components.append(
    Component(
        "Your Game Client",
        func=launch_client,
        component_type=Type.CLIENT,
        game_name="Your Game",
    )
)
```

## Testing Your Client Locally

1. Create a YAML for your game and place it in `Players/`.
2. Run `python Generate.py` to create a multiworld.
3. Run `python MultiServer.py <output_file>` to start a local server.
4. Run your client and connect to `localhost:<port>`.
5. Test location checks and item receives.

> **Tip:** Use `MultiServer.py --log_network` to see all packets in the server log.

## Next Step

Proceed to [06 - Testing Your Integration](06_testing_your_integration.md) to learn how
to write and run automated tests.
