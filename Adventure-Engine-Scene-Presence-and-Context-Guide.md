# Adventure Engine — Scene Presence & Context: The Complete Guide

*Covers Adventure Engine v0.9.9.2 / AE Bridge HUD*

This is the companion to the Resources & Actions guide. That one explained the rules the world runs on. This one explains the other half: **what the AI actually sees when it writes the next paragraph**, and how the engine decides which slivers of your enormous world to hand it.

If you take one idea away from this guide, take this: **the AI does not see your world. It sees a small, carefully assembled summary of the part of your world that matters right now.** Almost every design decision in the engine exists to keep that summary short, current, and relevant — because a language model only has room for so much text, and burning that room on a tavern the player left three hours ago means there's less room for the person standing in front of them.

---

## Part 1 — The mental model: the camera, the dossier, and the encyclopedia

Before any specifics, here's the picture everything else hangs on. The engine sorts every piece of information in your world into one of three jobs.

- **The camera (the live scene).** Where the player physically is *right now*, and who is physically in the room with them. This is the present tense — it changes every time the player moves. It's short, and it's rebuilt from scratch on every single turn.
- **The dossier (who someone is).** The stable facts about a character or place that don't change turn to turn: a blacksmith's gruff personality, a ruined keep's history. This only needs to be in front of the AI *when that character or place is actually relevant*, and it should read the same on turn 2 as on turn 200.
- **The encyclopedia (deep background).** The lore nobody needs in every scene — a faction's secret history, a religion's founding myth, a region's politics. This should appear *only when the story mentions it*, and stay invisible otherwise.

The engine has a different storage location for each job, and the entire system is built around routing each fact to the right one. Getting this routing right is what makes the difference between an AI that feels like it remembers your world and one that feels like it has amnesia or, worse, drowns in irrelevant detail.

Here's the punchline up front, and the rest of the guide is the explanation:

| The job | Where it lives | When the AI sees it | Rebuilt how often |
|---|---|---|---|
| The live scene (camera) | **Author's Note** | Every single turn | From scratch, every turn |
| Who's-here identity (dossier) | **Lorebook** (force-activated) | When that character/place is present | Written once, stays stable |
| Deep background (encyclopedia) | **Lorebook** (keyword-triggered) | When a trigger word appears in the story | Written once, stays stable |
| Permanent world canon | **Memory** | Always | Written once at world-build time |

---

## Part 2 — What is "the scene," really?

The word **scene** has a precise meaning in the engine, and it's narrower than you might guess.

### 2.1 The scene is one room

Every location in your world is a node in a tree. A region contains settlements; a settlement contains buildings; a building contains rooms. The player's body sits somewhere in that tree. **The scene is the single innermost location the player's body is currently inside** — not the region, not the building, just the room.

When the engine needs to know "what room is the player in," it doesn't just read their position directly. It walks *up* the tree from wherever the player's body is until it hits the first thing marked as a location, and that's the room. (Internally this is the `getEffectiveRoomId` step.) This walk-upward design is what lets a player stand inside a *vehicle* that is itself parked inside a *town square*: the vehicle is the innermost location, so the vehicle is the scene, and the town square is just its container. Move the vehicle, and the scene moves with the player automatically — because "what room am I in" is always answered relative to where the body currently sits in the tree.

### 2.2 Scene presence — who's in the room

A character is **scene-present** when their body is in the same room as the player's body. That's the whole rule. The engine keeps a true/false `scenePresent` flag on every character and recomputes it whenever the player moves: it finds the player's room, then for every active character it checks whether *their* room (resolved by the same walk-upward) matches. Same room → present. Different room, or no body in the world at all → not present.

This recomputation (`recalculateScenePresence`) runs automatically at every moment that could change who's in the room: after travel, after an effect teleports someone, after a manual location change, after the clock advances NPCs along their schedules. You never set scene presence by hand in normal play — the engine derives it from physical co-location, every time.

### 2.3 The one manual override

There is a manual scene-presence toggle in a character's editor, but the UI itself warns you it's temporary: *"This will be automatically reset to the correct value when you change locations."* It exists for momentary authoring fixes, not for storytelling. The instant the player travels, physical co-location reasserts itself and your override is gone. Treat it as a debugging convenience, not a tool.

---

## Part 3 — The Author's Note: the live scene snapshot

The **Author's Note** (often shortened to **AN**) is a special slot NovelAI keeps very close to the most recent story text — meaning the AI weights it heavily. The engine treats the AN as **the live scene snapshot**: a small block, rebuilt completely on every turn, describing the present moment and nothing else.

### 3.1 Why the AN is rebuilt from scratch every turn

This is the single most important property of the AN, so it's worth dwelling on. Everything in the AN is *dynamic* — it changes as the story moves. The current room. Who's present. What each present NPC is feeling and scheming this very moment. The objective the player is chasing. The time of day. None of that is true for long, so none of it can be written down once and left alone. The engine throws the whole block away and reassembles it fresh before every generation (this happens in the `onBeforeContextBuild` hook). That guarantees the AI never reads a stale "the guard is suspicious of you" line three scenes after you already won the guard over.

### 3.2 What goes into the AN, in order

The engine assembles the block top to bottom in a deliberate order, because text nearer the player's input carries slightly more weight. The order is:

1. **The current objective** (if you've pinned a quest). Leads the block so the AI's north star sits at the very top. If the objective has a destination, a turn-by-turn route is computed and appended (`Tavern → Market Road → Keep`), and if it has a deadline, a live countdown rides along (`(2 days remaining)`, `(due today)`, `(overdue by 1 day)`).
2. **The world clock** (if enabled) — a single date/time stamp.
3. **The location line** — the current room's name, its current-state description, and its atmosphere. This is the "camera view": what the player physically senses on arrival.
4. **The scene roster** — a single line listing everyone present with the player by name (`Present with you: Gareth, Mira.`).
5. **One compact dynamics line per present NPC** — covered next.

### 3.3 The per-NPC dynamics line — the heart of the AN

For each scene-present NPC, the engine emits *one* tight line describing only their **living dynamics** — the things about them that are true *this turn and maybe not the next*. The default shape is:

> `[Name — relationship archetype, stage with target (active conditions). Goal: current desire. Thinking: immediate thought]`

Each piece is a different "speed" of change, and the engine tracks them on separate internal clocks:

- **Relationship archetype & stage** — slow. *Rival → Wary Ally*. Changes across many scenes.
- **Conditions** — the active status effects from their resources (*poisoned, drunk*). Index-0 "nothing's wrong" states are silently dropped so the line stays clean when nothing is active.
- **Goal (the mid-range layer)** — medium. A sustained desire shaping their decisions across the current stretch of story.
- **Thinking (the ephemeral layer)** — fast. Their immediate internal state at this exact moment, regenerated frequently as the scene unfolds.

**Why split a character's psychology into three speeds?** Because they decay at wildly different rates, and refreshing them all on the same schedule would be wasteful and incoherent. A character's *immediate thought* should churn almost every turn. Their *standing goal* should hold for a scene or two. Their *relationship* should shift only on meaningful beats. By layering them, the engine can cheaply update the fast stuff often while leaving the slow stuff stable — and the AI reads a character who has both a fleeting mood *and* a throughline, which is what makes them feel like a person instead of a mood ring.

Critically: if an NPC's line renders to nothing but their bare name — no relationship, no goal, no thought worth mentioning — the engine **drops the line entirely**. The roster already named them as present; a content-free detail line would just be noise. Silence is a feature.

### 3.4 What the AN deliberately does NOT contain

The AN is the live layer, so static identity is banished from it on purpose. A character's physical description, personality, voice, and backstory are *not* in the AN — those don't change turn to turn, so putting them here would mean re-paying for the same unchanging text on every single generation while also crowding out the dynamic content that actually needs the prime real estate. Identity lives in the lorebook instead (Part 4). The AN carries *only* the layer that moves.

### 3.5 The AN is shared, so the engine is a polite tenant

The Author's Note isn't the engine's private property — you might be using it yourself, and the World-Build Wizard writes a "Style & Voice" region into it too. So the engine never clobbers the whole note. It marks its own contribution with a hidden sentinel and, every turn, strips out *only its own previous block* before appending a fresh one. Anything you wrote by hand survives untouched, and the engine's block never piles up into duplicates no matter how many times the hook runs.

---

## Part 4 — The Lorebook: identity and deep background

The **Lorebook** is NovelAI's keyword-triggered context system. Each lorebook entry has a body of text and a set of **keys** (trigger words). Normally, when one of those keys appears in recent story text, the entry's body gets injected into context; when no key is present, the entry stays silent and costs nothing. The Adventure Engine uses the lorebook for the two *stable* jobs from Part 1: **identity (the dossier)** and **deep background (the encyclopedia)**.

### 4.1 Character identity lives in the lorebook

Every character can have a linked lorebook entry holding their static dossier: the identity line (`Gareth — human male, 34, blacksmith`), physical description, personality core, voice and mannerisms, background, core values, and any custom fields you've defined. This is written once when you create the character and re-synced only when you actually edit those fields — it is otherwise rock-stable, which is exactly what you want for facts that don't change.

**The clever part — force-activation.** A normal lorebook entry only fires when its keyword happens to appear in the prose. But the engine doesn't want to gamble on whether the AI happened to type "Gareth" last paragraph — if Gareth is standing right there, the AI needs his dossier *guaranteed*. So on every turn, for every scene-present character (and the player), the engine reaches into the lorebook and **force-activates** their entry, switching it on regardless of whether its keyword appeared. Present in the room → dossier guaranteed in context. Not present → the entry falls back to ordinary keyword behavior, so it can still fire if the story merely *mentions* an absent character by name.

This is the elegant division of labor that makes the whole system work: **the AN says "Gareth is here and he's furious with you"; the lorebook says "Gareth is a 34-year-old blacksmith with burn-scarred forearms and a slow temper."** Live state in the note, stable identity in the lorebook, each refreshed on its own appropriate schedule, never duplicating the other.

### 4.2 The fallback for unlinked characters

Not every character needs a permanent lorebook entry — a one-scene nobody doesn't warrant one. For a scene-present character with no linked entry, the engine still builds their static dossier on the fly and injects it as a **temporary** (single-generation) lorebook entry, then discards it. So presence always guarantees identity in context, whether or not you bothered to create a permanent entry. You never get a character standing in the room with the AI knowing nothing about them.

### 4.3 Equipment is ephemeral on purpose

What a character is wearing or carrying *changes*, so it doesn't belong in their stable dossier. Equipment is emitted fresh each generation as its own temporary lorebook entry (`Gareth is wearing/carrying: leather apron, smith's hammer.`). Keeping it out of the permanent entry means equipping or unequipping an item never forces a re-sync of the stable dossier — the identity entry stays content-stable and cache-friendly, and gear updates ride a cheap throwaway entry instead. This is the same "fast layer vs. slow layer" instinct from the AN applied to the lorebook.

### 4.4 Location, item, faction, and species deep lore

Locations, items, factions, and species can each carry a lorebook entry too, and here the design draws a sharp line you should internalize:

- A location's **description and atmosphere** (the camera view) inject via the AN *while the player is standing there*. They answer "what does this place look like right now."
- A location's **lorebook entry** holds its *deep background* — history, culture, secrets, politics — and is **deliberately the separate `Lorebook Entry` text field, NOT the description**. It fires on its keyword whenever the place is *named* in the story, even from afar.

The reason they're kept strictly separate is to avoid double injection. While you're in the Old Keep, its description is already in the AN; if its lorebook entry *also* repeated that description, the AI would read the same paragraph twice and waste the room on it. So the rule the engine enforces, and the rule the World-Build Wizard coaches you toward, is: **the immediate description is what a camera senses on arrival; the lorebook entry is the hidden history a camera can't see, and the two must never duplicate each other.** Faction and species entries work the same way — a faction entry holds its public-vs-private tensions and standings ladder; a species entry holds its anatomy and cultural drives. None of it needs to be present every turn; all of it should surface the moment the story names it.

### 4.5 Why this split matters: triggered vs. always-on

The deepest reason for routing background to the lorebook rather than the AN is **cost discipline**. The AN is paid for on *every* turn. A lorebook entry is paid for *only on turns where its trigger word appears*. Your world might have two hundred locations and fifty factions; if all that lore were always-on, it would be both unaffordable and incoherent. By making deep lore keyword-triggered, the engine lets you author an essentially unlimited encyclopedia that stays completely silent until the story reaches for a specific piece of it — at which point that one piece, and only that one piece, surfaces.

---

## Part 5 — Memory: the permanent canon

**Memory** is NovelAI's always-on, top-of-context block. The engine reserves it for the handful of facts that are true *everywhere, always, for the whole story*: the setting's name and scope, its core themes, the central conflict, the rules of romance and combat, and the macro stakes. The World-Build Wizard assembles these from your answers and commits them **once**, as static world canon. (The wizard's "Style & Voice" output is the exception — it targets the Author's Note instead, since voice is a steering instruction rather than a fact.)

The distinction between Memory and the lorebook is the distinction between **canon that frames everything** and **canon that's only sometimes relevant**. "This is a world where the dead don't stay dead" belongs in Memory — it colors every scene. "The Ashguard mercenary company was founded after the Siege of Vael" belongs in a lorebook entry — it matters only when the Ashguard come up. If you find yourself putting location- or character-specific detail in Memory, that's a sign it should be a lorebook entry instead; Memory is for the immutable frame, not the furniture.

---

## Part 6 — Travel: how moving rebuilds the world

Travel is where scene presence, the AN, and the lorebook all visibly snap into action, so it's the best way to see the whole system move at once.

### 6.1 What actually happens when the player travels

When the player commits a move to a new location, the engine runs a fixed sequence (`commitTravel`):

1. **Move the body.** The player's body entity is re-parented into the destination. This single change is what makes everything downstream true.
2. **Update the camera.** The player's "current location" pointer is set to the destination.
3. **Run arrival auto-triggers.** Any Auto actions waiting at the destination get their chance to fire *before* the AI writes (so an ambush or a greeting is already staged).
4. **Reposition scheduled NPCs.** Before deciding who's present, the engine advances offstage NPCs to wherever their hourly schedule says they should be at the current time (covered in 6.3).
5. **Recompute scene presence.** Now that bodies are in their final positions, the engine recomputes the `scenePresent` flag for everyone against the new room.
6. **Generate.** The next turn builds — and because steps 1–5 already ran, the fresh AN describes the new room with its new roster, and the lorebook force-activates the dossiers of whoever's now present.

Notice the ordering discipline: *bodies move first, presence is computed last, generation happens after both.* That's why an Inject-Story-Text effect in a travel action must be placed **before** the Move-Player effect — the narration of leaving should be written while you're still mechanically "here," and the move itself is what hands off to the new scene. (The Resources & Actions guide flags this same ordering rule from the action-authoring side.)

### 6.2 Travel tiers — why a long journey and a short step feel different

Not all movement is equal. Stepping from a tavern's common room into its back room is not the same as crossing a continent, and the engine lets them feel different through **travel tiers**. The depth of a destination in the world tree picks which travel template runs:

- **Tier 1 (top-level / regions)** — the big journeys between major places. *"You set out for the Ashlands."*
- **Tier 2 (sub-locations)** — moving around within a region or settlement. *"You head to the market."*
- **Deeper / default** — the small steps between adjacent rooms. *"You leave for the back room."*

Each tier has its own default template (editable in the Action Library like any other), and through the Locations wizard each can carry its own **entry cost** (spend a resource to enter — a toll, fuel, stamina), its own **time cost** (hours added to the world clock on arrival), and its own **narration setting** (a grand journey gets narrated; a one-step shuffle can be silent so it doesn't clutter the prose with a paragraph for every doorway). This is how you make crossing the wilderness *cost* something and *take* time while walking across a room stays free and instant — the same travel machinery, tuned per scale.

### 6.3 Schedules — a world that moves while you don't

NPCs don't have to stand frozen where you left them. A character can carry an **hourly schedule** — a timetable of which location they should be in at which hour. Whenever travel commits or the clock is advanced manually, the engine quietly walks each scheduled offstage NPC to wherever their timetable says they belong for the current hour, *before* recomputing presence. The blacksmith is at the forge by day and the tavern by night, with no per-turn overhead, because schedules are only resolved at those movement/time checkpoints rather than every generation.

Two deliberate safeguards make this feel natural rather than eerie:

- **No vanishing while watched.** A scheduled NPC who is *currently in the room with the player* is left exactly where they are. People don't blink out of existence mid-conversation because the clock ticked.
- **Arrivals you can wait for.** Schedules *can* move an NPC *into* the player's room. So a player who knows the captain returns to the hall at dusk can simply wait there, and the captain will walk in — the world keeping its own appointments independent of the player.

### 6.4 Travel paths and the objective route

When you've pinned an objective whose target is somewhere else in the world, the engine computes the actual route through the location tree from where you are to where the target is, and renders it into the AN's objective line (`Tavern → Market Road → Keep`). The player (and the AI) always knows not just *what* the goal is but *how to physically get there* from the current room — recomputed every turn as the player closes the distance.

---

## Part 7 — Knowledge: who knows what about whom

There's one more layer worth understanding, because it's subtle and easy to miss: the engine separates *what is true about an entity* from *what a given character has learned about it*.

Each entity can carry a **knowledge string** — a short statement of the established, knowable facts about it. Each character keeps a list of **known entities**, and when a character becomes *aware* of an entity, a snapshot of that entity's knowledge string is copied into their personal record. The point of copying rather than referencing is that knowledge is **per-character and frozen at the moment of learning**: the AI can be told what a specific character knows about a specific thing without assuming everyone knows everything, and without that knowledge silently updating if the underlying fact later changes. When it's relevant, the engine can surface a one-time line — `[Mira's established knowledge of the Old Keep: ...]` — so the AI writes a character who knows what they should know and *doesn't* know what they shouldn't.

The design reason is dramatic integrity. A world where every character automatically knows every fact is a world with no secrets, no discovery, and no dramatic irony. By tracking knowledge as a per-character, point-in-time snapshot, the engine preserves the gap between what the world knows and what *this person* knows — which is where most of a story's tension actually lives.

---

## Part 8 — Putting it together: where does each thing go?

Here's the practical routing table for the question you'll ask most often — *"I have this piece of information about my character / place / faction; where do I put it?"* The guiding question is always **how often does this change, and when is it relevant?**

| You have... | It goes in... | Because... |
|---|---|---|
| A character's looks, personality, voice, backstory | **Lorebook entry** (character) | Stable identity — stable, and needed only when they're present |
| A character's current mood, immediate thought | **AN** (ephemeral layer, automatic) | Fastest-changing layer; rebuilt every turn |
| A character's standing goal this stretch of story | **AN** (mid-range layer, automatic) | Medium-speed; holds a scene or two |
| A character's relationship with the player | **AN** (relationship line, automatic) | Slow but live; shifts on meaningful beats |
| What a character is wearing/carrying | **Lorebook** (ephemeral equipment entry, automatic) | Changes, so kept off the stable dossier |
| What a place looks/feels like on arrival | **Location description + atmosphere** → AN while present | The camera view; injected only while you're there |
| A place's hidden history, culture, secrets | **Lorebook entry** (location, the separate lore field) | Encyclopedia — fires when the place is named; must not duplicate the description |
| A faction's public face vs. private reality, standings | **Lorebook entry** (faction) | Relevant only when the faction comes up |
| A species' anatomy and cultural drives | **Lorebook entry** (species) | Relevant only when the species comes up |
| What a specific character has learned about a thing | **Knowledge string → known-entity snapshot** (automatic on awareness) | Per-character, point-in-time; preserves secrets |
| The setting's themes, central conflict, world rules | **Memory** (via the World-Build Wizard) | Immutable frame — true everywhere, always |
| A steering instruction about prose voice/style | **AN** (Style region, via the Wizard) | A direction, not a fact; wants high weight |

If you're ever unsure, run the two-question test: **Does this change turn-to-turn?** If yes → it's AN material (and usually the engine handles it automatically). If no → it's lorebook or Memory. **Is it true everywhere, or only sometimes relevant?** Everywhere → Memory. Only when named → lorebook. That's the entire routing logic in two questions.

---

## Quick-Reference Cheat Sheet

**The three jobs:** Camera (live scene, → AN) · Dossier (stable identity, → Lorebook force-activated) · Encyclopedia (deep lore, → Lorebook keyword-triggered)

**The scene** = the single innermost location the player's body is in (resolved by walking *up* the tree to the first location).

**Scene-present** = a character whose body is in the player's room. Derived automatically from co-location, recomputed on every move. Never set it by hand for storytelling.

**Author's Note carries (all dynamic, rebuilt every turn):** objective + route + countdown · clock · location description + atmosphere · present-roster line · one dynamics line per NPC (relationship · conditions · goal · thought).

**Author's Note never carries:** static identity (looks, personality, voice, backstory) — that's lorebook.

**Lorebook carries (stable, written once):** character dossiers (force-activated when present) · location/item/faction/species deep lore (keyword-triggered when named) · ephemeral equipment (throwaway each turn).

**The golden rule for places:** description = what a camera senses now (→ AN while present); lorebook = hidden history (→ fires when named). Never duplicate.

**Memory carries:** immutable world canon — themes, central conflict, world rules. Committed once at world-build time.

**Travel sequence:** move body → update camera → arrival auto-triggers → resolve schedules → recompute presence → generate. (Inject narration *before* the move effect.)

**Travel tiers:** Tier 1 (regions, big journeys) · Tier 2 (sub-locations) · Deeper/default (room-to-room). Each can set entry cost, time cost, and whether the step is narrated or silent.

**Schedules:** NPCs follow an hourly timetable; resolved on travel/time-change only. Never vanish someone the player is watching; can walk NPCs *into* the player's room so arrivals can be waited for.

**Knowledge:** per-character, point-in-time snapshot of an entity's facts — preserves secrets and dramatic irony.

**The two-question routing test:** Changes turn-to-turn? → AN. Otherwise → true everywhere? Memory : only-when-named? Lorebook.
