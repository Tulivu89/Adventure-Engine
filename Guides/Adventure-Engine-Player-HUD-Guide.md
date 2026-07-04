# Adventure Engine — Player's Guide: Your HUD

*Covers Adventure Engine v1.7.0 / AE Bridge HUD v0.9.9.16*

The other guides in this series are written for the person building your world. This one is written for **you** — the player. It's a tour of the HUD: what's on it, what each button does, and how to use it while you play. Nothing here requires knowing how the engine works under the hood.

---

## Part 1 — Two HUDs, one game

Depending on how your game is set up, you'll be playing with one of two interfaces (sometimes both at once):

- **The Bridge HUD** — a floating panel that lives on top of your story. Look for a small **🗺 AE** button, usually in a corner of the screen. It's draggable — grab it and drop it wherever's out of your way. Tap it to open the full panel.
- **The Native HUD** — a row of buttons built into NovelAI's own toolbar: **Map, Inspect, Action, Player, CYOA, Calendar**. Each one opens its own popup window. No extension required.

Both talk to the exact same character sheets, inventory, and story state — they're just two different windows into it. Everything in this guide exists in both places, sometimes laid out a little differently. Where it matters, this guide calls out both.

### The two side trays (Bridge only)

Next to the **🗺 AE** button are two small arrows (chevrons), one on each side:

- **Right arrow** opens a tray of four quick-look buttons: **World, Inspect, Action, Local**. Tapping one pops up a floating widget right over your current spot in the story — handy for a quick check without losing your place. These are shortcuts to the Map, Scene, Actions, and Player tabs described below.
- **Left arrow** opens the **CYOA tray** — covered in Part 6.

---

## Part 2 — The Story tab: getting the HUD out of your way

Tapping the book icon (Bridge) collapses the panel down to just the toggle button, so you can read and write without it taking up screen space. It doesn't close anything or lose your place — it just tucks itself away until you tap it open again.

---

## Part 3 — The Map tab ("World")

Shows a small graph centered on wherever you currently are: the room above you, the rooms/areas below or connected to it, and (if you're in one) the vehicle you're riding in. Tap any node to select it, then use:

- **Inspect** — look at it without going anywhere.
- **Travel Here** — move there. The story continues from your arrival.
- **Teleport** — move there silently, with no story text generated. Useful for backtracking or fixing a mistake without narrating it.

If you're inside a vehicle (a ship, a car, anything that itself travels), an **Operate** button appears — switch it on and the map shows the vehicle's own routes through the wider world instead of the room you're standing in.

---

## Part 4 — The Scene tab ("Inspect"): who and what's around you

This is your at-a-glance roster of the current location.

- **Characters** — a grid of portraits. The **👁 Scene / 🌐 All** toggle switches between "just who's actually here right now" and "everyone you've ever met." Present characters are highlighted; tap anyone to open their detail view.
- **Interactive**, **Present NPCs**, **Items**, and **Local Narratives** — everything else physically here: things you can act on, background people, objects, and any quest thread tied to this specific place.

### Tapping into someone or something

Tap any character, item, or narrative to open its detail view:

- **Characters** get a **Snapshot** section (generate a written description or an image of them exactly as they are right now — this doesn't touch their permanent profile, it's just a look at the moment), a **Follow me / Stay here** toggle for anyone traveling with you, their **Active Hooks** (unresolved tensions involving them), their **Inventory**, and any **Narratives** attached to them. A wrench icon opens their full editable profile.
- **Minor NPCs** (background characters without a full profile) get a much simpler view — just a description and a **Promote to Major** button if they turn out to matter more than expected.
- **Items** get an **Equip/Equipped** toggle plus whatever entity-specific actions they carry.
- **Narratives** (quests/threads) show their state — 🔒 Locked, 🔥 Active, ✅ Done — and a **Track Quest** button. Tracking one pins it to the top of your Quests list (Part 7) and, if it has a destination, shows you the route there.

---

## Part 5 — The Actions tab ("Action"): doing things

This is where you actually act. At the top is the **Goal & Approach** composer:

1. **Write what you're doing**, in your own words, in the text box. This is the freeform "I try to pick the lock" / "I try to talk him down" box — it tells the story (and any dice roll) what you're attempting.
2. Optionally pick an **attribute** to roll against (or "No check" if this doesn't need one), a **stake** (what's actually on the line — see the Resources & Actions guide if you want the full mechanics), and a **target** (yourself, the room, or anyone/anything present).
3. If you have a **Luck** resource with points banked, a stepper appears letting you spend some for a bonus on the roll.
4. **Execute** runs it now. **Save** stores your current setup as a reusable preset underneath, so a favorite move doesn't need re-typing every time.

The little **🔔 Story / 🔕 Silent** toggle up top switches whether your actions write story text as they resolve, or just apply their effects quietly.

Below the composer: any **actions belonging to things in the scene** (a door, an NPC, an item) show up as their own buttons, grouped by what they're attached to — tap one to run it directly. A **Last Roll** readout at the bottom shows the dice math and outcome of whatever you just resolved.

---

## Part 6 — CYOA: Choose Your Own Adventure *(new)*

Sometimes you don't want to write your approach from scratch — you want a menu. That's what CYOA gives you.

- **Bridge:** open the left tray and pick **Action Choices**, **Dialog Choices**, or **Mixed Choices** (or **Image Studio** — see Part 9).
- **Native:** tap the **CYOA** toolbar button, then pick **Action**, **Dialog**, or **Mix**.

Either way, the engine reads the current moment and hands you four tagged options (aggressive, diplomatic, tactical, wildcard, and so on) suited to that mode. Above them you'll find the same attribute/stake/target pickers as the composer, so you can set those once and then just tap a choice.

- In the **Bridge**, tapping an option runs it immediately as an action.
- In the **Native** HUD, tapping an option loads its text into the Action screen's Goal & Approach box so you can review or tweak it before running it.

If you don't like any of the four, just close the widget — nothing has happened yet, and you can always fall back to writing your own approach in the Actions tab.

---

## Part 7 — The Player tab (Party Hub): you

This is your own character sheet, at a glance and in play.

### Editing your character

Look for a small **🪪** button (Bridge, top-right of the banner) or an **Edit Profile** button (Native, top of the Player screen). **This is where you go to enter or change your character's information** — name, appearance, personality, voice, background, attributes, and anything custom your game defines. It opens the same full character editor your GM uses for every character in the world; on this button, it's simply already pointed at you.

### The rest of the tab

Three sub-sections:

- **Stats** — your resources (health, stamina, whatever your world tracks), your standing with any factions, and your skill ratings. An **Edit/Lock** toggle unlocks these for direct adjustment when needed.
- **Inventory** — what you're carrying and wearing, plus (if you're traveling with others) each party member's own gear and resources.
- **Quests** — every narrative thread tied to you, tracked ones pinned to the top.

An **Open Journal** button sits at the top of this tab — covered next.

---

## Part 8 — The Journal *(new)*

The Journal is your own running record, written in your own words — and unlike a private diary, it actually feeds back into the story.

- **Active entries** ride along as "The story so far," so the AI is reminded of them no matter how long ago you wrote them or how far the conversation has scrolled.
- **Archived entries** are kept but never fed back in — good for old notes you don't need influencing things anymore, without losing them for good.

### Writing an entry

Type into the box at the bottom and hit **Add entry**. Check **Stamp with story date/time** if you want it tagged with the in-world moment it happened (handy for a long game where "when did that happen?" matters).

### What you can do to an existing entry

| Button | What it does |
|---|---|
| **Edit** | Rewrite it yourself. |
| **Stamp / Unstamp** | Toggle the in-story date/time tag. |
| **Archive / Unarchive** | Move it out of (or back into) what the AI actually sees. |
| **Rewrite** | Let the AI rewrite your note in the story's own voice. |
| **Compress** | Let the AI shorten it into a tighter recap. |
| **Delete** | Gone for good. |
| **Merge all** | Combine every active entry into one single note (appears once you have more than one) — good housekeeping once the list gets long. |

Use the Journal for anything you want the story to remember that never quite made it into the prose, to correct something, or just to keep your own recap of a long-running game.

---

## Part 9 — Image Studio

Opened from the camera icon on the left tray (Bridge) or reached via the Player/Inspect screens where a Snapshot button appears (Native). Give it a **Base Prompt** and, if you want, a **Negative/Unwanted** description and one or two **character names** to pull their look in automatically. Two assist buttons:

- **Auto-Snap** — reads back a handful of recent paragraphs and builds a prompt from them for you (adjust how many paragraphs it scans).
- **Enhance** — polishes a rough prompt into a fuller one before you commit to generating.

Hit **Generate** to render it with your world's active image preset. The result appears right in the widget with a download button and a one-tap **Regenerate**.

---

## Part 10 — The World Clock

If your game has time-of-day turned on, a small clock chip sits right on your location banner: the current in-story stamp, plus **+1h** / **+1d** shortcuts, a **Set** option to jump straight to a specific day and hour, and a 📅 button that opens the full **Calendar** for a wider view.

---

## Part 11 — Look & Feel: Themes

The gear/settings icon opens a theme picker — five ready-made looks (**Default Dark, Runestone, Cyberpunk, Unicorn Barf, Witch's Hollow**) plus a **Theme Forge** for building and saving your own color scheme. Purely cosmetic — nothing here changes how the game plays.

---

## Quick-Reference Cheat Sheet

**Two interfaces, same data:** Bridge (floating **🗺 AE** panel + side trays) and Native (NovelAI toolbar buttons) both read/write the same character sheets and world state.

**Bridge panel tabs:** Story (minimize) · Map (World) · Scene (Inspect) · Actions (Action) · Player (Party Hub).
**Native toolbar buttons:** Map · Inspect · Action · Player · CYOA · Calendar.

**Map:** Inspect / Travel Here / Teleport (silent) · Operate mode for vehicles.

**Scene:** roster grid (Scene-only or All) · tap anyone/anything for their detail view (Snapshot, Follow Me, Hooks, Inventory, Narratives) · minor NPCs can be Promoted to Major.

**Actions:** Goal & Approach composer (freeform text + optional attribute/stake/target/Luck) → Execute or Save as preset · entity-attached actions listed below · Story/Silent toggle · Last Roll readout.

**CYOA (new):** Action / Dialog / Mix → four tailored choices → pick one to run (Bridge) or load into the composer to review (Native).

**Player tab:** 🪪 / Edit Profile → **enter your character's info here** · Stats / Inventory / Quests sub-tabs · Open Journal button.

**Journal (new):** your own record in your own words · Active entries stay live in context as "The story so far"; Archived ones don't · per-entry Edit / Stamp / Archive / Rewrite / Compress / Delete · Merge all to consolidate.

**Image Studio:** Base Prompt + optional Negative + character names · Auto-Snap (from recent story) · Enhance · Generate → download/regenerate.

**Clock:** stamp + `+1h` / `+1d` / Set, right on the location banner · 📅 opens the Calendar.

**Themes:** gear icon → 5 built-ins + Theme Forge for custom looks. Cosmetic only.

**Want the mechanics underneath?** The Characters guide, the Resources & Actions guide, and the Scene Presence & Context guide in this series go deeper into how everything above actually works.
