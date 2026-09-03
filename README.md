CSE423 – Computer Graphics Project Proposal
Lighthouse Keeper: Before the Storm
A 3D Exploration and Survival Game

1. Project Overview
Lighthouse Keeper: Before the Storm is a 3D exploration and survival game developed using Python and PyOpenGL. The player takes on the role of a lighthouse keeper stranded on a small island as a storm approaches from the sea.
The player's objective is to restore the island's lighthouse beacon before the rising storm makes the island impassable. To do this, the player must explore the island, collect fuel canisters, power up a generator, cross a previously inaccessible section of the island via a restored bridge, pass through a locked lighthouse gate, and finally activate the rotating beacon all while the surrounding water level continuously rises and floods low-lying paths.
The project is built entirely from primitives and functions available in the provided 3D OpenGL template: cubes, cylinders, spheres, quads, basic transformations, perspective projection, camera positioning, keyboard/mouse input, the animation/idle loop, and on-screen text rendering.
2. Project Aim
To develop an interactive 3D exploration and survival game that demonstrates practical use of 3D transformations, camera control, collision detection, timed animation, resource-dependent progression, and game-state management while remaining fully achievable using only the functions available in the provided OpenGL template.
3. Objectives
Construct a small, interconnected 3D island environment from basic primitives.
Implement a controllable player character with movement, rotation, and collision.
Implement a resource collection system (fuel) tied to game progression.
Implement a machine-interaction system (generator) gated by a resource requirement.
Implement animated progression barriers (gate and bridge) that unlock in sequence.
Implement a continuously rising water level that dynamically floods parts of the map.
Implement a countdown timer that creates time pressure.
Implement a lives system with respawn-on-hazard behavior.
Implement a full win condition culminating in an animated beacon.
Display all relevant game information through an on-screen HUD.
Implement a complete game-state system (Playing / Paused / Game Over / Victory / Restart).
Implement simple cheat/debug modes for testing and demonstration.
4. Game Story
The player is the keeper of a lighthouse on a small, isolated island. A severe storm is approaching, and the lighthouse's beacon is inactive because the emergency generator has run dry and part of the island's infrastructure, a bridge connecting the two halves of the island, has been damaged.
The player must explore the island, gather enough fuel to power the generator, wait for the bridge to restore itself once power is back, cross to the lighthouse, get past its locked gate, and activate the beacon — all before the rising tide floods the low paths and the storm's countdown reaches zero.
5. Main Game Objective and Progression Flow
Ultimate objective: Restore the lighthouse beacon before the storm timer reaches zero or the player runs out of lives.
START
  ↓
Explore Island
  ↓
Collect Fuel
  ↓
Activate Generator (requires enough fuel)
  ↓
Bridge Restores
  ↓
Cross to Lighthouse Area
  ↓
Gate Opens (requires generator ON)
  ↓
Reach Beacon
  ↓
Activate Beacon
  ↓
VICTORY
Failure occurs if the timer reaches zero, or if the player loses all lives to environmental hazards.
6. Main Features (12 Core Systems)
Feature 1: 3D Island Environment
A small, interconnected island built from cubes, cylinders, spheres, and flat quads: island terrain, surrounding water, the lighthouse, the dock (spawn point), scattered rocks, a generator area, and the two-part bridge crossing. The island is deliberately divided into sections so the player has to actually navigate rather than walk in a straight line to the objective.
Implementation: Static geometry built once at startup using translation and scaling on the available primitives.
Feature 2: Player Control (Movement, Rotation, Collision)
The player is represented by a small combination of primitives (sphere for the head, cylinder/cuboid for the body). The player can move forward/backward and rotate left/right, with position updated continuously in the idle loop.
Collision is handled with simple distance/coordinate checks: if the player's next position would be too close to a solid object (island boundary, rock, closed gate, generator housing), the movement is rejected instead of applied.
Key
Action
W
Move forward
S
Move backward
A
Rotate left
D
Rotate right

Feature 3: Camera System
A third-person camera follows the player at a fixed offset behind and above their current facing direction, updated every frame via gluLookAt, so the camera turns and repositions as the player moves and rotates. This gives the player a consistent view of both themselves and their surroundings, which matters for spotting flooded paths ahead of time.
Feature 4: Fuel Collection
Fuel canisters (cylinders) are scattered around the island, including some in less obvious spots to encourage exploration. When the player's position overlaps a canister:
the canister is removed from the scene,
the fuel counter increases,
the score increases.
FUEL: 2/5
SCORE: 200
Feature 5: Generator Interaction (Fuel-Gated)
The generator starts in an OFF state. Approaching it and pressing E attempts activation:
if fuel_collected >= REQUIRED_FUEL:
    generator_on = True
else:
    show "NOT ENOUGH FUEL, Required: 3, Collected: 2"
Once activated, the generator flips a single shared state flag that both the bridge and the gate check, this is what creates the exploration → collection → progression dependency, rather than letting the player walk straight to the end.
Feature 6: Barrier/Progression System (Gate + Bridge)
Both the bridge and the lighthouse gate are implemented as instances of the same underlying mechanic: a solid object that translates from a "blocked" position to an "open" position once its required condition becomes true.
Bridge: blocked (gap in the path) until generator_on == True, then animates into position over a few frames, connecting the two island sections.
Gate: closed until the player has crossed the bridge and reached the lighthouse base, then slides open, granting access to the beacon.
Reusing one animated-barrier function for both objects keeps the implementation simple while still giving two visually distinct progression moments.
Feature 7: Rising Tide & Flooding Hazard
A single water_level variable increases steadily over time (in the idle loop), independent of anything the player does — this is the core "environment changes even if you don't" mechanic that sets the game apart from a static level.
The water quad's height/scale is updated to visually reflect water_level.
Any low-elevation path where water_level > flood_threshold becomes flooded.
If the player is standing in a flooded zone, that counts as a hazard collision.
Beginning:  water_level = LOW      → all paths accessible
Later:      water_level > threshold → low path floods, player must reroute
Feature 8: Countdown Timer
A visible countdown (TIME: 120, decreasing every second via the idle loop's delta-time) represents time remaining before the storm becomes critical. Reaching zero triggers Game Over regardless of remaining lives, creating urgency independent of the tide.
Feature 9: Lives System
The player starts with a fixed number of lives (e.g. 3). Touching a flooded/hazard zone reduces lives by 1 and respawns the player at the dock (start position) rather than resetting the whole run. Reaching 0 lives triggers Game Over.
Hazard collision → Lives -= 1 → respawn at dock
Lives == 0        → GAME OVER
Feature 10: Victory Sequence
Reaching the beacon and pressing E is only accepted once the generator is on and the gate is open, this prevents a "walk straight past everything" exploit. On success:
the beacon (cylinder/cuboid/sphere assembly atop the lighthouse) begins continuously rotating via glRotatef in the idle loop,
the game state switches to VICTORY,
a final message and score are displayed.
LIGHTHOUSE RESTORED!
SCORE: 850
Feature 11: HUD (Heads-Up Display)
An on-screen 2D text overlay (via glutBitmapCharacter, as already demonstrated in the template's draw_text) shows all relevant live information in one place:
FUEL: 3/5     LIVES: 2     SCORE: 450
TIME: 82      GENERATOR: ON     GATE: OPEN
Feature 12: Game State Management (Playing / Paused / Game Over / Victory / Restart)
A single state variable controls overall flow:
PLAYING ⇄ PAUSED
   ↓
GAME OVER  or  VICTORY
   ↓
(press R) → PLAYING (all variables reset)
Pausing (P) freezes player input, the timer, the tide, and all animations simultaneously — since they're all just skipped when the state isn't PLAYING, rather than needing separate pause flags per system. Restarting (R) resets every game variable — player position, fuel, score, lives, timer, water level, generator/bridge/gate flags — back to their initial values.
7. Cheat / Debug Modes
Two simple, low-cost toggles for testing and demonstration:
Key
Cheat
Effect
F
Infinite Fuel
Generator activation ignores the fuel requirement
T
Tide Freeze
water_level stops increasing

Both are single boolean flags checked inside existing systems (fuel requirement, tide update), no new systems required to support them.
8. Full Controls Reference
Key
Action
W
Move forward
S
Move backward
A
Rotate left
D
Rotate right
E
Interact (generator / beacon)
P
Pause / Resume
R
Restart
F
Cheat: Infinite Fuel
T
Cheat: Freeze Tide

9. 3D Objects Used
Object
Primitives
Player
Sphere (head), cuboid/cylinder (body)
Lighthouse
Cylinders, cuboids, sphere (beacon housing)
Fuel canisters
Cylinders
Generator
Cuboids, cylinders
Bridge
Cuboids
Gate
Cuboid
Rocks
Scaled spheres/cuboids
Water
Large flat quad
Beacon
Cylinder + cuboid + sphere assembly, rotated continuously

10. Use of 3D Transformations
Translation — player movement, bridge/gate motion, fuel placement, respawn positioning.
Rotation — player facing direction, beacon spin, camera orientation via gluLookAt.
Scaling — rock size variation, water-level visual height, fuel canister sizing.
11. Technical Implementation & Template Compliance
Built in Python with PyOpenGL, using only functions demonstrated in the provided 3D template: glTranslatef, glRotatef, glScalef, glPushMatrix/glPopMatrix, gluPerspective, gluLookAt, gluOrtho2D, glutSolidCube, gluCylinder, gluSphere, gluNewQuadric, GL_QUADS, GL_POINTS, glRasterPos2f, glutBitmapCharacter, and the standard GLUT input/display/idle callback set. glutTimerFunc is not used; all timing (tide rise, countdown, respawn) is handled inside the idle callback using elapsed real time.
12. Collision Detection
All collision uses simple distance or coordinate-range checks — no physics library involved:
Player vs. solid objects: reject movement if the resulting position is within a minimum distance of a wall/rock/closed barrier.
Player vs. fuel: overlap check triggers collection.
Player vs. generator/gate/beacon: proximity check gates whether the E key does anything.
Player vs. flood zone: the player's position is checked against the flooded path's coordinate range, compared against current water_level.
13. Difficulty Progression
Early game: low water level, full timer, most paths open, low risk.
Mid game: water level climbing, some low paths starting to flood, player must plan routes around the rising danger.
Late game: water level high, fewer safe paths remain, timer running low — the player must reach the beacon quickly with whatever route is still open.
14. Uniqueness and Difference from Assignment 3
Assignment 3's core loop is combat-driven: move, aim, shoot enemies, track hits/misses. Lighthouse Keeper's core loop is exploration- and resource-driven:
Assignment 3:      Player → Shoot → Enemy defeated → Progression
Lighthouse Keeper:  Explore → Collect → Interact → Adapt to rising tide → Reach objective
The rising tide is the key differentiator: it's an environmental system that changes the map over time independent of player action, meaning a route that's safe early on can become unusable later — a form of dynamic-environment gameplay that doesn't exist in a straightforward shooting game, and doesn't require any physics engine or advanced rendering to implement.
15. Development Strategy (Phased)
Phase 1 — Environment: island, water, lighthouse, dock, rocks, generator area.
 Phase 2 — Player: model, movement, rotation, camera, collision.
 Phase 3 — Collection: fuel objects, collection detection, score, HUD.
 Phase 4 — Progression: generator, gate/bridge barrier system.
 Phase 5 — Environmental Systems: rising tide, flooding, hazard collision.
 Phase 6 — Game Management: timer, lives, game states, restart.
 Phase 7 — Polish: beacon animation, HUD refinement, cheat modes, testing.
Phases 1–4 alone already form a complete, demoable core loop (explore → collect → interact), so remaining time can be safely allocated toward Phases 5–7 without risking an incomplete submission.
16. Expected Outcome
A fully functional 3D exploration and survival game in which the player navigates an island, collects resources, restores infrastructure, adapts to a rising water level, manages a countdown timer, avoids environmental hazards, and completes the game by reaching and activating the lighthouse beacon before the storm arrives.
17. Summary of Features
#
Feature
Purpose
1
3D Island Environment
Creates the game world
2
Player Control
Movement, rotation, collision
3
Camera System
Third-person following view
4
Fuel Collection
Resource gathering, scoring
5
Generator Interaction
Resource-gated progression
6
Barrier/Progression System
Controls access via gate + bridge
7
Rising Tide & Flooding
Dynamic environmental threat
8
Countdown Timer
Creates urgency
9
Lives System
Limited margin for error
10
Victory Sequence
Final objective and win state
11
HUD
Displays all live game information
12
Game State Management
Controls Playing/Paused/Over/Victory/Restart
—
Cheat Modes (Infinite Fuel, Tide Freeze)
Testing/demonstration support

18. Conclusion
Lighthouse Keeper: Before the Storm is proposed as a feature-rich but manageable 3D Computer Graphics project. By combining resource collection, machine interaction, animated progression barriers, a continuously changing environment, and full game-state management, the project comfortably exceeds the required feature count relative to Lab Assignment 3 while remaining conceptually and mechanically distinct from it — all achievable using only the primitives, transformations, camera functions, and input/animation mechanisms provided in the 3D OpenGL template.


