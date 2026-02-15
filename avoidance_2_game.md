# Avoidance 2.0

## Code

The source code for this game can be found here on GitHub: <https://github.com/SabaSabaXYZ/avoidance-2.0>.

This game is also playable in your browser through itch.io: <https://s2saba.itch.io/avoidance-20>.

## Overview

After years of focus on higher level languages like Haskell, Common Lisp, TypeScript, and Java, I found that my low-level skills had started to atrophy.
To remedy this, I decided to revisit C for my next project.
Since I already had an original game concept developed for the terminal (see [Avoidance](https://sabadev.xyz/avoidance_game)), I figured it would be good to recreate it with a proper GUI to make it more palatable to casual players.
I also wanted to make the game easier to distribute than asking prospective players to download GHC and build it from source.

## Revisiting C

I chose to build the game using Raylib.
Raylib took care of a lot of low-level details like window creation.
It also provided me with a bunch of utilities for drawing text, collision checking, and so on.
Nevertheless, I still maintained full control over memory management and the game architecture.

I placed the entire game state into a single struct that I would pass around to different methods.
To maintain performance, I opted for a design similar to the entity component system.
Namely, I defined identifiers for the box, the player, and the first box thief.
I then defined an array of positions and an array of movement directions, each laid out such that the character identifier serves as the index to fetch its own position and direction.
The rationale behind this approach is that the game would typically be updating movement directions en-masse, then positions en-masse.
By grouping these together, the CPU can load a set of directions or positions in batches via a cache line.

To improve performance even further, I initially allocated the positions and directions on program startup via the heap.
This memory would then be released when the program is terminated.
Tracking the number of characters would then fall to a single counter called `character_count` that would be increased or reset as needed.
Hence, there are no slowdowns caused by garbage collection.

## Targeting other platforms

The only major headache I ran into was setting up a build system and integrating third-party libraries.
Raylib was easy enough to install via CMake for Linux, but setting it up for cross-compilation to Windows and Web Assembly required a bunch of steps that boiled down to the following:

- Clone the Raylib repository into a separate directory (e.g. `~/Documents/raylib`).
- Build `~/Documents/raylib/src` using CMake to produce a static library, making sure to pass in the appropriate flags for either Windows or WASM.
- Construct a shell script that runs a GCC command to compile and link the project, making sure to reference the newly-compiled Raylib static library.

Although my initial Linux implementation allocated the character data on the heap during startup, this inexplicably triggered a crash on the Mingw-compiled Windows binary.
To fix this, I switched to using stack allocation for everything.
When I then started targeting Web Assembly, I had to override the stack allocation size for the compiled WASM binary during compilation.

## Releasing on itch.io

Once I had the web assembly files generated and tested locally, releasing to itch.io turned out to be trivial.
First I had to ensure that I had an `index.html` file containing the game.
Next, I just had to zip all the game files together and upload them to itch.io.
After entering in the game details and description and uploading some screenshots, the game was ready to publish.
You can play the game here: <https://s2saba.itch.io/avoidance-20>.
