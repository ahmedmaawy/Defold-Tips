# About

These tips originate from the [Sub Strike](https://github.com/benjames-171/defold-games/tree/master/Sub%20Strike) Game by Ben James.

This game is simple: A ship at the top of the water basin tries to hurl explosive barrels at submarines at the bottom. The game dynamics is determined by how well you time your barrels by making best judgement on when and where you deploy these barrels. You also need to evade the mines that submarines deploy that try to float to the surface and explode when reaching the surface.

## Game Initialization

The `/main/main.collection` is where all the action is. This collection contains:

* All audio file references that are used throughout the game, even within nested collections that are spawned. They are all nested in the `sound` game object.
* The `handler` Game object, that loads the Game Collection or Menu Collection, co-ordinated by the `handler.script` file. Although it loads the `game/core/game.collection` collection. The collection contains various game objects that make the game tick.

## How clouds are generated

There are 2 large cloud and 3 small cloud objects that are deployed towards the top part of the game scene. The all derive the same script. Their speed of horizontal movement is set on initialization as so:

```lua
function init(self)
	self.speed = math.random(100, 200) / 100
	if self.large then self.speed = self.speed * 0.1
	else self.speed = self.speed * 0.05
	end
	sprite.set_constant("#sprite", "tint", vmath.vector4(1,1,1,0.75))
end
```

Their movement is dictaed by their `update` function:

```lua
function update(self, dt)
	local pos = go.get_position()
	-- Move the cloud with the specificed speed
	pos.x = pos.x - self.speed
	if pos.x < -MARGIN then
		-- Re-caliberate the position of the cloud once they move out of display margins.
		-- Bring them back to the right side.
		pos.x = data.CANV_W + MARGIN
		pos.y = math.random(250,280)
	end
	go.set_position(pos)
end
```

## Spawning Enemies

The `level/level.script` in the `game.collection` spawns enemies. The spawning happens when the `spawn` function called by the level `update` function. As so:

```lua
if self.time >= self.basetime then
	self.time = 0
	self.super = self.super + 1
	local super = false
	if self.super >= SUPER_FREQ then
		super = true
		self.super = 0
	end
	factory.create("#enemy_factory", nil, nil, {super = super})
end
```

## How the enemy operates

The enemy hovers along the screen from one direction to the other. During initialization, the position (left or right) is created at random (make reference to the `init` function for `game/enemy/enemy.script`):

```lua

...

-- Random determine the position as so:

pos.x = spawn[math.random(1,2)]

...

if pos.x > 0 then
	
	-- Move it to the right or left, and flip it based on the randomization result.
	
	sprite.set_hflip("#sprite", true)
	self.move.x = -self.move.x
	pos.y = pos.y + SPACE
	particlefx.play("#pfx-right")
else
	particlefx.play("#pfx-left")
end
```

### Spawning the mines

On each `update` call the mines are spawned by calling the `mine` function as so:

```lua
local function mine(self, dt)
	local pos = go.get_position()
	
	-- Move the submarine	
	
	if self.move.y == 0 and pos.x > LAND and pos.x < data.CANV_W - LAND then
		
		-- We make use of the timer to spawn a mine after specific time intervals

		self.time = self.time + dt
		if self.time > self.minetime and data.state == data.STATE_PLAYING then
			self.time = 0

			-- Spawn from the mine factory

			factory.create("#mine_factory")
		end
	end
end	
```

## Special Effects

The effects for a mine are mostly triggered when the mine reached the top of the water as so (triggered by the mine's `update`):

```lua
if pos.y > (data.SCR_H / data.PIXEL_SIZE) - 64 then
	msg.post("/effect1", "splash", {pos = go.get_position()})
	go.delete()
end
```

Or explode when in contact with the ship in the `on_message`:

```lua
if message_id == hash("contact_point_response") then
	msg.post("/effect", "explode", {pos = go.get_position()})
	go.delete()
end
```

Same applies to the `bomb` deployed by the ship, when in contact with the Submarine, in the bomb's `on_message`:

```lua
if message_id == hash("contact_point_response") then
	msg.post("/effect", "explode", {pos = go.get_position()})
	go.delete()
end
```

### Where the effects come from

The `/effect` being called above is actually in the `game` collection's `effect` game object. This is being addressed in the `msg.post` within the objects above