![Banner](https://i.imgur.com/bH2Yuz6.png)

## About

**mta-add-models** is a MTA resource that acts as a library, making use of [engineRequestModel](https://wiki.multitheftauto.com/wiki/EngineRequestModel) function to add new models
- syncs all added models with all players
- minimalistic, optimized and bug free

More info:
- MTA Forum thread: [here](https://forum.mtasa.com/topic/133212-rel-add-new-models-library/#comment-1003395)
- Contact (author): **Discord** Nando#7736

## Supported Types

- [x] players
- [x] peds
- [x] objects
- [ ] vehicles  *[coming soon]*

## Commands

- /listmods **lists all defined mods**
- /allocatedids **shows all allocated mod IDs in realtime**
- /selements **lists all streamed in elements for debugging purposes**
- /pedskin [ID] **creates a ped and sets their skin to a default or new ID**
- /myskin [ID] **sets your skin to a default or new ID**

## Quick Tutorial

- place mod files newmodels/models (dff & txd)
- list them in meta.xml like the example
- define them in shared.lua inside modList like the example
- use the commands to test, have fun!

## Implementing

Click [here](#gamemode-implementations) for specific gamemode implementations like OwlGaming.

### (❗) Explanation

- This applies to elements created **serverside** with the following functions:
  - `createPed` (use a placeholder ID when creating, e.g. 1)
  - `createObject` (use a placeholder ID when creating, e.g. 1337)
  - `createVehicle` (use a placeholder ID when creating, e.g. 400)
  - `spawnPlayer` (use a placeholder ID when spawning the player, e.g. 1)
- After creating these elements, you have to:
  - Fetch element data name from this resouce using `getDataNameFromType(elementType)`
  - Set their model ID via element data with the name you just obtained
  - (Optional) Check if model ID is custom using `isDefaultID(elementType, modelID)` and `isCustomModID(elementType, modelID)`
  - (Optional) Fetch element model name, if custom, using `getModNameFromID(elementType, modelID)`
- This resource makes the clients listen to the set element datas in order to apply model IDs accordingly on their game
- **DO NOT USE** `setElementModel` or `getElementModel` **ANYMORE** in **serverside** scripts for setting/getting model IDs
- **See examples below** to understand how what's been described can be put in place.

## Lua Examples

### Example #1

(**serverside**) Spawning a ped with a new skin ID:

**Before you would use setElementModel serverside, with clientside-set models you can't**
```lua
local skin = 20001 -- valid modded skin ID that you defined
local ped = createPed(0, 0,0,5) -- creates a ped in the center of the map; skin ID 0 is irrelevant
if ped then
   -- setElementModel(ped, skin) -- bad!
   local data_name = exports.newmodels:getDataNameFromType("ped") -- gets the correct data name
   setElementData(ped, data_name, skin) -- sets the skin ID data; clients listening for this data will apply their corresponding allocated model ID on the created ped
end
```

### Example #2

(**serverside**) Spawning an object with a new model ID:

**Before you would use setElementModel serverside, with clientside-set models you can't**
```lua
local model = 50001 -- valid modded object ID that you defined
local object = createObject(1337, 0,0,8) -- creates an object in the center of the map; model ID 1337 is irrelevant
if object then
   -- setElementModel(object, model) -- bad!
   local data_name = exports.newmodels:getDataNameFromType("object") -- gets the correct data name
   setElementData(object, data_name, model) -- sets the model ID data; clients listening for this data will apply their corresponding allocated model ID on the created object
end
```

### Example #3

(**serverside**) Spawning a player after login and setting their skin ID:

**Before you would use setElementModel serverside, with clientside-set models you can't**
```lua
-- TODO: fetch player data from database, here we use static values:
local x,y,z = 0,0,5
local rx,ry,rz = 0,0,0
local int,dim = 0,0
local skin = 20001

spawnPlayer(thePlayer, x,y,z, 0, 0, int, dim) -- spawns the player in the center of the map; skin ID 0 is irrelevant
setElementRotation(thePlayer,rx,ry,rz)

-- setElementModel(thePlayer, skin) -- bad!
local data_name = exports.newmodels:getDataNameFromType("player") -- gets the correct data name
setElementData(thePlayer, data_name, skin) -- sets the skin ID data; clients listening for this data will apply their corresponding allocated model ID on the player
```

### Example #4

(**serverside**) Saving a player's skin ID on disconnect:

**Before you would use getElementModel serverside, with clientside-set models you can't**
```lua
addEventHandler( "onPlayerQuit", root, 
  function (quitType, reason, responsibleElement)
    -- local skin = getElementModel(source) -- bad!
    local data_name = exports.newmodels:getDataNameFromType("player") -- gets the correct data name
    local skin = getElementData(source, data_name)
    if skin then
    	-- TODO: save skin ID in the database
    end
  end
)
```

## Gamemode Implementations

### [OwlGaming Gamemode](https://github.com/OwlGamingCommunity/MTA) - Custom Peds

In order to have custom new skins, you're going to have to search through all the scripts for **serverside** instances of:
- **getElementModel** `replace with setElementData -> skinID`
- **setElementModel** `replace with getElementData -> skinID`
- **createPed** `use 0 as skinid; use setElementData -> skinID afterwards`
- **spawnPlayer** `use 0 as skinid; use setElementData -> skinID afterwards`

See the proper implementation examples below, they're helpful.

Example scripts that you need to adapt:
- Clientside peds in character selection screen [account/c_characters.lua](https://github.com/OwlGamingCommunity/MTA/blob/main/mods/deathmatch/resources/account/c_characters.lua)
- Serverside character spawning [account/s_characters.lua](https://github.com/OwlGamingCommunity/MTA/blob/main/mods/deathmatch/resources/account/s_characters.lua)
- Serverside player skin saving [saveplayer-system/s_saveplayer_system.lua](https://github.com/OwlGamingCommunity/MTA/blob/main/mods/deathmatch/resources/saveplayer-system/s_saveplayer_system.lua)

For new skin images used in the inventory, place them in [account/img](https://github.com/OwlGamingCommunity/MTA/tree/main/mods/deathmatch/resources/account/img) as ID.png with a minimum of 3 digits.

You will find a lot more scripts that need to be changed if you want to use new IDs to the maximum potential, for example:
- Shops/NPCs having custom skin IDs
- Supporting custom skins in the [clothes](https://github.com/OwlGamingCommunity/MTA/tree/main/mods/deathmatch/resources/clothes) system
- ...

## Final Note

Feel free to update this README.md if you wish to provide tutorial(s) for other implementations, or generally improve the current documentation.

Thank you for reading, have fun!