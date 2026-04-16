# Mars Trail — Game Specification

## Overview

**Mars Trail** is a single-player, browser-based survival/strategy game inspired by *The Oregon Trail* (1990s edition). Instead of traveling the American frontier by wagon, the player leads a crew of colonists across the surface of Mars in a pressurized rover, navigating from a landing zone to a newly established colony outpost.

The game features a retro pixel/terminal aesthetic, turn-based decision-making, resource management, random events, and AI agent-driven NPCs that have personalities, memory, and autonomous decision-making.

---

## Setting & Story

**Year:** 2187  
**Location:** Mars — Valles Marineris region  
**Premise:** Earth has established a permanent colony on Mars called **New Gale Crater**. You are the commander of a 4-person crew that has just landed at **Ares Station** (an older base). Your rover must travel 3,200 km across the Martian surface to reach New Gale Crater before your supplies run out.

---

## Core Gameplay Loop

```
Start → Choose Crew → Buy Supplies → Travel →
  ↳ Random Event / Decision Point →
    ↳ Manage Resources → Rest / Repair / Trade →
      ↳ Reach Outpost (shop, heal, interact with AI NPCs) →
        ↳ Continue Travel → ... → Arrive at Colony (Win) or Die (Lose)
```

Each "day" of travel is one turn. The player chooses a pace and actions each turn.

---

## Game Phases

### Phase 1: Crew Selection
- Player names their commander (themselves)
- Choose 3 crew members from a roster of 6 candidates
- Each crew member has: **name**, **role**, **skill bonuses**, **personality trait**
- Roles: Engineer, Medic, Geologist, Botanist, Pilot, Security
- Traits affect AI agent dialogue and decision suggestions

### Phase 2: Supply Purchase
- Starting budget: 2,000 credits
- Items available:
  | Item | Unit | Cost | Notes |
  |------|------|------|-------|
  | Oxygen Tanks | 1 tank | 50 cr | Each tank = 5 crew-days |
  | Food Rations | 1 crate | 40 cr | Each crate = 10 crew-days |
  | Water Packs | 1 pack | 30 cr | Each pack = 8 crew-days |
  | Rover Parts | 1 kit | 100 cr | Repairs rover damage |
  | Med Kits | 1 kit | 80 cr | Heals injuries/illness |
  | Batteries | 1 cell | 60 cr | Powers equipment, +travel range |

### Phase 3: Travel
- Total distance: 3,200 km
- Daily travel depends on pace:
  | Pace | km/day | O2 Usage | Risk Level |
  |------|--------|----------|------------|
  | Cautious | 20 | Normal | Low |
  | Steady | 40 | Normal | Medium |
  | Fast | 60 | +50% | High |
  | Reckless | 80 | +100% | Very High |

### Phase 4: Outposts
- 5 outposts along the route (roughly every 640 km)
- At each outpost the player can:
  - Buy/trade supplies (prices vary, some items scarce)
  - Repair the rover
  - Heal crew at medical bay
  - Talk to AI agent NPCs for advice, lore, trade deals

### Phase 5: Ending
- **Win:** Reach New Gale Crater with at least 1 surviving crew member
- **Lose:** All crew die, or rover is destroyed beyond repair, or critical resources hit zero

---

## Resource System

| Resource | Depletes Per Day (4 crew) | Effect at Zero |
|----------|---------------------------|----------------|
| Oxygen | 0.8 tanks | Crew takes damage each turn |
| Food | 0.4 crates | Crew health degrades |
| Water | 0.5 packs | Crew health degrades, morale drops |
| Rover HP | Event-driven | Rover disabled, game over if no parts |
| Batteries | 0.1 cells | No comms, no sensors, more random events |

- Resources scale with living crew (fewer crew = less consumption)
- Morale is a hidden stat affected by events, deaths, and decisions

---

## Random Events

Events trigger based on pace, distance, and probability. Categories:

### Environmental
- Dust storm (lose time, possible damage)
- Meteor shower (rover damage possible)
- Solar flare (battery drain, possible crew injury)
- Temperature drop (extra O2 usage)
- Terrain obstacle (detour = extra distance)

### Mechanical
- Rover malfunction (needs parts to fix)
- Tire blowout (lose travel speed temporarily)
- Comms array failure (no outpost info for next stop)
- Life support glitch (O2 spike)

### Crew
- Crew member illness (needs med kit or health degrades)
- Injury from accident (needs med kit)
- Crew argument (morale drop)
- Crew finds something useful (bonus supplies)
- Crew member has an idea (unique bonus from their role)

### Encounters
- Abandoned rover (scavenge for parts/supplies)
- Distress signal (help = lose supplies, gain morale; ignore = morale hit)
- Strange geological formation (geologist bonus)
- Old colony ruins (lore event, possible supplies)

---

## AI Agent System

The game includes AI agents that control NPC behavior. Each agent has:

### Agent Architecture
```
Agent {
  name: string
  personality: string        // e.g., "gruff trader", "nervous scientist"
  memory: []                 // remembers past interactions
  goals: []                  // what they want to achieve
  decisionEngine(context) → action   // picks what to say/do
}
```

### Agent Types

1. **Outpost Merchant Agent**
   - Sets prices based on supply/demand (tracked per playthrough)
   - Remembers if player drove a hard bargain
   - May offer deals or refuse trade based on past interactions
   - Personality affects dialogue tone

2. **Crew Member Agents**
   - Each crew member has an AI agent that suggests actions
   - Suggestions are influenced by their role + personality
   - They react to events ("As your engineer, I think we should slow down")
   - Their morale affects the quality of suggestions
   - Can refuse orders at very low morale

3. **Event Agent (Game Master)**
   - Selects and narrates random events
   - Adjusts difficulty based on player's current state
   - Ensures pacing (no 5 disasters in a row, ramp up near end)
   - Generates descriptive text for events and scenery

4. **Encounter NPCs**
   - One-off characters met during events
   - Have a brief personality and goal
   - Interact for one conversation then leave

### Agent Decision Engine
Agents use a weighted scoring system for mechanical decisions (pricing, event selection, difficulty):
```
score(action) = 
  baseWeight(action) 
  + personalityModifier(action, personality) 
  + memoryModifier(action, pastInteractions)
  + contextModifier(action, gameState)
```
The highest-scoring action is selected, with some randomness for variety.

### LLM-Powered Dialogue

All agent dialogue is generated via the **OpenAI API** (GPT-4o-mini for speed, GPT-4o available as an option). The LLM generates contextual, personality-driven text for all agent types.

#### API Key
- Player enters their OpenAI API key on the title screen (stored in `localStorage`)
- A "Settings" option on the title screen allows updating the key
- The API key is required to play — no fallback to template-based dialogue
- Key is sent directly to OpenAI from the browser (no backend proxy)

#### LLM Integration Points

| Agent Type | LLM Usage |
|------------|-----------|
| Crew Members | Contextual advice, reactions to events, banter, morale-driven commentary |
| Outpost Merchants | Haggling dialogue, personality-driven trade talk, deal offers |
| Event Agent (Game Master) | Narrating random events, describing scenery, outcome text |
| Encounter NPCs | One-off character conversations with generated personality |

#### Prompt Architecture
Each LLM call includes a system prompt with:
- Agent identity (name, role, personality trait)
- Current game state summary (resources, crew health, distance, morale)
- Agent memory (last 10 relevant interactions)
- Tone directive ("retro sci-fi terminal aesthetic, terse and atmospheric")
- Response length constraint (1-3 sentences for crew remarks, up to a paragraph for events)

Example system prompt for a crew agent:
```
You are Eng. Okafor, the rover engineer on a Mars expedition. You are pragmatic
and dry-humored. The crew is on day 34, 1,400 km into a 3,200 km journey.
Rover HP is at 40%. Oxygen is low. Respond in-character in 1-2 sentences.
Use a retro sci-fi terminal communication style.
```

#### API Configuration
```
LLM_CONFIG = {
  provider: 'openai',
  model: 'gpt-4o-mini',           // default, user can switch to gpt-4o
  endpoint: 'https://api.openai.com/v1/chat/completions',
  maxTokens: 150,                  // keep responses snappy
  temperature: 0.8,                // some creativity
  retryAttempts: 2,                // retry on transient failures
  timeoutMs: 10000                 // 10s timeout per call
}
```

#### Error Handling
- On API failure: display "[Comms interference — static on the line]" as in-universe fallback
- Rate limit / timeout: queue and retry with exponential backoff
- Invalid key: prompt player to re-enter on next interaction

---

## UI Design

### Layout (Retro Terminal Aesthetic)
```
┌─────────────────────────────────────────────┐
│  MARS TRAIL v1.0          Sol 47  Day 23    │
├─────────────────────────────────────────────┤
│                                             │
│  [Landscape/Scene ASCII Art Area]           │
│                                             │
├─────────────────────────────────────────────┤
│  ┌─Status──────────┐ ┌─Crew───────────────┐│
│  │ Distance: 920km │ │ Cmdr. Chen   ♥♥♥♥  ││
│  │ To Go: 2,280km  │ │ Dr. Vasquez  ♥♥♥   ││
│  │ Pace: Steady    │ │ Eng. Okafor  ♥♥♥♥♥ ││
│  │ Rover HP: 85%   │ │ Geo. Tanaka  ♥♥♥♥  ││
│  └─────────────────┘ └────────────────────┘│
│  ┌─Supplies────────────────────────────────┐│
│  │ O2: ████░░ 12  Food: █████░ 8          ││
│  │ H2O: ███░░░ 6  Parts: ██░░░ 3          ││
│  │ Med: ██░░░░ 2  Batt: ████░░ 7          ││
│  └─────────────────────────────────────────┘│
├─────────────────────────────────────────────┤
│  [Narrative Text / Event Description]       │
│  [AI Agent Dialogue]                        │
│                                             │
├─────────────────────────────────────────────┤
│  [1] Travel  [2] Rest  [3] Status          │
│  [4] Talk to Crew  [5] Check Map           │
│                                             │
│  > _                                        │
└─────────────────────────────────────────────┘
```

### Visual Style
- Dark background (#0a0a0a) with green/amber terminal text
- CRT scanline effect (CSS)
- Pixel/monospace font (e.g., "Press Start 2P" or "VT323")
- ASCII art for landscapes (craters, mountains, dust storms, outposts)
- Animated text display (typewriter effect for narrative)
- Progress bar showing distance traveled on a map strip
- Health shown as heart/shield icons or bars

### Screens
1. **Title Screen** — ASCII art logo, "Press Enter to Begin"
2. **Crew Selection** — Card-style crew roster
3. **Supply Store** — Shopping interface with budget tracker
4. **Travel Screen** — Main gameplay view (layout above)
5. **Event Screen** — Narrative overlay with choices
6. **Outpost Screen** — Shop + NPC dialogue
7. **Game Over / Victory** — Stats summary, score

---

## Technical Architecture

### Stack
- **Single HTML file** with embedded CSS and JavaScript
- No build tools or frameworks
- Vanilla JS with ES6+ features
- All game data defined in JS objects/arrays
- AI agents implemented as JS classes with decision logic + OpenAI API for dialogue
- State machine for game flow
- **External dependency:** OpenAI API (requires user-provided API key)

### File Structure
```
mars-trail/
├── index.html          # Single-file game (HTML + CSS + JS)
├── SPEC.md             # This specification
├── TASKS.md            # Implementation task list
└── README.md           # Project overview and how to play
```

### Code Architecture
```
JavaScript Modules (all in one file, organized by section):

1. CONFIG         — Game constants, balance numbers
2. GameState      — Central state object, save/load
3. Agent          — Base AI agent class
4. MerchantAgent  — Outpost trader logic
5. CrewAgent      — Crew member AI
6. EventAgent     — Random event selection/narration
7. LLMClient      — OpenAI API wrapper, prompt builder, error handling
8. GameEngine     — Core game loop, resource calc, travel
9. UIController   — DOM manipulation, rendering, animations
10. AsciiArt      — Landscape/scene art data
11. Main          — Init, event listeners, start
```

---

## Scoring

At game end, calculate a score:

```
score = (crewSurvived × 500)
      + (daysUnderPar × 100)       // par = 80 days
      + (remainingSuppliesValue)
      + (eventsHandledWell × 50)
      - (crewLost × 300)
      + difficultyBonus
```

Display a rank:
| Score | Rank |
|-------|------|
| 3000+ | Mars Legend |
| 2000-2999 | Colony Hero |
| 1000-1999 | Survivor |
| 500-999 | Barely Made It |
| < 500 | Dust in the Wind |

---

## Future Enhancements (Out of Scope for v1)
- Multiple difficulty levels
- Persistent high score board (localStorage)
- Sound effects (retro beeps)
- Multiplayer crew decisions
- Mobile touch controls
- Procedurally generated terrain/events
