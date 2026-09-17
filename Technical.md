This file serves as a technical explanation of the more complex systems of Heavy Weather.

### The short explanation of key systems:
* **Room Generation:** After clicking 'start', a list of rooms is generated as structs alongside their connections and special rooms. Moving between rooms changes which struct is accessed.
* **Perks:** Perks are created as structs holding stat changes, requirements, and abilities. Updates on perk availability are checked constantly; when one is available, it's marked until it is unlocked.
* **Items:** Items are also structs including stat changes and abilities. Dropped items use a generic 'item' object that references the item's sprite and ID for retrieval.
* **Enemy Formations:** Enemy formations are picked from a pool of 2 difficulties depending on which stage of the floor you're on. Each enemy's location is stored in regards to the center of the room; when loaded, it simply calculates which way to orient the formation.

### A longer explanation if you're still reading:

#### Room Generation:
* Each room is defined as a struct containing identifying and gameplay information such as:
  * Room's ID
  * Width and height
  * Location in a grid
  * North, East, South, West connections, or if there is nothing there.
  * The enemy formation inside the room.
  * Background
  * Gameplay variables such as cleared, visited, or type of room.
* Using this struct, the generator places the starting room at `0,0` with an ID of `0` in the array and starts the creation loop.
  * This loop holds an algorithm which ends after creating a random amount of rooms, dependent on which part the player is on.
  * Using a random room from the array as a base, the algorithm picks a random direction and checks for population:
    * **If populated:** The algorithm continues without creating a new room, not incrementing the loop.
    * **Else:** The prospective room is created and fills out the connection with the base room and new room, incrementing the loop.
* After the array is populated, leftover room artifacts are cleared and certain rooms are picked for special properties:
  * **Room ID 0** (starting room, center of map) is set as the `START` room, removing any enemies in it.
  * The dead-end room physically closest, or one of the closest, is set as the `TREASURE` room, holding unique scenery and the Chest object.
  * The dead-end room physically furthest, or one of the furthest, is set as the `TRADE` room, holding a trader and the next part of the tower.
* Afterwhich, the player is brought into the `START` room.
  * **HOWEVER**, if the player has already passed the first 2 parts, room generation is overridden and the room list is manually populated with the `PASSAGE` and `BOSS` rooms.
* After creation, the generator persists and draws out the map on the HUD, defining room connections and coordinates.

![Part 1 map example](image.png)
