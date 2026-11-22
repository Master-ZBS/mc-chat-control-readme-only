# TwitchMod

A Minecraft mod that lets Twitch chat (and people in-game) add all sorts of ridiculous things to Minecraft using AI.

Start your messages with `!` to tell the mod that you want something added or changed.

It then sends your message to an AI, which converts it into Lua code and JSON that the mod uses.

For example:

User: `!make an item that makes mobs disappear when you right-click them`

The AI can make an item that removes mobs without killing them, with particles and sounds when they disappear. It can also add a crafting recipe for the item.

You can also change things that already exist.

User: `!make the disappearing item spawn black particles instead`

The AI can modify the existing item or its code without having to make a completely new one.

Oh, and the AI is named Conner.

# Things it can do

Some things it can do (in no particular order):

- Make new items (up to 500)
- Give items custom names, descriptions and rarities
- Use existing Minecraft and modded textures for items
- Give items special abilities
- Make crafting recipes
- Change existing recipes, including vanilla and modded ones
- Remove and restore recipes
- Spawn mobs
- Remove mobs without killing them
- Damage and heal entities
- Teleport things
- Launch things into the air
- Change entity sizes (with Pehkui)
- Spawn particles
- Play sounds
- Give potion effects
- Run Minecraft commands
- Make temporary events
- Make permanent changes
- Modify things it's already made
- Make things explode
- Probably more stuff

# How it works

The mod has a bunch of functions that the AI can use to interact with Minecraft.

It can generate Lua code for events, create items and add recipes. All of this gets stored in JSON files in the world folder.

Items and recipes can be added or changed without restarting Minecraft or using `/reload`.

The mod has 500 item slots that are already registered, so the AI can turn an unused slot into a new item whenever it wants.

Items are permanent, but events can be temporary.

The AI can also run an event immediately, so you can ask it to do something like start a TNT storm without needing an item for it.

### World context and history

TwitchMod provides the AI with information about the current Minecraft world, installed mods, existing custom content, and previous actions.

This information is collected from several JSON files:

- **`world_data.json`** — Contains information about the current world and its state.
- **`instance.json`** — Contains information about the Minecraft instance, including installed mods and their versions. the AI can use this to determine which mods are available.
- **`items.json`** — Stores definitions of custom items created through TwitchMod.
- **`recipes.json`** — Stores custom recipes, recipe replacements, and removals.
- **`events.json`** — Stores registered Lua events that can be triggered during gameplay.
- **`event_history.json`** — Records previous actions, including changes to custom content and event triggers.

These files allow the AI to understand what has already been created or modified without relying entirely on the viewer's message.

#### How the AI uses this information

When processing a request, the AI should consider the current world definitions before generating new content. For example, if a viewer asks to modify an existing recipe, the AI can identify the recipe through `recipes.json` rather than creating an unrelated one.

The history file provides additional context about previous activity. Unlike the definition files, it records what happened rather than describing what currently exists.

Historical entries are informational and should not be interpreted as instructions to repeat previous actions.

The data represents the state captured when the request was prepared. It may not include every live entity, block, inventory, or gameplay change occurring in Minecraft.

### Conversational requests

Not every message beginning with `!` requests a modification to Minecraft. Some messages may be questions, greetings, or simple tests of the AI connection.

TwitchMod handles these messages using the same JSON and Lua event system as other requests. The AI does not return plain-text responses directly. Instead, it can create an event that performs an appropriate action, such as displaying a message in Minecraft chat.

For example, if a viewer asks whether the AI is working, the AI can generate a temporary Lua event that responds with "Yes!" and select that event using `run_event`.

Every response must be a valid JSON object containing `events`, `items`, and `recipes` arrays, with an optional `run_event` field.

For conversational requests:

- Create an event containing the Lua code needed to respond.
- Use the supported Lua API to display the message or perform another appropriate action.
- Set `run_event` to the event's ID so it executes immediately.
- Use the appropriate event lifetime or cleanup mechanism for one-time responses, rather than leaving unnecessary permanent events registered.

Each entry in the JSON arrays must follow the required format, including its `id` and `action` fields.

For example, a request to spawn five zombies can create an event that spawns them, while a request saying "Hello!" can create an event that replies in Minecraft chat. Both use the same JSON response structure and execution system.

The actual Lua code must use functions supported by TwitchMod's Lua API. The AI should not invent functions or generate raw Minecraft commands where an appropriate Lua API function already exists.

# Multiplayer

The mod works on multiplayer servers too.

The server handles the Lua code and sends item information to connected players, so everyone can see and use the generated items.

Everyone needs the mod installed, but they don't need to download newly created items separately.

# Limitations

The AI doesn't actually write new Java classes or generate new textures. Instead, it works with the functions and textures that already exist in Minecraft and TwitchMod.

So it can't do literally anything, but there's a lot of stuff it can do.
