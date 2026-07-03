# Adventure Engine — Characters: The Complete Guide

*Covers Adventure Engine v1.6.2 / AE Bridge HUD v0.9.9.9*

This is the third guide in the series. The Resources & Actions guide covered the rules a character lives under; the Scene Presence & Context guide covered how the engine decides what the AI sees about them. This one is about the character themselves — how they're built, how you inspect and steer them at the table, and the handful of mechanics (following, targeting, scheduling, minor NPCs) that only make sense once you understand the record underneath.

---

## Part 1 — What a character actually is

Every character is really two linked things:

- **A `CharacterRecord`** — the data: identity fields, the four narrative fields, attributes, resources, relationship, schedule, hooks. This is what the Characters tab and the edit modal work with.
- **A body `EntityRecord`** — the physical presence in the world tree, whose `parentId` is wherever they currently are. Scene presence, travel, and Follow Me all operate on this body, not on the character record directly (see the Scene Presence guide, Part 2).

One character is flagged `isProtagonist` — the player. Everyone else is an NPC. A character can be **Active** or **Inactive**; only Active characters appear in the index, the Dramatis Personae list, and receive automatic dynamics updates (goal/thought/relationship regeneration). Deactivating a character (or the `kill_char` effect) is how you retire someone without deleting their history.

---

## Part 2 — Creating a character

There are three on-ramps, scaled to how much the character matters.

### 2.1 The World-Build Wizard (batch, from your worldbuilding chats)

In the Characters tab of the wizard, you can generate a batch of full characters straight from your brainstorming chats, the same mechanism used for Species and Factions. You pick which chat(s) to draw context from, generate, and review each one in the gallery before committing — nothing is written to the lorebook until you accept it. This is the fast path for filling out a cast at world-build time.

### 2.2 Manual creation, one field group at a time

From the Characters tab (wizard or in-play HUD), **+ Add Character** opens a blank record and the full edit modal. You can type everything by hand, or lean on the **regen** button next to each narrative field (see 3.2) to generate just that field from a concept dump — useful when you know roughly who someone is but don't want to write four paragraphs yourself.

### 2.3 Quick drop-in NPCs

For someone who doesn't warrant a full dossier — a barkeep who has one line — use **Add Minor NPC** instead of the full character flow. It asks for only a name, a one-line "what are they doing here," and (optionally) a Bestiary archetype to seed from. No lorebook entry, no narrative fields, no resources. It spawns directly at the player's current location as a child entity, the same shape as an item. See Part 8 for the full minor-NPC lifecycle.

---

## Part 3 — The full edit modal, section by section

The character edit modal is opened from the Characters gallery, from a scene roster, or via `AE_OPEN_CHAR_EDIT` from the bridge. It's organized as collapsible sections; everything auto-saves as you type or toggle.

### 3.1 Identity

The one-glance fields: **Species, Age, Gender, Role**, plus a **Brief** (one tight sentence identifying who they are at a glance) and **Nicknames**. These feed the identity line that heads their lorebook entry — `Gareth — human male, 34, blacksmith` — and the Dramatis Personae cast list. Each has its own regen button.

### 3.2 Narrative Profile — the four static fields

This is the stable dossier described in the Scene Presence guide (Part 4): **Physical Description, Personality Core, Voice & Mannerisms, Background**, plus a semicolon-separated **Core Values** list. Each field:

- Has a **per-field regen button** that rewrites just that field from the concept profile (Identity + whatever you've already written), so you can tune one facet — say, Voice — without disturbing the rest.
- Carries **default authoring instructions** baked into the engine (e.g. Physical Description is explicitly told to skip clothing and mannerisms, since those live elsewhere) — you can override these per-field in World Settings if you want a different house style.
- Has a **budget** (a soft token-length target) so the lorebook entry stays a dossier, not an essay.

Editing any of these re-syncs the linked lorebook entry the next time it's read; the engine doesn't do it on every keystroke, only when you actually save.

### 3.3 Attributes — the Four Fs

If the character uses the default rule system, an Attributes section rates them across **Force, Finesse, Focus, Flair** (see the Resources & Actions guide, 1.5) from -1 (weak) to +3 (expert), 0 being untrained. These are the modifiers that ride along on any 2d6 roll that leans on that attribute.

### 3.4 Resources

Character Resources — health, stamina, sanity, personal coin — live here as their own pools/tracks/conditions, independent per character. New characters are auto-seeded from the **Character Resources** template in World Settings; if you add a new template resource later, a **Sync** option pushes it out to existing characters that predate the change without touching what they already have.

### 3.5 Relationship

Each character can carry a **Relationship Archetype** (Rival, Mentor, Love Interest, etc.) and a **Stage** index into that archetype's own stage ladder. This is what feeds the relationship line in the Author's Note dynamics block (Scene Presence guide, 3.3) and what `add_modifier`/action effects like `set_char_sheet` or dedicated relationship effects can move.

### 3.6 Daily Schedule

An hourly timetable: a set of rows, each an hour-of-day plus a target location. See Part 5 for how this resolves during play.

### 3.7 Custom sheet fields

If you've defined custom static fields or sheet categories in World Settings, they appear here too, and are injected into the lorebook entry the same way the four core fields are (labelled, in the order you defined them).

---

## Part 4 — Two inspection surfaces, same data

You can look at a character in two different UI surfaces, and it's worth knowing which is which.

**The full edit modal** (Part 3) is the authoring view — every field is editable, nothing is live-only. Open it any time, including outside of play.

**The character inspection / focus view** — available both as the bridge HUD's Character tab and as the native in-worker HUD panel — is the *in-play* view. It's read-mostly with a handful of live action buttons, built from a `buildEntityDetailPayload` snapshot rather than the raw record, and includes things the edit modal doesn't:

- A **Snapshot** section — an on-demand generated image-prompt/description of the character as they currently are (equipment, state, current scene), assembled the same way as `assembleSnapshotImage`. Collapsed by default; generating one doesn't touch the stable dossier.
- The **Follow Me / Stay here** toggle (non-protagonist characters only — see Part 6).
- **Active Hooks** — narrative tensions currently pressuring this character (from `add_hook` effects), shown as cards you can resolve.
- **Inventory** — carried items, each with its own entity actions.
- **Narratives** — plot threads attached to this character.

Both the bridge and native versions call the *exact same* underlying functions (`toggleCharFollow`, `resolveCharHook`, etc.) rather than keeping separate copies, so whichever surface you use, the state is identical and there's no divergence to worry about.

---

## Part 5 — Schedules

A character with **Schedule enabled** and at least one schedule row is walked to wherever their timetable says they belong whenever `resolveSchedules()` runs — which is *only* at travel commits and manual clock changes, never on every generation, so it costs nothing in the common case.

The resolution rule: sort the rows by start hour, and pick whichever row's start hour is the latest one at or before the current hour (wrapping to the last row if the current hour is before all of them — "yesterday's last slot carries over"). Then that row's location becomes the character's body's new parent, exactly like a `set_entity_parent` effect.

Two safeguards keep this from feeling mechanical:

- **No vanishing while watched.** A scheduled character who is currently co-present with the player is left alone regardless of what the clock says.
- **Arrivals you can wait for.** A schedule *can* walk someone into the player's room — so a player who knows the guard captain returns at dusk can simply wait, and the captain walks in on schedule.

### The All-Schedules modal

Rather than opening each character's card individually, the **All-Schedules** modal lists every character's timetable in one place, read *and* write. It's not a separate copy of the data — it calls the exact same load/save/picker functions as each character's own Daily Schedule section, so editing from either surface updates the same underlying rows.

---

## Part 6 — Follow Me / Stay Here

Any non-protagonist character can be toggled to **follow** the player. Mechanically this is nothing more than a parent-swap on the character's body entity:

- **Not following:** the character's body sits wherever it normally would (a room, or wherever their schedule/story left them).
- **Following:** the character's body's parent is set directly to the *protagonist's* body entity, rather than to a room.

The toggle (`toggleCharFollow`) reads the current state by checking whether the character's body's `parentId` already equals the protagonist's body id:

- If **not** following → parent becomes the protagonist's entity id (they now travel wherever the player travels, since `getEffectiveRoomId` walks up through the player to find the room).
- If **already** following → parent is set back to the protagonist's *current room* (they stay put, released into the room the player is standing in right now, not teleported elsewhere).

Every travel commit recomputes scene presence afterward (Scene Presence guide, 6.1), so a following character is automatically present in every new room the player enters — no separate "keep them present" logic needed; it falls straight out of the parent-chain walk.

The toggle is available from the bridge HUD's follow button, the native inspection view's Follow Me button, and the `AE_TOGGLE_FOLLOW` bridge message — all three call the identical shared function.

---

## Part 7 — Targeting

"Targeting" spans two related but distinct things: how an **action** picks who it acts on, and how an **effect** refers to that choice once made.

### 7.1 `targetMode` — does this action need a target at all?

An action (or entity action) can declare `targetMode: 'player_pick'`, which means: before running, prompt the player to choose a target from what's actually available in the scene. Leaving it unset (or `'fixed'`) means the action runs without a runtime target picker — it either doesn't need one, or its target is implied by context (e.g. the entity it's attached to).

When `player_pick` is on, you can optionally narrow the picker with `targetFilter` — a list like `['character']` or `['item', 'location']` — so a "Give" action only offers items, while an "Attack" action only offers characters. An empty filter means anything in the scene is fair game. Targets are always scoped to the player's current room; you can't target something three rooms away.

The Quick Action composer's target dropdown is where players actually see this: it lists `[No target]`, `[Self]`, the current location, every present character, every present item, and — as of a recent update — every present **minor NPC** too, since minor NPCs are entity bodies just like anyone else and drop straight into the same list.

### 7.2 Scope — how effects refer to the target afterward

Once a target is chosen (or an action has an implicit actor/target pair), every effect and condition refers to *who it applies to* through a small fixed vocabulary of scopes, not a raw ID:

| Scope | Means |
|---|---|
| `__self__` | The entity the action/rule is attached to |
| `__player__` | The protagonist, always |
| `__target__` | Whoever was picked (by `player_pick`) or passed in as the action's target |
| `__parent__` | The scope entity's current parent location |
| `__current_room__` | The player's current room |
| `__ask__` | Prompt the player at runtime for a specific value (e.g. "Give" asks who to give the item to) |
| a concrete entity id | That exact entity, always — used when an effect should hit a specific named location or character regardless of targeting |

This is why the same library action template (Pick Up, Give, Purchase, etc.) works on any entity you attach it to: the template only ever talks about `__self__`/`__target__`/`__player__`, and those resolve differently depending on what actually triggered the action. `{Actor}` and `{Target}` placeholders in injected text and messages resolve from this same pair.

---

## Part 8 — Bestiary & minor NPCs

Not every person or creature in your world needs a permanent dossier. The engine splits the cast into **major characters** (full `CharacterRecord`s, Part 1–3) and **minor NPCs** (throwaway entity bodies with a name and a description, no lorebook, no narrative fields, no resources).

### 8.1 The Bestiary — archetypes, not individuals

The **Bestiary** (a tab in the Characters section) holds **archetypes**: reusable spawn templates like "Tavern Drunk" or "City Guard," each with a name, description, and optionally its own entity actions and `{a|b|c}` text alternations so multiple spawns from the same archetype don't all read identically. You can generate a batch of archetypes straight from your worldbuilding chats — the wizard feeds in existing archetypes so it doesn't generate duplicates — or add them one at a time.

### 8.2 Spawning

Two ways to get a minor NPC into the scene:

- **Manually**, via Add Minor NPC (2.3), optionally seeded from a Bestiary archetype.
- **From an action effect**, via `spawn_minor_npc` — pick an archetype and a count (1–10), and that many instances are created at the player's current location when the effect fires. This is the mechanism behind random encounters and crowd scenes: put different spawn counts on different resolution bands (a Strong Hit clears the alley, a Failure spawns three thugs) for encounter variety.
- **Scene-grounded generation** — `generateSceneNpc` invents a single background character on demand, using the live story text and current location as context so they actually fit the moment, rather than being picked from a fixed template. This returns a name/brief for review; it doesn't spawn anything by itself until you accept it.

### 8.3 In-scene: what a minor NPC gets

A minor NPC's entity-detail rendering is deliberately thin: its description, any entity actions it carries, and two buttons — **Edit Entity** (opens the plain entity editor, not the full character modal) and **Promote to Major**.

### 8.4 Promotion

If a background figure turns out to matter, **Promote to Major** (`promoteMinorNpc`) upgrades their entity into a full `CharacterRecord` — at which point they gain everything Part 3 describes: narrative fields, attributes, resources, a lorebook entry, schedule, the works. This is the one-way door between "background figure" and "character," and it's designed to be taken lazily — spawn cheap, promote only the ones the story actually leans on.

---

## Part 9 — What else is big enough to mention

A few mechanics that touch characters heavily but are covered properly elsewhere — flagged here so nothing falls through the cracks:

- **Hooks** (`add_hook`) — a pressurized narrative tension planted on a character (a pressure value 0–100, a description, an optional target), surfaced in both inspection views and fed into context. Resolving one is a single click from either HUD.
- **Dynamics** (arc/relationship, goal, thought) — the three-speed psychology layer that drives the Author's Note's per-NPC line. Fully covered in the Scene Presence guide, Part 3.3; nothing about it lives in the character record itself beyond the Relationship fields in 3.5 — the goal and thought layers are regenerated on their own cadence, not hand-edited.
- **Knowledge** (per-character learned facts about entities) — covered in the Scene Presence guide, Part 7. Not exposed in the edit modal; it accrues automatically as characters become aware of things.
- **Equipment** — deliberately *not* part of the stable dossier (Scene Presence guide, 4.3); managed through `set_equipped` and the Inventory section of the inspection view, not the edit modal.

---

## Quick-Reference Cheat Sheet

**A character = a `CharacterRecord`** (identity, narrative fields, attributes, resources, relationship, schedule, hooks) **+ a body `EntityRecord`** (physical position). Presence, travel, and Follow Me act on the body.

**Creation:** Wizard batch-gen from chats (fast cast-fill) · manual + per-field regen (one at a time) · Add Minor NPC (throwaway, no dossier).

**Edit modal sections:** Identity → Narrative Profile (4 fields, each regen-able) → Attributes (Four Fs) → Resources → Relationship (archetype + stage) → Daily Schedule → custom sheet fields.

**Two inspection surfaces, same functions underneath:** edit modal (authoring, everything editable) vs. focus/inspection view (in-play, snapshot + Follow Me + hooks + inventory + narratives).

**Schedules:** resolved only at travel/time-change checkpoints, never per-turn. Never vanishes someone currently in the room; can walk someone in so the player can wait for them. All-Schedules modal reads/writes the same rows as each character's own card.

**Follow Me:** a body parent-swap — following = parented to the protagonist's entity; not following = parented to a room. Presence then falls out automatically from the normal co-location recompute on travel.

**Targeting:** `targetMode: 'player_pick'` (+ optional `targetFilter`) decides whether/what a player picks at runtime; the fixed scope vocabulary (`__self__ / __player__ / __target__ / __parent__ / __current_room__ / __ask__` or a concrete id) is how every effect refers to the result afterward.

**Bestiary:** archetypes = reusable spawn templates, not individuals. Spawn via manual add, `spawn_minor_npc` effect (band-variable counts for encounter variety), or scene-grounded on-demand generation. Minor NPCs are cheap and disposable; **Promote to Major** is the one-way upgrade to a full character.

**Elsewhere in the series:** Dynamics (arc/goal/thought) and Knowledge → Scene Presence & Context guide. Resources, Stakes, and the Four Fs → Resources & Actions guide.
