# About

These tips originate from the [GB Rausers](https://github.com/britzl/gbrausers) Game by Björn Ritzl.

## Game Initialization

The `/game/game.collection` is where all the action is. This collection contains:

* The `game` Game Object, which contains the game initialization script under `game.script` and `enemyplane_factory` factory to spawn enemy planes.
* `hud` collection to hold the GUI.
* References to other collections and game objects that include:
	* Collection to hold the `player` object
	* Game Objects for the Buildings.
	* Collection containing logic to Spawn Clouds dynamically.
	* Water surface, which is setup as a collision object.

## Spawning Enemy Planes

Enemy planes are spawned in 2 scenarios:

* When the game initializes (under the `game.script`).
* When an enemy plane is destroyed (it sends a broadcast message via `ludobits` library, which is captured by the `game.script`).

## The Player and the Global Camera

The main camera for the player is attached to the `player` collection. The `player` collection also has the `camera` game object, which has its position updated on each `update` function call of the `player` game object as so:

```lua
-- adjust camera
local target_offset = vmath.rotate(go.get_rotation(), UNITVECTOR_UP) * distance * 15
self.camera_offset = self.camera_offset + (target_offset - self.camera_offset) * dt
go.set_position(pos + self.camera_offset + CAMERA_OFFSET, "camera")
```

## About the clouds

The clouds are spawned procedurally from the cloud factory. The `clouds.script` does this in its initialization script as so:

```lua
self.clouds = {}

for i=1,350 do
	local position = vmath.vector3(math.random(-4000, 4000), math.random(20, 800), -1)
	local id = factory.create("#cloudfactory", position, nil, {}, math.random(4, 10) / 5)
	table.insert(self.clouds, id)
end
```

## Generating Debris

You will find `debrisfactory` factory being referenced in a couple places. The debris are generated when the following actions take place:

* When the `building` collision objects are hit.
* When an `enemy` collision object is hit with `GROUP_PLAYERBULLET`.

Make reference to the `on_message` calls for both `building` and `bullets` to see how these debris are generated.

## The Water Splash Effect

This is one of the most fascinating bits in the game. When a plan is close to water, a watersplash effect takes place. This is how it works:

* The `water` collection has a collision object.
* It also has a `splash` factory which is linked to the `splash` game object.
* The `player` object has 2 collision objects, one is more outside then the other. The outer collision object is called the `splashtrigger`
* The `on_message` function on the `water.script` script file is structed to do the rest:

```lua
if message.group == GROUP_PLAYER or message.group == GROUP_ENEMY then
	self.splash_timeouts[message.other_id] = self.splash_timeouts[message.other_id] or socket.gettime()
	
	if socket.gettime() >= self.splash_timeouts[message.other_id] then
		local pos = message.other_position
		pos.y = 0
		factory.create("#splashfactory", pos)
		self.splash_timeouts[message.other_id] = socket.gettime() + 0.05
	end
end
```
* The `init` function of each `splash` object is made to animate the object to play a splash effect as so:
```lua
local height = math.random(10,15)
go.animate(".", "position.y", go.PLAYBACK_ONCE_PINGPONG, go.get_position().y + height, go.EASING_OUTQUAD, 0.5, 0, function()
	go.delete()
end)
```

## Differentiating the enemy bullet and the player bullet

The enemy bullet and player bullet are both different game objects that work on the same script file `bullet.script`. The only difference is in the collision group and mask settings of their colliders.