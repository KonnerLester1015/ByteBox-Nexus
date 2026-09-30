---
title: "R6 GeoGuessr"
sidebar:
  exclude: true
---

## Introduction

**R6 GeoGuessr** is a browser game where players identify *Rainbow Six Siege* maps from nothing but their floor-plan blueprints. It borrows the "where in the world is this?" loop of GeoGuessr and applies it to the game's 28 maps and map reworks (at the time of making the game).

I built it because *Tom Clancy's Rainbow Six Siege* is one of my all-time favorite games to play with friends, and I wanted a fun browser game we could come back to again and again.

The entire game is built with plain **HTML, CSS, and JavaScript**. There are no frameworks, libraries, or build tools, and the only external dependency is a Google Font. It is hosted as a static site on GitHub Pages.

{{< callout type="info" >}}
  **Play it live:** [konnerlester1015.github.io/R6GeoGuessr](https://konnerlester1015.github.io/R6GeoGuessr/)  
  **Source code:** [github.com/KonnerLester1015/R6GeoGuessr](https://github.com/KonnerLester1015/R6GeoGuessr) (GPL-3.0)
{{< /callout >}}

![R6 GeoGuessr gameplay](/R6GeoGuesserGame.png)

## Objectives

- Build a complete, replayable game loop (menu, gameplay, results).
- Keep the game **data-driven**, so adding a map means adding images and a JSON entry rather than changing code.
- Offer two difficulty levels from the same assets.
- Wrap the game in a cohesive tactical-themed UI that feels like part of the game it is about.

## How to Play

| Step | What happens |
| ---- | ------------ |
| **1. Choose an operation** | **Casual** shows blueprints as-is. **Realism** randomly rotates each blueprint by 0°, 90°, 180°, or 270°. |
| **2. Study the blueprint** | A random floor from a random map is shown. |
| **3. Enter your guess** | Type a map name. Autocomplete suggests matches, and the answer must match a map name exactly. |
| **4. Deploy the answer** | A correct guess moves to the next map. A wrong guess reveals a different floor of the same map. |
| **5. Request intel** | Reveals another floor of the current map without submitting a guess. |
| **6. Extract** | Skips the map and reveals the answer. |
| **7. Mission complete** | After all maps are played, the final score and total time are shown. |

A run covers every map once, in random order.

## Program Architecture

The project is deliberately small and flat:

| File | Purpose |
| ---- | ------- |
| `index.html` | Markup for all three screens (menu, game, game over) and the footer. |
| `scripts/game.js` | All game logic: state, rendering, input handling, timer, scoring (about 500 lines). |
| `styles/style.css` | Theming, animations, and responsive layout (about 1,000 lines). |
| `data/maps.json` | The map database: each map's name and its blueprint filenames. |
| `blueprints/bluewhite/` | 76 blueprint images across the 28 maps. |

### Screens as states

The page holds three `div` containers with a shared `screen` class (`menu-screen`, `game-screen`, `gameover-screen`). A small `showScreen()` function toggles an `active` class so only one is visible at a time. There is no router and no page reload, so moving between screens is instant.

### One state object

All mutable game data lives in a single `gameState` object: the current mode, the current map, which floors are still unseen, the score, the timer, and the current rotation. Every function reads from and writes to that object, and a single `updateScore()` function pushes it to the UI. For a project this size, it keeps the logic easy to follow.

### Data-driven maps

Each map is a small JSON entry. The game derives everything else from it: the list of playable maps, the autocomplete suggestions, the number of floors, and the image paths.

```json
{
  "name": "HEREFORD BASE",
  "bluewhite": ["Hereford_1.jpg", "Hereford_2.jpg", "Hereford_3.jpg", "Hereford_4.jpg"]
}
```

The data currently holds **28 maps**: 18 with three floors, 9 with two, and Hereford Base with four. The image set is keyed by name (`bluewhite`), so a second set (for example, colored blueprints) could be added as another key without restructuring the game.

## Game Flow

1. `init()` loads `maps.json` and wires up all event listeners.
2. The player picks a mode, which calls `startGame()`. It resets the score and timer and loads the first map.
3. `loadRandomMap()` picks a random map that has not been used yet and a random starting floor. If no unused maps remain, the game ends.
4. The player submits a guess:
   - **Correct:** the score increases and the next map loads after a short pause.
   - **Incorrect:** the image container shakes, and the next unseen floor of that map is revealed. If every floor has been shown, the map counts as failed and the answer is revealed.
5. `endGame()` stops the timer and shows the final score and mission duration.

## Key Implementation Details

### Random floors without repeats

Every map starts with a pool of floor indices. Each time a floor is shown, whether by a wrong guess or the **Request Intel** button, one index is drawn at random and removed from the pool. The same floor is never shown twice, and the pool running empty is what tells the game a map has been fully revealed.

```javascript
// Build a pool of floor indices, then draw from it without replacement
gameState.availableFloors = floors.map((_, index) => index);

const randomIndex = Math.floor(Math.random() * gameState.availableFloors.length);
gameState.currentFloorIndex = gameState.availableFloors.splice(randomIndex, 1)[0];
```

The same pattern drives three different features (first floor, wrong-guess reveal, and the intel button), which keeps the behavior consistent.

### Realism mode: rotated blueprints

Realism mode picks a random angle each time a blueprint loads and applies it with a CSS transform. A `transition` animates the turn.

A 90° or 270° rotation swaps an image's visual width and height, but its layout box stays the same, which can make it clip or overflow its container. To handle this, the code swaps the `max-width` and `max-height` constraints for those two angles.

```javascript
const rotations = [0, 90, 180, 270];
gameState.currentRotation = rotations[Math.floor(Math.random() * rotations.length)];
mapImage.style.transform = `rotate(${gameState.currentRotation}deg)`;

// Adjust image dimensions for 90/270 degree rotations
if (gameState.currentRotation === 90 || gameState.currentRotation === 270) {
    mapImage.style.maxWidth = '58vh';
    mapImage.style.maxHeight = '100%';
} else {
    mapImage.style.maxWidth = '100%';
    mapImage.style.maxHeight = '58vh';
}
```

Because the rotation is re-rolled on every image load, revealing another floor also changes the orientation, which makes Realism noticeably harder than Casual.

### Autocomplete

The suggestion list is built from `allMapNames` with a case-insensitive substring match, so typing `TOW` suggests `TOWER`. The full list also appears when the field is focused and empty, which helps players who do not remember exact names.

```javascript
const value = e.target.value.toUpperCase().trim();

const filtered = value.length > 0
    ? gameState.allMapNames.filter(name => name.includes(value))
    : gameState.allMapNames;
```

Guesses are compared as trimmed, uppercased strings, so the answer must match exactly. Autocomplete is the bridge that makes that practical, since players select a name rather than typing it perfectly. The suggestions list every map, not just the ones in the current run, so they never reveal which maps are still unused.

## Design and Styling

The UI is designed to feel like part of *Siege*'s tactical world:

- **Typography:** [Quantico](https://fonts.google.com/specimen/Quantico), a military-style typeface, with uppercase text and wide letter spacing throughout.
- **Palette:** dark grays with tactical gold accents, military green for success, red for failure, and blue for intel.
- **Themed copy:** the interface uses mission language throughout: *Operation: Casual*, *Request Intel*, *Extract*, *Mission Complete*. The code underneath still uses plain names like `easy-mode-btn`, so the theme is only a layer of copy.
- **Animated smoke background:** two oversized pseudo-elements (`::before` and `::after`), each filled with layered, very low-opacity radial gradients, drift slowly on 80-second and 100-second loops. One runs in reverse, and `mix-blend-mode: overlay` blends them into the dark background. Only `transform` is animated, which keeps the effect smooth.
- **Feedback animations:** a soft pulse and green glow for correct guesses, a shake and red glow for wrong ones.
- **Responsive layout:** media queries adjust the layout at several widths (1440, 1366, 768, and 480 px) and heights, and blueprint heights are sized in `vh` units so the image and the input controls fit on one screen.

## Challenges and Lessons Learned

**Keeping logic and content separate.** Moving all map information into `maps.json` meant that expanding the game, for instance when adding the map reworks, required no code changes. It is the decision that simplified the rest of the project.

**Image weight.** The blueprints are high-resolution (5000 × 3750 JPEGs at roughly 3 MB each, about 210 MB in total). Because only one image loads at a time, the game stays playable, but the first load of each blueprint depends on the player's connection. Resizing to a display-appropriate resolution, converting to WebP, and preloading the next blueprint would cut the load time.

## Running It Locally

Because the game loads `maps.json` with `fetch`, it is best served over HTTP rather than opened as a file.

1. Clone the repository:

   ```bash
   git clone https://github.com/KonnerLester1015/R6GeoGuessr.git
   cd R6GeoGuessr
   ```

2. Start any static file server, for example:

   ```bash
   python -m http.server 8000
   ```

3. Open `http://localhost:8000` in your browser.

### Adding a map

1. Add the blueprint images to `blueprints/bluewhite/`, named `MapName_#.jpg`. File names are case-sensitive.
2. Add an entry to `data/maps.json` with the map's name and image filenames.

That is all; the autocomplete, floor count, and map pool update automatically.

## Future Improvements

- **Optimize images** (resize, WebP, and preload the next blueprint).
- **Track best scores and times** in the browser so players can chase a personal record.
- **Account for intel in scoring.** Right now a correct guess scores the same whether it took one floor or all four.
- **Count extractions as failures.** Skipping a map currently reduces the remaining count without adding to "failed missions."

## Summary

R6 GeoGuessr is a small project with a deliberately simple foundation: one JSON file, one state object, and three screens. It covers a full game loop, two difficulty modes, resilient loading, and a themed UI, all without a dependency. It is a fun way to test your knowledge of *Rainbow Six Siege* maps, and was a great way to practice building a complete browser game from scratch and being able to make it public.