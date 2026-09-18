This file serves as a technical explanation of the more complex systems of Heavy Weather.

### The short explanation of key systems:
* **Room Generation:** After clicking 'start', a list of rooms is generated as structs alongside their connections and special rooms. Moving between rooms changes which struct is accessed.
* **Perks:** Perks are created as structs holding stat changes, requirements, and abilities. Updates on perk availability are checked constantly; when one is available, it's marked until it is unlocked.
* **Items:** Items are also structs including stat changes and abilities. Dropped items use a generic 'item' object that references the item's sprite and ID for retrieval.
* **Enemy Formations:** Enemy formations are picked from a pool of 2 difficulties depending on which stage of the floor you're on. Each enemy's location is stored in regards to the center of the room; when loaded, it simply calculates which way to orient the formation.

### A longer explanation if you're still reading:

#### Room Generation:
* Each room is defined as a struct containing identifying and gameplay information such as:
  * Rooms ID
  * Width and height
  * Location in a grid
  * North, East, South, West connections, or if there is nothing there.
  * The enemy formation inside the room.
  * Background
  * Gameplay variables such as cleared, visited, or type of room.
* Using this struct, the generator places the starting room at `0,0` with an ID of `0` in the array and starts the creation loop.
  * This loop holds an algorithm which ends after creating a random amount of rooms, dependent on which part the player is on.
  * Using a random room from the array as a base, the algorithm picks a random direction and checks for occupation:
    * **If occupied:** The algorithm continues without creating a new room, not incrementing the loop.
    * **Else:** The prospective room is created and records the connection with the base room and new room, incrementing the loop.
* After the array is populated, leftover room artifacts are cleared and certain rooms are picked for special properties:
  * **Room ID 0** (starting room, center of map) is set as the `START` room, removing any enemies in it.
  * The dead-end room physically closest, or one of the closest, is set as the `TREASURE` room, holding unique scenery and the Chest object.
  * The dead-end room physically farthest, or one of the farthest, is set as the `TRADE` room, holding a trader and the next part of the tower.
* Afterward, the player is brought into the `START` room.
  * **HOWEVER**, if the player has already passed the first 2 parts, room generation is overridden and the room list is manually populated with the `PASSAGE` and `BOSS` rooms.
* After creation, the generator persists and draws out the map on the HUD, defining room connections and coordinates.

![Part 1 map example](image.png)

Fun fact: the map display was initially used for playtesting and debugging, but playtesters found it to be too helpful to remove. Thus, it was given its own place in your arsenal.

#### Room movement:
* Starting at Room 0, movements are initiated by Door objects.
  * Doors are locked if enemy count is above 0.
* After contact with an open door, the floor controller's fade effect is set to `FADING OUT` and player and enemies are frozen:
  * When the fade effect reaches 1 (max opaque), the floor generator checks what object called it.
    * `DOOR:` the next room is accessed
    * `LADDER:` the room list is cleared, part is increased, and a new map is generated
    * `BOSS DOOR:` the room list is cleared, and the boss segment loads in place of a map
* During the `DOOR` fade, the current room is unloaded and
  * the background is set to the new room's background (important for special rooms)
  * The correct wall-barriers and doors are loaded
  * Camera, player positioning, and special objects are loaded.
  * Using the room's 'contents' variable, certain actions are taken
    * `COMBAT:` enemies are loaded (only on specifically combat rooms)
    * `TREASURE` & `TRADER` both heal the player
    * `BOSS:` loads the boss
    * After which, housekeeping such as refilling items, setting `visited` to true, and setting fade to `FADING IN`.
* After the fade in is completed, player and enemies are unfrozen, completing the movement.

#### Perks and Items:
* Both items and perks are fairly similar, being stored as structs containing stat changes, abilities, names, and respective IDs for both systems.
* Items:
  * Item IDs refer to the items enumeration, ensuring the system can be easily expanded upon.
  * Items must be found in `CHEST` or `TRADER` objects, and don't have requirements to use.
  * When an item is picked up, it calls a function to adjust the player's stats and activate its ability.
  * Items have 2 abilities. if the player presses space while its animation is still active, the secondary ability is used instead.
  * If an item is dropped, its stat changes are removed.
  * Items on the ground use a generic `item` object, holding which item it refers to and the sprite of that item.
  * Item trading, buying, or selling can be done at `TRADER` NPCs.
* Perks:
 * Perk IDs act similar to item IDs, referring to the perks enumeration.
 * Perks can be unlocked in the perk menu after its specific requirements have been met, signified by the 'paper' sound after availability opens.
 * Hovering over the perks icon in the perk menu displays its name, a short description, its requirements, and its availability `LOCKED`, `AVAILABLE`, `UNLOCKED`.
 * Perk availability is checked constantly by a function, assessing if both a perk's requirements are met and if it's still `LOCKED`.

![Screenshot of the Perk Menu](PerkDisplay.png)

#### Enemy Formations:
* Each enemy is placed in a 'pool', defining what characteristic it has, such as `light`, `ranged`, `heavy`, or `special`.
* The formation is recorded as an array of structs, each containing an enemy role and its position relative to the center.
 * When enemy role is called, it refers to its characteristic pool, pulling an enemy from it. This maintains variety while still keeping defined challenges for each room.
 * Enemy position is dependent on which direction the player enters from, transforming its coordinates accordingly.
   * `NORTH:` x = -relativeX, y = -relativeY
   * `EAST:` x = relativeY, y = -relativeX
   * `SOUTH:` x = relativeX, y = relativeY
   * `WEST:` x = -relativeY, y = relativeX

#### Boss Design:
* The demon Baal serves as the final roadblock for the demo, utilizing storm and lightning attacks like the player.
* Baal is given 3 different attacks to represent the floor, lightning 'jabs'
