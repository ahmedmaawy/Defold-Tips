# About

These tips originate from the [Planet Zero](https://github.com/benjames-171/defold-games/tree/master/Planet%20Zero) Game by Ben James.

I find Ben Jame's Platformer games as a source of inspiration in terms of how platformer games by default can be structred. So I am documenting one of these games and what I have learnt from its structure so that other can emulate them.

## Game Initialization

The `/main/main.collection` is where all the action is. This collection contains:

* All audio file references that are used throughout the game, even within nested collections that are spawned. They are all nested in the `sound` game object.
* The `handler` Game object, that loads the Game Collection or Menu Collection, co-ordinated by the `handler.script` file. Although it loads the `game/core/game.collection` collection, that collection contains 2 more proxies to load `level1` or `level2`, which are in the `game/levels` folder.

## Shared Library

The game library `/main/data.lua` is used to maintain game state. It also contains various shared functions that are relevant to the game, for instance:

* All variables in the file are used to keep track of game state. Its usually the case when you see `local data = require "main.data"` in any file.
* `world2tile` and `tile2world` are used to convert tile coordinates to world coordinates and vice versa.
* `loadgamefile` and `savegamefile` are used to load and save game state, however I am not sure the functionalities were implemented, yet.
* Other relevant utility functions.

## How Levels work

Each level is stored as a collection under `game/levels` and all have a uniform structure. In that:

* They all load the `game/core/common` collection, which contains common elements that are common to each level.
* They have a `level` game object that is then also uniform in structure to each level. They need to be uniform in structure because they need to work within the context of the `level.script` file and load the `tilemap` object. However, the `tilemap` in level1 and level2 load different tilemap files.

How these tilemaps are populated in the game is topic of next section:

## Populating Tilemaps

Ben James has a special way of populating elements in a tile map. He iterates the entire tilemap, and instead then replaces relevant tiles with actual game objects. As so:

```lua
-- Get the Tilemap Bounds

sx, sy, w, h = tilemap.get_bounds("#tilemap")

-- Iterate X and Y axis of the tilemap and populate elements

for y = sy, h+sy-1 do
	for x = sx, w+sx-1 do
		
		-- Get the type of the tile
		
		local t = tilemap.get_tile("#tilemap", "world", x, y)

		-- Check against which types of tile it is. Then reset the tile type and set it to
		-- default tile index (0) and instead, spawn the relevant game object in its place

		if t == 1 then

			-- Special Case:
			-- If its a player, don't spawn the player, instead, place the player
			-- at the relevant location
			
			tilemap.set_tile("#tilemap", "world", x, y, 0)
			msg.post("common/player", "position", {pos = data.tile2world(vmath.vector3(x, y, 0.5))})
		elseif t == 2 then
			factory.create("#sentry_factory", data.tile2world(vmath.vector3(x, y, 0.2)))
		elseif t == 7 then
			tilemap.set_tile("#tilemap", "world", x, y, 0)
			factory.create("#bug_factory", data.tile2world(vmath.vector3(x, y, 0.2)))
		elseif t == 14 then
			tilemap.set_tile("#tilemap", "world", x, y, 0)
			factory.create("#beetle_factory", data.tile2world(vmath.vector3(x, y, 0.2)))
		elseif t == 16 then
			tilemap.set_tile("#tilemap", "world", x, y, 0)
			factory.create("#rock_factory", data.tile2world(vmath.vector3(x, y, 0.2)))
		elseif t == 21 then
			tilemap.set_tile("#tilemap", "world", x, y, 0)
			factory.create("#door_factory", data.tile2world(vmath.vector3(x, y, 0)))
		end
	end
end
```

## Camera Positioning in the Game

The camera is centrally handled from `game/core/camera.script` which is embedded into the `common` collection's `gui` game object.

The camera checks to see the player's global position in the tilemap loaded and adjust's the camer's position in accordance to that.

```lua
local function newscreen(self)
	self.flip = not self.flip
	
	-- These 4 sprites are actually in charge of loading the game's background.
	-- The background is repositioned

	sprite.set_hflip("#sprite", self.flip)
	sprite.set_hflip("#sprite1", self.flip)
	sprite.set_hflip("#sprite2", self.flip)
	sprite.set_hflip("#sprite3", self.flip)
	sound.play("main:/sound#chirp")	
end

function update(self, dt)
	
	-- Get the player's global position according to the
	-- loaded data file that maintains game state. Make use of
	-- dimensions of the display screen to determine which position
	-- the player is at
	
	local x = math.floor(data.playerpos.x / data.CAN_W) * data.CAN_W
	local y = math.floor(data.playerpos.y / data.CAN_H) * data.CAN_H

	-- Adjust background position

	local pos = vmath.vector3(x, y, 0)
	if pos ~= self.pos then newscreen(self) end
	self.pos = pos

	-- Set the position of the camera and save the position in game state

	go.set_position(pos)
	data.screenpos = pos
end
```

## All other concepts

All other concepts can be understood when you understand the fundementals of defold. i.e. how Game Objects work, Collections & Collection Proxies / Factories, Game Object Factories, Sounds, GUI elements, Tilemaps, Scripts, Game Project Settings, Collisions, Particles, etc.