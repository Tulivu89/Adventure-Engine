# Adventure Engine

A text-adventure / RPG layer for NovelAI's Text Adventure mode. Adventure Engine turns a freeform story into a stateful game: characters with resources and relationships,
a scene that knows who's in the room, a dice-driven action system, a living
journal, and a HUD to see and steer all of it — without leaving the NAI editor.

It's two userscripts working as one system:

|           | **Engine** (`adventure-engine-v1.0.0.txt`)       | **Bridge + HUD** (`ae-bridge-hud-v1.0.0.user.js`)    |
| --------- | ------------------------------------------------ | ---------------------------------------------------- |
| Runs in   | NovelAI's own scripting sandbox (Web Worker)     | Your browser, as a Violentmonkey/Tampermonkey script |
| Required? | Yes — this is the game                           | No — optional, but this is the visual half           |
| Does      | All logic: state, storage, dice, story injection | All display: HUD, map, gallery, theming, images      |

The engine alone is playable through a native fallback toolbar built into NovelAI's
own UI. Install the bridge too and you get the full HUD experience: tabs, panels,
themes, portraits, and generated scene art.

---

## For Players

**A world that remembers.** Characters carry resources (HP, stamina, relationship
stages, custom stats), schedules, and knowledge of what they have and haven't
witnessed. None of it depends on how much fits in the AI's context window — it
lives in structured storage and gets rebuilt into the story every turn.

**A scene that's aware of itself.** The engine tracks who's actually present in
the current location and feeds the AI a live, per-character "dynamics" snapshot
each turn — not just a character sheet, but where things currently stand between
you and them.

**Actions with real stakes.** A 2d6 + attribute dice engine resolves checks against
requirements, branching outcomes, and effects that can ripple across characters,
factions, and the world. Actions can be one-off template moves or become a
permanent part of a specific character or item.

**Choose-Your-Own-Adventure prompts.** On demand, the engine generates four
contextual options — action, social, or general — grounded in your character's
current state, for when you want a nudge instead of a blank page.

**A Journal that survives.** A player-authored running record of "the story so
far," stored outside the AI's context and reinjected as an always-on lorebook
entry, with archive/unarchive so old entries stop competing for space without
being lost. Entries can be AI-assisted (rewrite/compress) on request.

**Characters worth knowing.** A full character system — identity, four core
attributes, relationships, daily schedules, custom sheet fields — with a
World-Build Wizard chat that can batch-generate characters and world content from
a conversation instead of filling out forms one field at a time. A Bestiary
handles disposable/minor NPCs separately from named characters, with promotion
from one to the other when a throwaway NPC turns out to matter.

**Travel that means something.** Moving between locations rebuilds the world
around you — schedules advance, presence changes — and travel "tiers" make a
short step feel different from a long journey.

**A HUD built for the story, not around it.** Map, Scene/Inspect, Actions, Party,
Journal, and Image Studio tabs, a world clock with in-story time (not your
wall-clock), and a full Theme Forge with named skins (Cyberpunk, Runestone,
Unicorn Barf, Witch's Hollow) plus a custom theme editor. A no-bridge native
toolbar mirrors the essentials if you'd rather not install a userscript manager.

**Art that keeps pace with the story.** An Image Studio generates character
portraits, location scenes, and moment-to-moment illustrations of what's actually
happening — either automatically as events unfold ("auto-snap") or on demand
through a guided Director's Cut mode for a specific shot. Five swappable
generation presets and a persistent local gallery/roster keep everything
generated so far in one place.

---

## For Worldbuilders

Everything that shapes *your* world rather than just playing in one.

**Build the world by talking to it.** The World-Build Wizard is a chat surface
that turns a conversation about your setting into actual characters, locations,
and world content — batch-generated from what you describe, instead of filling
out forms one field at a time. Manual, field-by-field editing is still there for
when you want precision instead of speed.

**Design your own skill and resource systems.** Characters run on four core
attributes plus fully custom sheet fields, so you're not locked into a fixed
stat block. Resources can be global (shared party-wide) or personal to a
character, and can represent anything — health, reputation, currency, a countdown.
Factions, relationship archetypes and stages, and "stakes" (what's actually being
risked in a given moment) are all yours to define, not hardcoded assumptions
about what an RPG is.

**The Action Studio is where the real depth lives.** Every action is built in
phases — requirements, dice resolution and outcomes, effects, and branching
alternatives — and can target the self, another character, or the whole scene.
Actions can be one-off templates or become a permanent, evolving part of a
specific character or item, and one action can chain directly into another for
multi-step consequences. This is the tool for anything from a simple skill check
to a full combat or negotiation system.

**Things that run themselves.** Auto and Global flags let actions self-trigger
on any eligible entity without you invoking them by hand — a trap that springs
itself, a status effect that ticks every turn, a rule that quietly applies to
everyone in a faction. Schedules move NPCs through their day whether or not the
player is watching. CYOA generation and auto-snap illustration are the same
idea applied to prompting and art: the world keeps producing content without
you asking for each piece individually.

**Take your world with you.** A world-pack system can bundle your entire
configured world — characters, resources, actions, the lot — into the story file
itself, so sharing a `.scenario` export hands a new player a fully pre-built
world on import. A plain-JSON import/export path also exists for when you want
something human-readable and tool-friendly instead.
