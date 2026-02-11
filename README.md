# Svelte 3D User Globe Visualization

Just a small side project I threw together to mess around with 3D globe rendering and some geo data.

Basically I wanted to show company users on a spinning globe with points popping up at different lat/long locations. I pulled the country data from a TopoJSON file, turned it into usable features with topojson-client, and used globe.gl + Three.js to draw everything. The points come from a simple countries.js file I made with capital coordinates (could easily swap in real user data later).

It's not super fancy — just something I prototyped in the Svelte REPL to learn how these libraries play together. I also containerized it with Docker so I could run it consistently on localhost when showing it off.

## Features I ended up with
- Interactive 3D globe using Globe.gl / Three.js
- Points for user locations pulled from countries.js
- Loads real country polygons from World Atlas (50m resolution)
- Simple isOnLand check with d3-geo to see if a point lands on actual land
- Basic pulse animation on the points

## Tech I used
- Svelte (for the frontend/component stuff)
- Globe.gl + Three.js (the 3D magic)
- d3-geo + topojson-client (handling the map data)
- Docker (just to make running it easy without messing up my machine)

## How to run it locally (Docker way)
This was started in the Svelte REPL, so it's not a full repo with build scripts. But if you want to see it running:

1. Create a quick Svelte project:
   ```bash
   npm create vite@latest test-globe -- --template svelte
   cd test-globe
   npm install
