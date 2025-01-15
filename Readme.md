# Prototype Story Board

You are piloting a ship through dense atmosphere of an unknown planet
The emissions on your ship can be hidden, to slip past other ships unnoticed.
You drop off the navnet claiming an equipment malfunction, this is a low security area anyway.

Powering down various equipment to lower your own emissions.

You passively scan for other vessels using EM and particle detectors of various sensitivities, looking for a target.

# Individual Mechanics

One of the ways I'm hoping this ends up being cool to play is we try to make certain
interactions happen off the cuff. You can do more planning for higher efficiency, but we
strive to make decisions at the time to have most impact. We trade planning ahead for
choosing a play style, but never penalise you for searching for one. Any advantage you
gain by better planning, the game should help you with, but it should still retain
survival elements where you are searching for things you need, more or less endlessly.

**Beginning:**
No loose ends on starting, everything is taken from the world, and your death puts it
back into the world. You fly basically a piece of junk thats almost worthless,
salvaged by you from the most basic materials found lying around. Anything expensive
that you got is loaned to you on condition of completing various salvage missions,
by world's inhabitants. There are plenty of static wrecks everywhere

Practical: Anyone can loan hardware for a salvage mission, in fact, thats one way
to get cheap labor from the newcomers. The better the hardware, the more likely they
are to succeed, etc.

World itself has a few industries that make the required hardware occasionally, but
in truth the good stuff is so rare that salvage is really the only way to get your
hands on the stuff. Basic required equipment should be easier to find (with basic
tools, more advanced equipment doesn't help locating basic ones)

Piracy is another way to get hardware.

You spawn on a station

**Death and respawn:**
On death you respawn again with nothing, but the knowledge of your past life. The
lore is that you've found the log (or it was passed down to you from friends/family)
of your previous life, so you know where your wreck is, and all the stuff you've
encountered. you can pick up your predecessors contracts, because you have the
contacts (or possibly decryption keys) even if they are late.

**Stealth and Emissions:**

Each component on the ship emits energy based on its constituent components, every build is a little random,
with component age taken into account. This creates a unique signature that your ship can be recognised by.

Engines and weapons (as well as damaged ship sections) emit particles, which
can be captured by a particle sensor, reconstructing a "trail".

These signals get attenuated by distance, and any environmental area effects.

EM signals travel as a sphere from the origin, produced as quanta of energy gets used up in a ship systems.
Particles expand at a slow rate until their concentration is below background levels (or sensitivity of instruments)

**Ship components:**

Each ship is a simulation of its components, stepped by variable sized chunks called "quanta". Variable to
control server performance. On a most basic level a ship is a fuel tank + engine (a power plant is a type of engine).
When you want the ship to move, engine calculates how much it can move in a given quanta, requests some fuel from the
fuel tank, and steps through the sequence. Ship is modelled with buffers in between each component, so every step
is delayed by that buffer. Navigation requests the ship to move to x, y, z coords, distance is worked out and split
into first quanta + rest of distance. Engine says I can move N distance within that quanta, for K fuel. If buffer
between engine and fuel tank doesn't have K fuel, the ship won't move.

**AI Lives:**

Each AI Ship is simulated by like a person that accepts jobs off the board, pilots the ship, has a certain level
of skill, but obviously is still quite dumb. Using automated systems, and safety of the navnet. This is to create
a feeling of a living universe, where individual ships have personalities, and places to be. A bit like X universe
but without AI being cheaty. This might be difficult to implement, or balance.

**BSG style computer hacking**

Ability to override automated pieces of equipment on any ship, preventing people from building huge single
person ships, because it will be too difficult to micro, and overridden immediately. Automating /
Programming is done by some sort of non turing complete language, which can be executed in predictable
time. Hacker can connect and change any variable in the execution table, but not rewrite the code.
Think of it this way: You can write an automated gun battery reloader, but someone can hack it
and fire the shell out of order, causing an explosion. More primitive, the better in this case. But it
also means fewer, but bigger and simpler machines.

**Player Spawn**

Players spawn out of existing AI ships/users. 

# Interface

Purely keyboard driven, broken up into sections that can be quickly switched between, that are used for different
purposes. Engineering, Navigation, Communication. Each mode has their own keybindings that are optimised for it.

Engineering for example has keys assigned for each of the ship functions, and provide a way to drill down
to components, and toggle individual operation.

Navigation has a list of contacts, each would have its own key (aaa, aab, etc).

Helm commands are issued by entering course and speed, but to make some maneuvering fluid, should be able to take
control of a vessel manually, and nudge it.


# Absolute minimum viable product (from players perspective)

- Empty space, but with detection range of inverse cube of distance
- No particle sensors, just hitscan EM emissions
- Bare bones ship simulation, just fuel
- No combat
- ~~World is a sphere, lat, long, altitude/depth~~
- World is a flat 2d plane
- Some sort of navigation interface
- Some sort of barebones sensor interface

**Navigation Interface**

Hotkey ctrl-1. Displays local position with sensor feed overlayed. keyboard commands lay in a course and speed,
set a target location, manual mode of controlling the thrusters with keystrokes. translation, rotation, vector?
What sort of 3D display? Top down + depth

**Sensor Interface**

Every other contact is given a unique name, based on last seen list. At first this will be server side,
pretending that the navnet gives out that name. Later on this will correspond to the ship signature.
with client computer matching signatures it seen last. Contact history is stored within the UI, for now.


# Technical details

**Navigation**

All actions from ships are done via graphql, and are written into a journal log, then acked to the
client. The server can work out who then needs to be notified after that, and send out those
messages, modify any storage, etc then the log entry is deleted. For now client should have
the memory of what happened to it.

Everything is designed in a way that buffers all messages. Sensors pick up a trace of emissions
which get recorded as a list of contact events. Later on we can worry about cleaning them up.

Need to think if this is a log of events that happened, or list of work items to do.

Except how to deal with constant movement?

Database design: Stores time, position, target vector. The server engine interpolates
current position, and periodically sends it out to all the intersecting clients.

When helm requests to change a position, it does a qraphql mutation, server takes the current
time, and uses that to store new position and vector.

**Sensors**

EM emmissions are hitscan, tested based on attenuation over distance. Particle emissions work
on a delay, the server must keep a bunch of zones that have particles, and their decay rates,
then collission check each ship incase they enter any of them

Everytime there is an emission, server checks all ships within a certain distance, and sends
them messages on their channel (in the future possibly buffering seen contacts).

But how do emissions occur? Do they have to come from the ship sim?

Do all ships constantly emit ? That would be very costly server side

# Even more tech notes

Doobie provides a way to interface with PG Listen and Notify: https://tpolecat.github.io/doobie/docs/15-Extensions-PostgreSQL.html
