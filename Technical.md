# Technical List

For now this is a list of tech tasks that are next:

- Get ship scanning to go properly
- Build up client side game world properly, and update it from event stream.
- UI to set navigation vector
- Draw ships we get from sensors on UI

Start with a universe teeming with ships first? Driven by AI?
How to limit ship speed? Do they have a ship class, a template that
multiple ships can use? No, every ship is very unique.

Users Point of view:

Ships show up on scanner, based on their emissions, regardless of how far they are.
Ping sends out an emission, and reveals other ships in a small range. There could
1000s of ships around in the same area.
Event processing must be somehow event driven, which means storing ship
table in memory.

Open questions:
- Ship Emissivity field? Where to put it? Into the world ship table? update it after every frame?
  Perhaps better to put it in a separate ship sim table, but if that changes, it impacts every frame too,
  shouldn't that be synced?
- How to handle ad-hoc emissions (weapons fire, explosion) to ensure that the sensor reading doesn't
  continue onto the next frame.

Idea:
- Add `sensor sensitivity`, and `emmission` values to world object table. Each object
  has a sensor range, and how much it emits. Must be careful not to do this out of order,
  move increases emission, but frame scans ships in order. Process all the ship's sim actions
  first, then query for sensors. Two passes :(
- Treat ad-hoc emissions completely separately, bypassing world list table, just query using
  sensor sensitivity, and send out the messages immediately. Breaks rules of the frame.
- Treat move list as a generic input order list, which would include any ad-hoc emissions,
  just flag them like so.

UI State Machine:
- Event stream + Queries. Starts with Undefined. UI Subscribes, gets game time beginning,
starts buffering a list of received events. Issues queries required to pre-fill the client
side game world. If results game time is earlier than subscription guarantee, discard and
re-query. Probable hell with react state here.

Queries and event stream populate and update the world. The world itself is injected into
various components, and doesn't get modified, so react will not re-render when the world updates.
Canvas uses animation frames to query things from the world, and draw them at 60fps
World will contain all the information required to draw everything smoothly, including predicted
movement. Eventually it will also smooth out the predictions, etc.
UI can mark bits of itself if the world gets stale, where ship movement goes into predictions.
Not sure how to handle your own ship though.

# Movement Quanta

- UI Requests movement to target coords
- Request added to next frame table, respond with frame size.
- Frame process begins
- API asks ship sim to subtract fuel for quanta
- World update starts to run, processing all signals, using target coords
- All results are sent out to clients, with size of quanta, and time when it finishes
- UI waits for end of quanta window to issue new command
- (API may buffer the command if UI sends it too early)
- Quanta size dynamically adjusted based on how long that frame took.

UI has some client side simulation to prevent people asking for things that are
impossible, and UI showing them.

- Add to quanta table until next frame is ready to run
- When time reaches, a global timer starts running frame, selects all rows from frame table.
- Processes all requests/signals, with aid of ship sim
- Sends out all signals to all clients, as it processes, in parallel if possible
- Records all new ship positions
- Ends, records size of frame
- Wait for next frame to fill


What does the UI see?

- GraphQL mutation with "Go to target coords (x, y, speed)"
- Reply is:
  - Your movement is scheduled, if sim allows it
  - Proceed at locally simulated speed at the end of current frame
  - during the frame you'll recieve an update with exact movements
  - if not recieved by the end of frame, abort / requery / backtrack
- Client side waits for start of frame.
- Client side begins move.
 

Remember to document this shit in tests / source code better

