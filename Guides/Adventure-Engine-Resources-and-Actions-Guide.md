# Adventure Engine — Resources & Actions: The Complete Guide

*Covers Adventure Engine v0.9.9.1 / AE Bridge HUD v0.9.8.5*

This guide explains, in plain language, the two systems that make the Adventure Engine feel alive: **Resources** (the numbers and states the world tracks) and **Actions** (the things that can happen, and the rules for how they happen). No prior game-design or programming knowledge is assumed — every term is explained the first time it's used.

---

## Part 1 — Resources: the world's "stats"

A **resource** is anything in your world that has a value that can go up or down, or that moves through a sequence of named states. Gold, health, a town's food supply, a faction's trust in you, a character's relationship stage — all of these are resources under the hood, just shaped differently depending on what makes sense for that thing.

### 1.1 The three shapes a resource can take

When you create a resource (in **World Settings → Global Resources**, or as a **Character Resource**), you pick one of three shapes:

- **Pool.** A plain number that goes up or down — gold, ammo, a town's food stockpile. You can optionally give it a minimum and maximum (e.g. Health: 0–100). The "+ Add Pool" button creates this kind.
- **Track.** An ordered ladder of named stages the value moves up and down through — for example a Health track of *Healthy → Wounded → Critical → Dead*, or a town's Unrest track of *Calm → Tense → Unrest → Riot*. Instead of a raw number, the resource's value is just "which step on the ladder are we on." The "+ Add Track" button creates this kind.
- **Condition (technically a "discrete" resource).** Works like a Track, but it's meant for status effects rather than a meaningful progression — *None → Poisoned → Cursed → Blessed*, for example. The first state (index 0) is treated as "nothing's wrong" and is automatically left out of any text describing the character, so it won't clutter the story when nothing is active. The "+ Add Condition" button creates this kind.

Every resource also gets a **tag** (a short internal name starting with `#`, like `#gold` or `#health`) that the rest of the engine — conditions, effects, the Stakes system — uses to refer to it. You'll see this tag in dropdown pickers throughout the Action Studio.

### 1.2 Global (party) resources vs. character (personal) resources

There are two places a resource can live:

- **Global Resources** belong to the whole party/world — things like shared gold, a faction's general food supply, or an in-world clock. There's one shared value, and any character or location can read or change it.
- **Character Resources** belong to one specific character — health, stamina, sanity, that character's personal coin purse. Each character has their own independent copy of the value. New characters are automatically seeded with whatever resources are defined in **Character Resources** under World Settings (you can edit that template at any time; a "Sync" option pushes new template resources out to existing characters that predate the change).

If a global resource and a character resource happen to share the same tag, the character-level one is skipped for that character so there's no ambiguity about which one an effect is touching.

### 1.3 Factions

A **Faction** is a named group with an ordered relationship ladder, just like a Track resource — typically something like *Hostile → Wary → Neutral → Friendly → Allied*. The party has one standing with each faction; individual characters can *also* track their own personal standing with a faction independently of the party's (useful when an NPC has their own opinion of you that doesn't match the rest of their faction).

### 1.4 Relationship archetypes & stages

Separately from factions, each character can have a **Relationship Archetype** (a label describing the *kind* of relationship they have with the player or another character — Rival, Mentor, Love Interest, etc.) and a **Stage** — an index into that archetype's own ordered list of stages, running from "earliest/coldest" to "latest/warmest." Actions can read and change both.

### 1.5 The Four Fs (character attributes)

Every character sheet built on the default rule system rates a character in four broad attributes:

- **Force** — raw power and direct pressure: breaking, lifting, intimidating, overpowering, enduring. The brute-force approach.
- **Finesse** — precision and control: sleight of hand, stealth, agility, aim, delicate or careful work. Doing it cleanly.
- **Focus** — mind and will: perception, recall, analysis, concentration, resolve. Noticing, knowing, and holding steady.
- **Flair** — presence and charm: persuasion, performance, deception, style, improvisation. Winning people and the moment.

Whenever an action calls for a dice roll, it leans on one of these four (or another custom skill you've defined), and the character's rating in that attribute becomes the modifier added to the roll.

### 1.6 Stakes — what's actually being risked

A **Stake** is a small, reusable, named "wager" you build once and can attach to any dice-rolling action. It defines exactly what the player risks/gains and what the target risks/gains on each of the three possible 2d6 outcomes (explained next):

- **Strong Hit (10 or higher)**
- **Partial Success (7–9)**
- **Failure (6 or lower)**

For each outcome, you pick a resource (a character resource, a faction standing, or a relationship stage) and a signed amount — positive numbers are gains, negative numbers are costs — separately for "what happens to the player" and "what happens to the target." Leaving a slot blank just means nothing happens there. Stakes are built and managed in the **Stakes Library**, the first section of the Actions tab (more on that below), and they're what the player picks from when they take a freeform "do something risky" action during play.

### 1.7 The default dice engine: 2d6 + a Four F

The engine ships with a built-in resolution system — **2d6 Fail-Forward**, inspired by tabletop games like *Powered by the Apocalypse*. Here's exactly how it works:

1. The player (or an action's author) picks which of the Four Fs is relevant.
2. The engine rolls two six-sided dice and adds the character's rating in that attribute.
3. The total lands in one of three bands:

| Roll total | Result | What it means |
|---|---|---|
| 2–6 | **Failure** | The attempt fails. The story should describe the setback honestly and let it create a complication — no last-second saves. |
| 7–9 | **Partial Success** | The goal is achieved, but at a cost — something must be risked, paid, or sacrificed to get it. |
| 10+ | **Strong Hit** | The attempt clearly succeeds — a clean, decisive outcome with no cost and no complication. |

Critically, *even a failure moves the story forward* rather than just being a dead stop — that's what "fail-forward" means. Each band quietly hands the AI a short stage direction (a "GM note") describing the tone it should take, so the prose the AI writes actually matches the mechanical outcome. This whole system — the band ranges, their labels, and which Stake gets applied — is itself just a regular Action Library template (called **Default Resolution**) that you're free to open and edit if you want to swap in a different rule system entirely (a flat d20-vs-DC system, for instance).

---

## Part 2 — The Actions Tab

### 2.1 Where to find it

Open the engine's sidebar panel (labeled **🗺 Adventure**) and click the **🎬 Action** tab along the top, next to Characters, Drama, and World. This is "Action Library headquarters" — the place where you build and organize the *master copies* of every action in your game, before you ever attach one to anything.

### 2.2 Section 1 — Stakes Library

This is the authoring screen for the Stakes described above. Each Stake you create gets its own little table: three rows (Strong Hit / Partial / Failure) and two columns (Player / Target), where each cell lets you pick a resource and a signed amount. You can also mark a Stake as the **default pick** for one or more of the Four Fs, so when a player rolls using that attribute during freeform play, this Stake is pre-selected for them. Use **+ Add Stake** to create a new one.

### 2.3 Section 2 — Global Action Library

This is the master list of every **Action Template** you've built (a Template is explained fully in Part 3 below — for now, think of it as "the recipe"). For each template in the list you'll see its name, its tag(s), and a quick count of how many conditions and effects it has. From here you can:

- Click **+ New Action** to create a brand-new template and immediately open it in the Action Studio.
- Use the search box and tag filter chips to find a template once your library grows.
- Click the **duplicate** icon to clone an existing template (handy for making three similar variants of an attack, for instance).
- Click the **trash** icon to delete a template — except for a handful of **system templates** (Default Travel, Return Travel, and Default Resolution) whose trash icon is hidden, since the engine relies on them existing. You can still freely *edit* them, just not delete them.

A few built-in templates ship with every project and live in this same list: **Pick Up**, **Give**, **Drop**, and **Purchase** are pre-built player actions covering basic inventory interactions, and the travel/resolution system templates mentioned above. Treat them as examples worth opening and reading.

---

## Part 3 — Instance vs. Template (the single most important concept)

This is the idea that trips people up the most, so let's be very explicit about it.

- A **Template** is the master recipe, sitting in the **Action Library** (Section 2 of the Actions tab). It isn't attached to anything by itself — nobody in your world can actually *use* it yet. It's just a definition: "here's how a Lockpick attempt works, here's what it requires, here's what happens on success and failure."
- An **Instance** is an actual, usable copy of an action that's attached to one specific thing in your world — a particular door, a particular NPC, a particular sword. When the player is standing in front of that door, the door's instance is what shows up as a clickable action.

In other words: a Template is a blueprint; an Instance is the building. You can have one Template ("Pick Lock") and twenty different doors each holding their own Instance copied from it.

### 3.1 The three ways an action ends up on an entity

Open any entity (a location, item, character, or narrative object) in its editor and scroll to the **🎬 Action Studio** section. You have three options:

1. **Copy from the Library.** Click **📚 + Action Library**, pick a template, and the engine makes an independent copy and attaches it to this entity. From this point on, editing the original template in the Action Library does *not* change this copy — they've split apart. The copy quietly remembers which template it came from (so you can tell where it originated), but it's otherwise its own thing. This is the right choice when you want to customize one specific door's lock difficulty without affecting every other lock in the game.

2. **Build a brand-new one-off action directly on the entity.** There's no library template involved at all — you're authoring something unique to this one entity from scratch. Use this for truly one-of-a-kind moments that will never need to be reused anywhere else.

3. **Tag Subscriptions — the "live link" option.** Every entity has a **🏷 Tag Subscriptions** section listing every tag used anywhere in your Action Library. Check a tag's box, and *every* template carrying that tag is now resolved live on this entity — meaning no copy is ever made. If you later edit that template in the Library (change its dice, fix a typo, add an effect), the change instantly applies to every entity subscribed to that tag, with zero extra work. This is the right choice for things that should behave identically across many entities at once — every "campfire" in your world having the same Rest action, for example, or every member of the Town Guard faction sharing the same Arrest action. If you've also manually copied a template that's now covered by a tag subscription, the live subscribed version takes priority so you don't end up with the same action listed twice.

**Rule of thumb:** if many entities should always behave identically and you want one editing point, use Tag Subscriptions. If one entity needs its own customized variant, copy it from the Library. If it's truly unique and will never be reused, build it directly on the entity.

A template can also restrict which kinds of entities it's even allowed to attach to, using its **Applies To** entity-type filter (locations only, items only, etc.) — leaving it blank means "anything."

---

## Part 4 — The Action Studio, phase by phase

Click any template (or any entity's instance) to open the **Action Studio**, the builder where you actually define how an action behaves. It walks through four numbered phases plus a row of flags at the bottom.

### Phase 1 — Requirements (Conditions)

This box, labeled **1️⃣ Requirements (Base Conditions)**, is checked *before* the action is allowed to run. If even one condition fails, the action is either hidden from the player entirely or blocked outright, depending on context. Think of conditions as the bouncer at the door — full reference table is in Part 5 below.

### Phase 2 — Resolution & Outcomes (the dice engine for this action)

This is where you decide *whether* this particular action involves any randomness at all, and if so, how. Click the **Engine** button to choose from:

- **Auto-Success (No Roll)** — the action always works; there's nothing to resolve. This is the default for most utility actions (picking something up, opening an unlocked door).
- **D20 vs DC** — a classic Dungeons & Dragons–style check: roll a twenty-sided die, add a skill modifier, and compare the total against a target Difficulty Class (DC) you set.
- **2d6 Fail Forward (PbtA)** — the built-in default system described in Part 1.7, with its three outcome bands.
- **Custom 1d6 vs DC** — a lighter single-die check against a DC you set.
- **Custom Dice (NdN+M)** — type in any dice notation you like (e.g. `2d8+3`) for full control.

Once a roll type is picked, you'll see additional fields:

- **Dice** — overrides the engine's default notation if you want something nonstandard.
- **Check vs DC** (a checkbox) plus a number field for the DC itself, for the D20/1d6/custom-dice types.
- **Skill mod** — which character-sheet field (which of the Four Fs, or a custom skill) gets added to the roll.
- **Crit** — enables natural-maximum / natural-1 critical results on single-die DC checks.
- **Outcome Bands** — for systems like 2d6 Fail Forward that use bands instead of a flat DC, this is where you define the ranges (minimum/maximum roll total), their label, an optional GM note (a short stage-direction the AI receives, never shown directly to the player), and which effects fire when the roll lands in that band. Bands are checked in order and the *first* one whose range contains the roll wins, so put narrow/specific bands above broad catch-all ones.

If a resolution uses bands, those bands' effects are what actually fire — the plain Success/Failure effects in Phase 3 below are skipped in favor of whichever band matched. If there are no bands (a flat DC or auto-success), the action falls through to the ordinary Success/Failure split.

### Phase 3 — Effects

This is the consequences phase — what actually happens to the world once the action resolves. There are three buckets:

- **🟩 On Success** (or, for a no-roll action, simply **Effects to Apply**) — fires when the roll succeeds, or always for an auto-success action.
- **🟥 On Failure** — fires only when the roll fails. Only shown for actions that involve a roll.
- **⬜ Always (regardless of roll)** — fires no matter what the outcome was. This is the right place for things that should happen either way, like spending a resource cost, a global world consequence, or clearing a temporary status — it always runs *after* the success/failure branch.

The full list of available effect types, with plain-language explanations, is in Part 6 below.

### Phase 4 — Branches (alternative outcomes)

Sometimes a single action needs to behave completely differently depending on context — a "Talk to Guard" action that plays out one way if you're disguised and another way entirely if you're not. **Branches** let you define alternative versions of Phases 1–3 that override the base action when their own conditions are met.

Each branch has its own Conditions, its own Resolution, and its own Effects — a fully self-contained alternate path. Branches are checked top to bottom, and the **first branch whose conditions all pass wins**, completely replacing the base resolution and outcomes for this run. If no branch matches, the action falls back to its base (non-branch) behavior. Use **+ Add Branch** to create one, and reorder/delete as needed.

### Flags

Below the four phases sits a row of toggles that change how the action behaves as a whole:

- **Silent** — suppresses the toast notification normally shown when this action fires.
- **Hidden** — the action exists and can still be triggered programmatically (e.g. by a chain), but it won't appear in the player's visible list of choices.
- **One-shot** — the action can only fire once per entity, ever (tracked per-entity so the same template reused elsewhere still works normally). Once used, a small "✓ used" marker appears, and you can manually **Reset** it from the entity's Action Studio panel if you need to test it again.
- **Auto** — see Part 9 below; turns this into an action that evaluates and fires itself automatically rather than waiting for the player to click it.

### Tags

A free-text, comma-separated field for organizing your library (e.g. `combat, magic`) and — critically — for powering **Tag Subscriptions** (Part 3.1 above). The same tag field doubles as both an organizational label and a live-binding mechanism, so name your tags thoughtfully.

### Targeting

Toggle **Needs a target (player picks)** on for any action that should act on something the player chooses at the moment they take it — attacking a specific enemy, talking to a specific NPC in the room. When this is on, you can also restrict which *types* of entity are offered as valid targets. With targeting off, the action only ever has one implicit subject (whatever entity it's attached to).

---

## Part 5 — Conditions Reference

Every condition below can appear in a template's base Requirements, in a branch's own Requirements, or as a precondition gating an Auto action. Most conditions take a **scope** ("Applies to") — explained fully in Part 7 — to decide *which* entity or character the check is actually about.

| Condition | What it checks |
|---|---|
| **Requires Resource (Party)** | The party must currently hold at least a minimum amount of a chosen global resource. |
| **Entity State Is** | The target entity's current default/atmosphere state must match a specific named state. |
| **Story State Is** | A narrative gate on the entity must be in a specific state: Locked, Active, or Completed. |
| **Gate Is** | Whether the entity's travel/interaction gate is currently locked or unlocked. |
| **Located In** | The target entity's parent (its containing location) must be a specific place. |
| **Is Equipped** | Whether an item entity is currently equipped (to anyone). |
| **Player Is In Vehicle** | Passes only when the protagonist is currently inside a vehicle entity — handy for gating ship/vehicle-only travel routes. |
| **Character In Scene** | Whether a chosen character is currently present in the active scene. |
| **Character Alive** | Whether a chosen character is active (hasn't been killed/removed). |
| **Character Faction Standing** | A specific character's personal standing with a faction must match a chosen state. |
| **Character Resource** | Compares a character's personal resource — enter a number for a pool, or a state name (like "Dead") for a track/condition resource. |
| **Party Faction Standing** | The whole party's standing with a faction must match a chosen state. |
| **Relationship Archetype Is** | A character's current relationship archetype must match a chosen one. |
| **Relationship Stage** | Compares a character's relationship stage index against a number (0 = earliest stage). |
| **Raw Field Check** | An advanced escape hatch: a named field on any record must equal a specific value, for situations not covered by the options above. |

---

## Part 6 — Effects Reference

Effects are the building blocks of Phase 3 (Success/Failure/Always) and of every Outcome Band. Most also take a scope to decide who or what they apply to (Part 7).

| Effect | What it does |
|---|---|
| **Move Player Here** | Commits the protagonist's movement to this entity. Always place it *after* any Inject Story Text effect in the same list — it handles the body move, scene presence, any auto-triggers at the destination, and generates the next bit of story. |
| **Inject Story Text** | Writes a line directly into the visible story document. Supports `{Actor}`/`{Target}` placeholders that get swapped for the relevant names. |
| **Inject GM Message** | A hidden instruction sent only to the AI for its next generation — never written into the visible story. Useful for steering tone without the player seeing a stage direction. |
| **Pop-up Message** | Shows a small message panel directly to the player. Never sent to the AI at all — purely a player-facing note. |
| **Add Narrative Hook** | Plants a "pressurized" story tension on a character (an unresolved thread the AI is nudged to develop over time). Surfaces in that character's hooks panel and in future context. |
| **Add Skill Modifier** | Grants a temporary bonus or penalty to a character's skill, which automatically reverts if the player hits Undo. Pair it with Remove Skill Modifier to clear it manually later (e.g. when a buff's duration ends). |
| **Remove Skill Modifier** | Clears all active modifiers for a given character + skill combination. |
| **Modify Global Resource** | Adds, subtracts, or sets a party-wide resource. For a track resource, the "amount" is the target state index rather than a raw delta. |
| **Set Entity State** | Switches an entity's current default/atmosphere state to a different one. |
| **Set Story State** | Locks, activates, or completes a narrative gate on an entity. |
| **Set Gate** | Locks or unlocks an entity's travel/interaction gate. |
| **Move Entity** | Relocates an entity to a different parent location — physically moving an item, character, or even a sub-location elsewhere in the world graph. |
| **Equip / Unequip Item** | Equips or unequips an item entity onto a chosen character. |
| **Set Scene Presence** | Brings a character into, or removes them from, the currently active scene. |
| **Set Character Active** | Marks a character Active or Inactive. Active characters appear in the index, the dramatis personae, and receive ongoing automatic story-development passes. |
| **Set Character Sheet Field** | Writes a value directly into one of a character's sheet fields. Supports `{Actor}`/`{Target}` in the value. |
| **Kill Character** | Deactivates a character and drops their inventory in their current room. |
| **Set Character Faction** | Directly sets a character's personal standing with a faction. |
| **Modify Character Resource** | Adds, subtracts, or sets a personal resource on the targeted character — for the protagonist's own stats, set the scope to "The Player." Higher is generally better (a Health track's higher state means healthier). |
| **Set Party Faction** | Directly sets the whole party's standing with a faction. |
| **Modify Party Faction Standing** | Nudges the party's standing with a faction up or down by a number of steps (positive = friendlier, negative = more hostile), clamped to the faction's defined states. |
| **Set Relationship Archetype** | Switches a character's relationship archetype, with an option to keep their current stage rather than resetting it to 0. |
| **Modify Relationship Stage** | Advances or retreats a character's relationship stage — positive moves toward the warmest stage, negative toward the coldest, always clamped within the archetype's defined stages. |
| **Chain Action** | Runs an entirely separate action from inside this one. Covered fully in Part 8. |
| **Ask "How do you do it?"** | Pauses and shows the player a free-text field, letting them describe their own approach in their own words. Their answer is woven into the *next* Inject Story Text effect, so place this effect *before* that one in the list. |

---

## Part 7 — Scopes & tokens: who does an effect actually apply to?

Many conditions and effects need to know *which* entity or character they're talking about. That's the **Applies To / scope** picker, and it always offers the same four logical options, plus the ability to point at any specific entity by name:

- **This entity (self)** — the entity the action itself is attached to (the door, the item, the NPC).
- **The player** — the protagonist, always, no matter who the action is attached to.
- **Chosen target** — whatever entity the player picked, *only* available if the action's Targeting flag (Part 4) is turned on.
- **Parent location** — the containing location of the source entity (useful for "everyone in this room" effects).
- **A specific named entity** — pick any concrete location, item, character, or narrative object directly from the list.

Inside any text field (Inject Story Text, Pop-up body, hook descriptions, and so on) you can also drop in two simple placeholders that get swapped automatically when the action runs: **`{Actor}`** becomes the name of whoever is performing the action (usually the protagonist), and **`{Target}`** becomes the name of whoever or whatever the action is aimed at.

---

## Part 8 — Chains: running one action from inside another

The **Chain Action** effect lets one action trigger a completely separate action as part of its own outcome. When you add it, you choose:

1. **From** — either the **Action Library** itself (any template), or a *specific entity* in your world (so you can chain into one of that entity's own unique instances).
2. **Action** — which specific action from that source to run.

A chained action automatically inherits the same actor and target as the action that triggered it, and it **always runs silently** — it never pops its own toast or starts its own story generation, so it won't feel like a separate event to the player. To prevent an accidental infinite loop (Action A chains into B, which chains back into A, forever), the engine enforces a hard **maximum chain depth of 5** — if a chain tries to go six levels deep, it simply stops and logs a warning instead of crashing anything.

**When would you use this?** Anytime one action's consequences should logically include "and also do this other, already-defined thing" — a "Sound the Alarm" effect chaining into a separately-defined "Guards Become Hostile" action on every guard in the building, for instance, rather than duplicating that logic by hand on every alarm-triggering action in your world.

---

## Part 9 — Auto & Global actions: actions that fire themselves

Normally, an action sits and waits for the player to click it. Turning on the **Auto** flag changes that completely.

### How Auto works

An Auto action is **evaluated automatically after every single story generation** (not by the player clicking anything). The engine checks its Conditions, and if they pass, the action fires — entirely silently, applying its effects (and, if it has an `outcomeNote`, queueing a short note for the AI's next generation) without ever interrupting the flow with its own story generation. Because this runs every turn, **you should always gate an Auto action behind meaningful Conditions** — otherwise it will fire over and over, every single turn, forever. (One-shot, described in Part 4, is the natural partner here: combine Auto + One-shot for "the first time conditions X are true, do Y, then never again.")

### Which entities get checked

By default, the engine only evaluates Auto actions on entities that are *currently relevant to the scene*: the player's current location, anything directly inside that location, and anything connected to it. An Auto action sitting on an NPC in a town you left three areas ago simply won't be checked — it's out of scope.

### The Global sub-flag

Once Auto is turned on, a second checkbox appears: **⚙ Global (fire every turn, any location)**. Turning this on lifts the location restriction entirely — this entity's Auto actions get checked *every single turn, no matter where the player currently is in the world*. Use this sparingly and only for things that are genuinely world-spanning: a ticking doomsday clock, a weather system, a background political event that should keep progressing even while the player is somewhere else entirely. For anything scoped to "this room" or "this NPC," leave Global off so the engine isn't wasting effort checking things far outside the current scene.

---

## Part 10 — Two worked examples

**Example A — A locked door that needs a key.**
Create a template called "Open Iron Door." In Phase 1, add a *Requires Resource (Party)* condition checking for at least 1 of `#iron_key`. Leave Resolution as Auto-Success (no roll needed — either you have the key or you don't). In Phase 3's Success effects, add *Inject Story Text* ("The lock gives way with a heavy click."), followed by *Move Player Here*. In the Always bucket, add *Modify Global Resource* subtracting 1 `#iron_key` if you want the key consumed on use (or skip this if the key should be reusable). Attach it to the door entity by copying it from the Library.

**Example B — Persuading a guard, with branching based on disguise.**
Create a template called "Talk to Guard." Base Conditions: none (always available while the guard is present). Base Resolution: 2d6 Fail Forward, skill mod set to Flair. Base Effects: on Strong Hit, inject narrative describing the guard waving you through; on Failure, inject narrative describing suspicion rising and chain into a "Guard Becomes Hostile" action. Then add a **Branch** named "Disguised," with its own Condition checking the player's personal `#disguised` resource equals "True," and its own Resolution set to Auto-Success — since wearing a convincing disguise should just work, no roll needed. The branch's own Effects inject a line about walking past without a second glance. Because branches are checked first and the Disguised branch's condition only passes when the player is actually disguised, the dice-based base behavior only kicks in the rest of the time.

---

## Quick-Reference Cheat Sheet

**Resource shapes:** Pool (free number) · Track (named ladder, e.g. Health states) · Condition (status-flag ladder, index 0 = nothing active)

**Where resources live:** Global = shared by the whole party · Character = personal to one character

**Action Library location:** Sidebar → 🎬 Action tab → Section 2

**Template vs. Instance:** Template = master recipe in the Library, not usable by itself · Instance = a working copy attached to one specific entity

**Three ways to attach an action:** Copy from Library (independent copy) · Build directly on the entity (fully unique) · Tag Subscription (live, no copy — edits to the template propagate instantly)

**Action Studio phases:** 1) Conditions (gate) → 2) Resolution (the roll, if any) → 3) Effects: Success / Failure / Always → 4) Branches (alternate versions that override 1–3 when their own conditions match, first match wins)

**Flags:** Silent (no toast) · Hidden (not player-visible, still chainable) · One-shot (fires once ever, resettable) · Auto (fires itself every turn if conditions pass) · Global (Auto + ignores location scope)

**Scopes:** Self · Player · Chosen target (needs Targeting on) · Parent location · A specific named entity

**Chains:** One action triggers another; inherits actor/target; always silent; capped at 5 levels deep

**2d6 bands:** 2–6 Failure · 7–9 Partial Success (cost attached) · 10+ Strong Hit (clean win)
