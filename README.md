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

The AI receives information about the current world, existing custom content, installed mods, and previous TwitchMod actions. This includes `world_data.json`, `instance.json`, `items.json`, `recipes.json`, `events.json`, and `event_history.json`, along with the available TwitchMod texture IDs. World files describe the saved state, while history describes what happened previously; neither is an instruction to repeat an action. The snapshot may not describe every live entity or block.

### Conversational requests

Every request produces structured JSON. A greeting or question can be handled by a one-shot Lua event that uses `basicChat` or `sendPlayerMessage`, then identifies that event with `run_event`. This is different from returning a plain-text reply. The event is registered before it is run; `run_event` itself does not delete the registered event definition.

### Guide updates

TwitchMod downloads this README from GitHub on server startup and caches it in the user's `.twitchmod/conner_guide.md`. If the download fails it uses the cache, or the bundled `offline_guide.md` if no cache is available. A `guide-url.txt` file under `.twitchmod` can override the packaged GitHub URL. API credentials are stored outside the Minecraft instance.

# Multiplayer

The mod works on multiplayer servers too.

The server handles the Lua code and sends item information to connected players, so everyone can see and use the generated items.

Everyone needs the mod installed, but they don't need to download newly created items separately.

# Limitations

The AI doesn't actually write new Java classes or generate new textures. Instead, it works with the functions and textures that already exist in Minecraft and TwitchMod.

So it can't do literally anything, but there's a lot of stuff it can do.

# Lua API and function reference

This section documents the Lua functions exported by the **current Java source** of TwitchMod (Forge 1.20.1). It is both a reference for developers and a guide to generating compatible events. All names are case-sensitive. Optional arguments are shown in `[square brackets]` in the signatures below; the brackets are not Lua syntax.

**Important conventions:** `Vector3(x, y, z)` produces a coordinate/vector table with `.x`, `.y`, and `.z`. `player`, `entity`, `item`, and `block` below mean *Lua references returned by the corresponding APIs*, not their names or Minecraft IDs. For IDs, use registry strings such as `minecraft:zombie` and `minecraft:diamond`. Many lookups return `nil` if nothing is found. The command executor and its dimension matter for some functions. The source code remains the ultimate authority if the mod has been changed since this guide was written.

## Chat and commands (`LuaChatAPI`, `LuaCommandAPI`)

| Lua function | What it does |
|---|---|
| `basicChat(message)` | Broadcasts a plain-text system message to all players and logs it. |
| `print(message)` | Sends a message to the command executor rather than broadcasting. |
| `rawCommand(command)` | Runs a Minecraft command as the execution source; accepts an optional `/` prefix and returns its command result. |

Use `basicChat("Hello!")` for a greeting intended for everyone, or `sendPlayerMessage(player, "Hello!")` for a specific player. `rawCommand` is powerful and inherits the command source's permissions; it is not an unrestricted substitute for the Lua APIs.

## Vectors and general utilities (`LuaVector3API`, `LuaUtilitiesAPI`)

| Lua function | What it does |
|---|---|
| `Vector3(x, y, z)` | Creates a vector with `x`, `y`, and `z` numeric components. |
| `wait(seconds)` | Pauses sequential execution when used as a standalone supported Lua statement. |
| `waitTicks(ticks)` | Pauses sequential execution for Minecraft ticks. |
| `wait(seconds, callback)` | Schedules a callback after a delay (0–3600 seconds). |
| `waitTicks(ticks, callback)` | Schedules a callback after a delay (0–72000 ticks). |
| `playSound(soundId, position, volume, pitch [, category])` | Plays a registered sound around a `Vector3` in the executor's dimension. |
| `playSoundAtEntity(entity, soundId, volume, pitch [, category])` | Plays a registered sound at an entity in its dimension. |
| `spawnParticle(particleId, position, count)` | Spawns simple particles (1–200); types requiring extra data are unsupported. |
| `spawnDustParticle(position, red, green, blue, size, count)` | Spawns custom-colour dust particles; RGB range 0–1, size 0.01–4, count 1–200. |
| `runEvent(id)` | Requests execution of a previously registered Lua event by its ID. Circular calls are prevented. |

Sequential waits are processed by TwitchMod's Lua sequential-wait system; when in doubt, use the callback form. For one-time execution of an event immediately after an AI response, prefer the JSON `run_event` field rather than invoking `runEvent` while registering an event.

## Players (`LuaPlayerAPI`)

| Lua function | What it does |
|---|---|
| `getPlayer(name)` | Returns an online player reference, or `nil`. |
| `getPlayers()` | Returns an array of online player references. |
| `isPlayerOnline(name)` | Returns whether a player with that name is online. |
| `getPlayerName(player)` | Gets the player's name. |
| `isPlayerSneaking(player)` | Checks whether the player is sneaking. |
| `getPlayerLookVector(player)` | Gets the player's look direction as `Vector3`. |
| `getEntityDimension(entity)` | Gets the dimension ID of an entity (also works with player references). |
| `givePlayerItem(player, itemId, count)` | Gives a registered item; excess may drop at the player's feet. |
| `getPlayerItemCount(player, itemId)` | Counts an item in the player's inventory. |
| `removePlayerItem(player, itemId, count)` | Removes up to the specified count; returns the amount removed. |
| `getPlayerHeldItem(player)` | Gets the item ID currently held, or `nil` for empty hand. |
| `sendPlayerMessage(player, text)` | Sends a chat/system message to a specific player. |
| `getPlayerFood(player)` | Reads food level. |
| `setPlayerFood(player, amount)` | Sets food level. |
| `getPlayerXPLevel(player)` | Reads experience level. |
| `addPlayerXP(player, points)` | Adds or subtracts experience points. |
| `getPlayerGameMode(player)` | Gets game mode (such as `survival`, `creative`). |
| `setPlayerGameMode(player, mode)` | Changes the player's game mode. |

Player references are obtained from `getPlayer` or `getPlayers`; do not pass a name string into functions expecting a player reference.

## Entities (`LuaEntityAPI`)

| Lua function | What it does |
|---|---|
| `getEntity(uuid)` | Finds a loaded entity by UUID. |
| `getEntities()` | Lists loaded entities in the executor's dimension. |
| `getEntitiesOfType(typeId)` | Lists loaded entities matching a registry entity type. |
| `getEntitiesOfName(name)` | Lists loaded entities matching an exact custom name. |
| `findNearestEntity(position, radius [, typeId])` | Finds the nearest matching entity around a vector. |
| `findNearestEntityOfName(position, radius, name)` | Finds nearest entity with an exact custom name. |
| `entityExists(entity)` | Checks whether an entity reference is still resolvable. |
| `getEntityUUID(entity)` | Gets entity UUID. |
| `getEntityType(entity)` | Gets registry ID such as `minecraft:zombie`. |
| `getEntityName(entity)` | Gets displayed entity name. |
| `getEntityCustomName(entity)` | Gets custom name, if present. |
| `setEntityCustomName(entity, name)` | Sets an entity's custom name. |
| `getEntityPosition(entity)` | Gets its position as a vector. |
| `setEntityPosition(entity, position)` | Moves/teleports it to a vector position, subject to world checks. |
| `getEntityVelocity(entity)` | Reads velocity as a vector. |
| `setEntityVelocity(entity, velocity)` | Changes entity movement. |
| `getEntityRotation(entity)` | Returns a table with `.yaw` and `.pitch`. |
| `setEntityRotation(entity, yaw, pitch)` | Changes entity orientation. |
| `getEntityHealth(entity)` | Gets health of a living entity. |
| `setEntityHealth(entity, health)` | Sets health of a living entity. |
| `getEntityMaxHealth(entity)` | Gets maximum health. |
| `healEntity(entity, amount)` | Heals a living entity. |
| `damageEntity(entity, amount)` | Damages an entity. |
| `killEntity(entity)` | Kills an entity. |
| `removeEntity(entity)` | Removes an entity without necessarily killing it. |
| `setEntityOnFire(entity, seconds)` | Sets entity on fire. |
| `getEntityFireTicks(entity)` | Reads remaining fire ticks. |
| `addEntityEffect(entity, effectId, durationTicks, amplifier)` | Adds a potion/mob effect (duration 1–72000 ticks, amplifier 0–255). |
| `removeEntityEffect(entity, effectId)` | Removes an effect. |
| `spawnEntity(typeId, position)` | Spawns a registered entity at a vector position. |
| `getEntityDistance(entityA, entityB)` | Computes distance between two entities. |
| `getEntityTarget(entity)` | Gets a mob's current target, if applicable. |
| `setEntityTarget(entity, target)` | Assigns a mob target. |
| `getLookingAtEntity(player [, range])` | Returns an entity a player is looking at, if within range. |

Entities are represented by lightweight UUID-backed references. `getEntitiesOfName` checks *custom names*, whereas `getEntitiesOfType` checks registry IDs.

## Entity reference properties (`LuaEntityProperties`)

Entity references also support live property access, such as `entity.position` and `entity.health`. These are not additional global functions.

| Readable property | Notes |
|---|---|
| `uuid`, `type`, `name` | Identity, registry type and display name. |
| `position`, `velocity` | `Vector3` values. |
| `health`, `maxHealth` | Available for living entities. |
| `yaw`, `pitch`, `onFire` | Orientation and burning state. |
| `username`, `isSneaking`, `lookDirection` | Player-specific values. |
| `dimension`, `heldItem` | Dimension ID and player-held item reference. |
| `food`, `saturation`, `experienceLevel` | Player-specific stats. |

Writable properties include `name`, `position`, `velocity`, `yaw`, `pitch`, `onFire`, `food`, `saturation`, and `experienceLevel`, with type/range checks. Other properties are read-only.

## Blocks and redstone (`LuaBlockAPI`, `LuaRedstoneAPI`)

| Lua function | What it does |
|---|---|
| `getLookingAtBlock(player, range)` | Gets a loaded block targeted by the player (range 0.1–32). |
| `getBlock(dimensionId, position)` | Gets a loaded block reference from a dimension and vector. |
| `getBlockAt(dimensionId, x, y, z)` | Gets a loaded block reference from separate coordinates. |
| `canActivateRedstone(block)` | Checks whether a referenced block supports the API's redstone activation. |
| `activateRedstone(block, player)` | Activates a supported vanilla button/lever as that player. |

Block references are **read-only** and expose `.id`, `.position`, `.x`, `.y`, `.z`, `.dimension`, and `.isAir`. The block API does not load unloaded chunks. `activateRedstone` is restricted to supported blocks and is not a replacement for Forge's full interaction/claim permission checks.

## Item stacks, inventories and NBT (`LuaItemAPI`)

These functions manipulate **item stack references**. They do not register a new dynamic-item slot: new permanent item definitions are created through the AI response's `items` array.

| Lua function | What it does |
|---|---|
| `createItem(itemId [, count])` | Creates an item-stack reference for a registered item. |
| `getHeldItem(player [, hand])` | Gets held stack (`offhand` optional), or `nil`. |
| `getInventoryItem(player, index)` | Gets a slot reference; inventory indices 0–40. |
| `setInventoryItem(player, index, item)` | Copies the referenced stack into an inventory slot. |
| `giveItem(player, item)` | Gives a copy of a stack, dropping overflow. |
| `getItemId(item)` | Gets the registered item ID. |
| `getItemCount(item)` | Gets stack count. |
| `getItemMaxCount(item)` | Gets maximum stack count. |
| `setItemCount(item, count)` | Changes count. |
| `getItemDamage(item)` | Gets current durability damage. |
| `getItemMaxDamage(item)` | Gets maximum damage. |
| `setItemDamage(item, damage)` | Changes durability damage. |
| `getItemName(item)` | Gets displayed item name. |
| `setItemName(item, name)` | Sets custom item name. |
| `clearItemName(item)` | Clears custom name. |
| `getItemNBT(item [, path])` | Gets NBT (or a specific nested value) in Lua representation. |
| `getItemNBTString(item [, path])` | Gets full or partial SNBT text. |
| `getItemNBTType(item, path)` | Gets the type of a particular NBT value. |
| `hasItemNBT(item, path)` | Checks whether an NBT path exists. |
| `setItemNBT(item, path, value [, type])` | Sets an NBT key, with optional explicit NBT type. |
| `addItemNBT(item, path, value [, type])` | Sets a key only if it does not already exist. |
| `removeItemNBT(item, path)` | Removes an NBT key. |
| `addItemNBTNumber(item, path, delta)` | Increments a numeric NBT value. |
| `mergeItemNBT(item, snbt)` | Merges SNBT compound text into stack data. |
| `getItemNBTKeys(item [, path])` | Gets keys of an NBT compound. |

Stack references expose readable `.id`, `.name`, `.count`, `.maxCount`, `.damage`, `.maxDamage`, and `.nbt`. Writable properties include `.name`, `.count`, `.damage` and nested `.nbt` keys. NBT type details matter: use `type = "snbt"` for explicit SNBT structures when a Lua map would lose type information.

## Hooks (`LuaHookAPI`)

| Lua function | What it does |
|---|---|
| `hook.Add(event, name, callback)` | Registers a named callback on a supported event; returns a hook connection. |
| `hook.Remove(event, name)` | Removes a previously registered named hook. |
| `hook.GetNames(event)` | Lists names of callbacks registered for an event. |

Hooks are managed by TwitchMod's Lua hook manager; their event names and callback arguments are defined by the hook system. Do not invent hook event names. A connection may expose a `Disconnect()` mechanism.

## Mod integration and Pehkui (`LuaModAPI`, `LuaPehkuiAPI`)

| Lua function | What it does |
|---|---|
| `isModLoaded(modId)` | Returns whether an installed Forge mod is loaded. |
| `isPehkuiLoaded()` | Returns whether Pehkui is installed. |
| `getEntityScale(entity [, type])` | Reads the current Pehkui scale. |
| `getEntityBaseScale(entity [, type])` | Reads base scale. |
| `getEntityTargetScale(entity [, type])` | Reads target scale. |
| `setEntityScale(entity, value [, type])` | Immediately changes scale. |
| `animateEntityScale(entity, target, ticks [, type])` | Animates scale over 1–1200 ticks. |
| `resetEntityScale(entity [, type])` | Resets scale to 1. |
| `setEntityScalePersistent(entity, enabled [, type])` | Sets persistence behavior when supported by Pehkui. |

Only `isPehkuiLoaded` is available when Pehkui is absent. Scale functions require Pehkui and a supported scale type.

## AI response JSON format

An AI reply is **one JSON object**, not Markdown or plain text. It contains three arrays, plus optional `run_event`:

```json
{
  "events": [],
  "items": [],
  "recipes": [],
  "run_event": null
}
```

Each operation requires an `id` and `action`. The supported actions are `add` and `remove`; recipes additionally support `restore`. Removing dynamic items is forbidden. `run_event` is either `null` or one event ID matching `[a-z0-9_-]{1,64}`. It runs once *after* definitions are applied; the event definition itself remains registered unless separately removed. The interpreter records applied changes in world history.

### Events

An event `add` operation requires `id`, `action`, `name`, and `code`. The `code` is a Lua string or an array of Lua source lines. Example of a one-shot conversational event:

```json
{
  "events": [{
    "id": "say_hello",
    "action": "add",
    "name": "Say hello",
    "code": "basicChat('Hello!')"
  }],
  "items": [],
  "recipes": [],
  "run_event": "say_hello"
}
```

This registers the event and runs it once. It does **not** automatically delete `say_hello` afterward. For ephemeral responses, use unique IDs and arrange for cleanup using a supported operation. Avoid registering persistent hooks merely to answer a greeting.

### Items

Item IDs must use the available slot format, for example `twitchchatcontrolmod:dynamic_item_001` through `_500`. The `add` operation requires `id`, `action`, `name`, `description`, `rarity`, and `texture`. Item slots are permanent and should be allocated sequentially without reusing an already assigned ID. Use resource IDs supplied in the available-texture context. Item behavior may be configured by other fields supported in `DynamicItemManager` and `ItemData`; don't assume unsupported fields will work merely because JSON validation accepts them.

### Recipes

Recipes are stored by ID and can override vanilla or modded recipes using their existing IDs. For `add`, include the `type` (the registered Minecraft recipe serializer) and its serializer-specific fields. For `remove`, provide the ID and `action: "remove"`. For `restore`, provide the ID and `action: "restore"`; this removes TwitchMod's override/removal marker so the original recipe can be restored when its snapshot exists.

Example of removing and later restoring a recipe:

```json
{
  "events": [],
  "items": [],
  "recipes": [{"id":"minecraft:diamond_sword", "action":"restore"}],
  "run_event": null
}
```

### Validating generated responses

Prefer existing IDs when editing existing content. Do not invent functions or registry identifiers; the available Lua APIs and the context files are the supported interface. All events must be well-formed; conversation is handled by Lua events rather than by an extra plain-text answer field. Check `instance.json` before relying on another mod, and treat `event_history.json` as historical context, not an instruction stream. TwitchMod may apply AI output automatically, so avoid unintended destructive actions.

## Practical Lua examples

A message to all players:

```lua
basicChat("Hello from TwitchMod!")
```

A message to one online player:

```lua
local player = getPlayer("Dev")
if player ~= nil then
    sendPlayerMessage(player, "Hello!")
end
```

Spawn a zombie near a known player:

```lua
local player = getPlayer("Dev")
if player ~= nil then
    local p = getEntityPosition(player)
    spawnEntity("minecraft:zombie", Vector3(p.x + 2, p.y, p.z))
end
```

Display red dust particles:

```lua
local player = getPlayer("Dev")
if player ~= nil then
    spawnDustParticle(getEntityPosition(player), 1, 0, 0, 1, 20)
end
```

These examples demonstrate the API's shape. A viewer's player name, installed mods, entity types, and available textures should come from request/world context rather than being hardcoded as `Dev` in general-purpose generated events.
