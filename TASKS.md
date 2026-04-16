# Mars Trail — Implementation Tasks

> Each task is a self-contained unit of work. Complete them in order.
> Status: ⬜ Not Started | 🔧 In Progress | ✅ Done
> Note: Task IDs are renumbered to match the actual implementation order in this file.

---

## Milestone 1: Shell & Core Systems

### Task 1.1 — HTML Skeleton & CSS Retro Theme
✅ Create `index.html` with the full page structure
- Title bar, scene area, status panels, narrative area, action bar, input prompt
- Dark terminal background (#0a0a0a)
- Green (#00ff41) and amber (#ffb000) text colors
- CRT scanline overlay effect (CSS pseudo-element)
- Import monospace/retro font (VT323 from Google Fonts)
- Responsive layout (centered, max-width ~800px)
- All panels match the spec wireframe layout

**Acceptance:** Opening `index.html` shows the styled empty game shell with scanlines and retro font.

---

### Task 1.2 — Game State & Configuration Constants
✅ Define `CONFIG` object with all balance numbers from spec
- Travel paces and their km/day, O2 modifiers, risk levels
- Supply costs, depletion rates per day per crew
- Distance total (3,200 km), outpost positions
- Event probabilities per pace
- Scoring formula constants

✅ Define `GameState` class/object
- Properties: day, distance, pace, supplies (O2, food, water, parts, medkits, batteries), crew array, rover HP, morale, credits, currentPhase, eventLog
- `reset()` method to initialize a new game
- `serialize()` / `deserialize()` for potential save/load

**Acceptance:** `GameState` can be created, reset, and all config values are accessible.

---

### Task 1.3 — Screen Manager & Transitions
✅ Implement screen state machine
- Screens: title, crewSelect, supplyStore, travel, event, outpost, gameOver, victory
- `showScreen(screenName)` — hide all, show target
- Smooth transitions (fade or slide)
- Keyboard input handler (number keys for menu choices)
- Input prompt at bottom of screen

**Acceptance:** Can navigate between all screens with keyboard input.

---

### Task 1.4 — API Key Entry Screen
⬜ API key input on title screen
- "Enter OpenAI API Key" field before game starts
- Key is masked (password-style input)
- "Settings" link on title screen to update key later
- Store in `localStorage`, load on return visits
- Validate key format and test with a lightweight API call
- Show clear error if key is invalid or missing

**Acceptance:** Player can enter, save, and update their API key. Game won't start without a valid key.

---

### Task 1.5 — Base Agent Class
✅ Create `Agent` base class
- Properties: name, personality, memory (array), goals (array)
- `remember(event)` — push to memory, trim to last 20 entries
- `scoreAction(action, context)` — weighted scoring with personality/memory/context modifiers
- `decide(actions, context)` — return highest-scoring action with slight randomness
- `getDialogue(context)` — return a personality-flavored text string

**Acceptance:** Can instantiate an `Agent`, feed it actions + context, and get a decision back.

---

### Task 1.6 — LLM Client (OpenAI Integration)
✅ Create `LLMClient` class
- Wraps OpenAI Chat Completions API (`gpt-4o-mini` default, `gpt-4o` option)
- `sendPrompt(systemPrompt, userMessage, options)` → returns response text
- Builds system prompts from agent identity + game state context + memory
- Config: `maxTokens: 150`, `temperature: 0.8`, `timeoutMs: 10000`
- Retry logic: 2 retries with exponential backoff on transient failures
- Error handling: return in-universe fallback text (`"[Comms interference — static on the line]"`)
- Rate limit awareness: queue requests, avoid parallel bursts

✅ Integrate LLM into all agent types
- `Agent.getDialogue(context)` calls `LLMClient.sendPrompt()` with agent-specific system prompt
- CrewAgent: advice, banter, event reactions (1-2 sentences)
- MerchantAgent: haggling dialogue, trade talk (1-3 sentences)
- EventAgent: event narration, scenery descriptions (up to 1 paragraph)
- Encounter NPCs: one-off character dialogue (2-3 sentences)

✅ API key management
- Store/retrieve key from `localStorage`
- Validate key with a lightweight test call on entry
- Surface invalid key errors clearly to the player

**Acceptance:** All agents generate contextual LLM-powered dialogue. API errors produce graceful in-universe fallback messages.

---

## Milestone 2: Onboarding Flow

### Task 2.1 — Title Screen & Crew Selection UI
✅ Title screen with ASCII art Mars logo
- "Press Enter to Begin" prompt
- Animated text flicker effect

✅ Crew selection screen
- Display 6 crew candidates as cards (name, role, trait, skill)
- Player selects 3 by pressing number keys
- Show selected crew with confirmation prompt
- Player names their commander

**Acceptance:** Player can start game, name commander, and pick 3 crew members.

---

### Task 2.2 — Crew Member Agents
✅ Create `CrewAgent` extending `Agent`
- Role-specific bonuses (Engineer: +repair, Medic: +heal, etc.)
- Suggestion generation based on current game state
  - Low O2 → "We should slow down to conserve oxygen"
  - Damaged rover → Engineer suggests repair
  - Sick crew → Medic urges using med kit
- Morale affects suggestion quality and willingness
- Personality traits affect dialogue style (optimistic, cautious, sarcastic, etc.)
- Define 6 crew candidate profiles (pick 3 at game start)

**Acceptance:** `CrewAgent`s produce context-appropriate suggestions with personality flavor.

---

### Task 2.3 — Supply Store UI
✅ Shopping interface
- List items with prices and current stock
- Player types number to buy, shows running budget
- Quantity selection (buy 1-N of each item)
- "Done Shopping" to proceed
- Budget bar showing remaining credits

**Acceptance:** Player can buy supplies within budget and proceed to travel.

---

## Milestone 3: First Playable Travel Loop

### Task 3.1 — Resource Management
✅ Implement daily resource consumption
- Deduct O2, food, water, batteries based on crew count and pace
- Apply zero-resource effects (health damage, morale drops)
- Track crew health individually (0-100 HP each)
- Crew member death at 0 HP (remove from active crew, reduce consumption)
- Rover HP degradation from events
- Morale calculation (hidden, affects crew agent behavior)

**Acceptance:** Resources deplete correctly per turn. Crew health responds to shortages.

---

### Task 3.2 — Travel System
✅ Implement travel turn logic
- Player picks pace → calculate distance traveled
- Advance day counter
- Consume resources
- Check for random event (via `EventAgent`)
- Check for outpost arrival
- Check win/lose conditions
- Update all UI panels

**Acceptance:** Each "Travel" action advances the game one day with correct state changes.

---

### Task 3.3 — Travel Screen (Main Game View)
✅ Implement the main gameplay UI from spec wireframe
- ASCII art landscape area (rotating Mars scenes)
- Status panel: distance, pace, rover HP, day counter
- Crew panel: names with health bars/hearts
- Supply bars with color coding (green/yellow/red)
- Narrative text area with typewriter effect
- Action menu: [1] Travel [2] Rest [3] Status [4] Talk to Crew [5] Check Map
- Distance progress bar (mini map strip)

**Acceptance:** Travel screen displays all info correctly and updates each turn.

---

### Task 3.4 — Crew Talk & Status Screens
✅ "Talk to Crew" screen
- List crew members, select one to talk to
- `CrewAgent` generates context-aware dialogue
- May include advice, complaints, lore, jokes based on personality
- Morale-dependent responses

✅ "Status" detail screen
- Full resource breakdown with numbers
- Crew health details
- Journey progress
- Recent event log (last 5 events)

**Acceptance:** Both screens show meaningful AI-driven content.

---

### Task 3.5 — Map Screen
✅ "Check Map" displays route
- ASCII map strip showing:
  - Landing zone (start)
  - 5 outposts with names
  - Current position marker
  - New Gale Crater (end)
- Distance labels
- Terrain hints

**Acceptance:** Map accurately reflects current journey progress.

---

## Milestone 4: Events & Outposts

### Task 4.1 — Event Agent (Game Master)
✅ Create `EventAgent` extending `Agent`
- Event pool: environmental, mechanical, crew, encounter events from spec
- Selection logic:
  - Weight by current pace (faster = more danger events)
  - Anti-repeat: don't pick same event twice in 5 turns
  - Pacing: track recent event types, avoid disaster streaks
  - Difficulty ramp: increase danger near end of journey
- Each event has: type, title, description, choices (array), outcomes per choice
- Generate narrative text for each event
- "No event" is a valid outcome (peaceful travel day)

**Acceptance:** `EventAgent` produces varied, context-appropriate events with narrative text.

---

### Task 4.2 — Event Resolution
✅ Implement event choice handling
- Display event description and choices
- Player selects a choice
- Apply outcome: resource changes, crew health changes, distance modifiers
- Crew agent may add commentary
- Log event to history
- Some events have skill checks (crew role bonuses apply)

**Acceptance:** Events present choices, outcomes affect game state, crew comments appear.

---

### Task 4.3 — Merchant Agent
✅ Create `MerchantAgent` extending `Agent`
- Tracks supply/demand per outpost (some items scarce, prices vary)
- Price calculation: `basePrice × scarcityMultiplier × playerReputationMod`
- Remembers if player haggled, bought a lot, or was rude
- Personality affects dialogue ("gruff trader" vs "friendly shopkeeper")
- Can offer special deals or refuse to sell
- Each of 5 outposts gets a unique merchant with its own personality

**Acceptance:** `MerchantAgent` generates price lists that vary by outpost and past player behavior.

---

### Task 4.4 — Outpost System
✅ Implement outpost stops
- 5 outposts at km 640, 1280, 1920, 2560, 3000
- Each has a name and a `MerchantAgent`
- Player can: buy supplies, repair rover, heal crew, talk to merchant NPC
- Prices generated by `MerchantAgent`
- Buying/selling updates inventory and credits
- Outpost names: Olympus Station, Tharsis Depot, Mariner Waypoint, Hellas Rest, Elysium Gate

**Acceptance:** Player can stop at outposts, interact with merchant, buy/sell, and continue.

---

### Task 4.5 — Event & Outpost Overlays
✅ Event overlay
- Darkened background, centered event panel
- Event title, description text (typewriter), choices as numbered options
- Outcome text after selection
- Crew agent commentary appears below outcome

✅ Outpost overlay
- Outpost name and ASCII art
- Merchant dialogue (AI-generated)
- Shop inventory with prices
- Actions: Buy, Repair, Heal, Talk, Leave

**Acceptance:** Events and outposts display as overlays with full interactivity.

---

## Milestone 5: Endgame & Polish

### Task 5.1 — ASCII Art Assets
✅ Create ASCII art for:
- Mars landscape (flat terrain, craters, mountains)
- Dust storm scene
- Rover
- Outpost/station
- Title screen logo ("MARS TRAIL")
- Victory scene (colony)
- Game over scene (broken rover / skeleton)
- At least 5 variants for travel scenery rotation

**Acceptance:** All scenes have appropriate ASCII art that fits the terminal aesthetic.

---

### Task 5.2 — Win/Lose Conditions & Scoring
✅ Implement end-game detection
- Win: distance >= 3200 with >= 1 crew alive
- Lose: all crew dead OR rover HP 0 with no parts OR O2/food/water 0 with no crew health left
- Calculate score per spec formula
- Determine rank
- Track stats: days traveled, events faced, crew lost, supplies remaining

**Acceptance:** Game correctly detects win/lose states and displays a scored summary.

---

### Task 5.3 — Game Over & Victory Screens
✅ Game over screen
- Cause of death / failure reason
- Journey stats summary
- "Press Enter to Try Again"

✅ Victory screen
- Celebration ASCII art
- Score breakdown
- Rank display
- Journey stats
- "Press Enter to Play Again"

**Acceptance:** Both end screens show stats, score, and allow restart.

---

### Task 5.4 — Balance Testing & Tuning
✅ Playtest full game loop multiple times
- Verify resource depletion rates feel fair
- Ensure event frequency/severity is balanced
- Confirm outpost spacing gives enough resupply opportunities
- Check that all paces are viable strategies
- Verify AI agents produce varied and meaningful output
- Test edge cases: all crew dead, no supplies, max everything

**Acceptance:** Game is completable but challenging. No softlocks or broken states.

---

### Task 5.5 — Final Polish
✅ Add typewriter text animation speed control
- Click/key to skip animation and show full text

✅ Add subtle CSS animations
- Blinking cursor on input
- Pulsing warning colors on low supplies
- Screen shake on damage events

✅ Final code cleanup
- Consistent naming conventions
- Remove debug/test code
- Add brief code section comments

**Acceptance:** Game feels polished, responsive, and plays smoothly start to finish.

---

## Task Dependency Graph

```
1.1 ──→ 1.3 ──→ 1.4 ──→ 2.1 ──→ 2.3
1.2 ──→ 3.1 ──→ 3.2 ──→ 3.3
1.5 ──→ 1.6 ──→ 2.2 ──→ 3.4
1.5 ──→ 1.6 ──→ 4.1 ──→ 4.2
1.5 ──→ 1.6 ──→ 4.3 ──→ 4.4
3.2 ──→ 3.5
4.2, 4.4 ──→ 4.5
3.3, 4.2, 4.4 ──→ 5.2
5.1, 5.2 ──→ 5.3
All core systems ──→ 5.4 ──→ 5.5
```
