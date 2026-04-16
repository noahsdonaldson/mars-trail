# Mars Trail

A static browser survival game inspired by The Oregon Trail, rebuilt as a Mars crossing with LLM-driven crew and merchant dialogue.

Lead a rover expedition 3,200 km across the Martian surface. Pick a crew, load the rover, manage oxygen, food, water, repairs, and morale, then survive hazards, breakdowns, and bad calls long enough to reach New Gale.

## How to Play

1. Open `index.html` in any modern browser.
2. No installation, build step, or local server is required.
3. If you want live AI dialogue, open the browser console and run `window.MarsTrail.llmClient.setApiKey('sk-...')`.

### Gameplay

- **Name your commander** and **pick 3 crew members** from a roster of specialists
- **Buy supplies** at the starting outpost with your 4,400-credit budget
- **Travel across Mars** day by day, choosing your pace:
  - Cautious — slow but safe
  - Steady — balanced
  - Fast — risky, burns more O2
  - Reckless — high speed, high danger
- **Handle random events**: dust storms, rover breakdowns, crew illness, mysterious signals
- **Stop at 5 outposts** to resupply, repair, and talk to AI merchant NPCs
- **Reach New Gale Crater** with at least one crew member alive to win

### Controls

All input is keyboard-driven.

- Number keys choose actions.
- `Enter` confirms or advances.
- `Escape` backs out of detail screens and overlays.
- `P` cycles travel pace.
- `Tab` reveals any currently typing text.
- `-` / `+` adjust text speed.

## Features

- **Retro terminal UI** — CRT scanlines, green phosphor text, ASCII art landscapes
- **AI Agent NPCs** — Crew and merchants powered by OpenAI (GPT-4o-mini), with personalities, memory, and contextual dialogue
- **Resource management** — Oxygen, food, water, rover parts, med kits, batteries
- **25+ random events** — Environmental hazards, mechanical failures, crew drama, discoveries
- **5 unique outposts** — Each with its own merchant personality and supply economy
- **Scoring & ranks** — From "Dust in the Wind" to "Mars Legend"

## Tech Stack

- Single HTML file — vanilla HTML, CSS, and JavaScript
- OpenAI API (GPT-4o-mini) — powers all agent dialogue
- Zero other dependencies
- Works offline for mechanics, requires internet for AI dialogue
- ~3,000 lines of code

## Requirements

- A modern browser
- An OpenAI API key if you want live dialogue

## GitHub Pages

This project is a good fit for GitHub Pages because it is a static site with no build pipeline.

- Push the repository to GitHub.
- In the repository settings, enable Pages from the `main` branch root.
- Your game will publish directly from `index.html`.

Important:
This project does not include a backend. GitHub Pages cannot safely store a private OpenAI API key, so players must provide their own key in the browser if they want live LLM dialogue.

## Project Structure

```
mars-trail/
├── index.html    # The complete game
├── .gitignore    # Git / macOS / editor ignores
├── .nojekyll     # Serve as a plain static site on GitHub Pages
├── SPEC.md       # Full game design specification
├── TASKS.md      # Implementation task list
└── README.md     # This file
```

## Development

See [SPEC.md](SPEC.md) for the full game design document including:
- Game mechanics and balance numbers
- AI agent architecture
- UI wireframes
- Scoring system

See [TASKS.md](TASKS.md) for the implementation task breakdown with acceptance criteria.

## Credits

Inspired by MECC's *The Oregon Trail* (1990). Set on Mars because wagons are boring.
