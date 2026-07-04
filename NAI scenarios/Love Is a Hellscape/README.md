# Love Is a Hellscape — Setup & Walkthrough
### Firefox edition

This game is a custom HUD and rules layer (Adventure Engine) sitting on top of an LLM-driven NovelAI text adventure. The Game Master is the AI itself — it's improvising prose around the hard state the engine tracks (location, time, stamina, quests). If something the GM narrates doesn't quite match what's below, go with whatever you like.

---

## 1. Install

**The HUD (map, quest tracker, character sheets) is optional.** You can play with just the story file — the GM narrates off the module's config either way. Install the HUD if you want the visual layer; skip to "Just the story" if you don't.

### Just the story
Import `[AE] Love Is a Hellscape v1.scenario` into NovelAI as a new story. The module's config, characters, and world state load automatically the moment the story opens — no separate JSON import, no startup prompt. You're playing.

### With the Bridge + HUD
**1. Install Violentmonkey**
Get the browser (I use Firefox) extension from [violentmonkey.github.io](https://violentmonkey.github.io/). This is what runs the HUD script alongside NovelAI.

**2. Install the bridge script**
Open [the Adventure Engine Bridge + HUD script on Greasyfork](https://greasyfork.org/en/scripts/582085-adventure-engine-bridge-hud-v0-9-6) and click Install. Violentmonkey will prompt you to confirm.

**3. Import the story**
In NovelAI, import `[AE] Love Is a Hellscape v1.scenario` as a new story, same as above. Config loads automatically either way — the bridge just adds the HUD on top.

**4. Refresh your browser**
This step matters: refresh the NovelAI tab right after installing the script, *before* you do anything else. The engine and bridge scripts need to "handshake" with each other on page load, and a freshly-installed script won't be live until the page reloads.

**5. Pick your HUD**
Open the sidebar — you'll find a HUD selector there now. Choose whichever HUD style you want to use; you can switch later without reimporting anything.

> If the HUD doesn't show up or seems stuck the first time, refresh again. A failed first handshake almost always clears up on the second try. Worst case, the story itself is unaffected — the config already loaded when you imported it.

---

## 2. First launch

The story opens straight into play — the config loaded automatically on import, so there's no startup card and nothing to click through first.

1. Read the opening prompt — it sets the scene and tone (dark comedy, corporate-bureaucracy-meets-dating-sim, Hell as an office park).
2. **If you installed the Bridge + HUD:** open it from the sidebar (the icon depends on which HUD you picked in step 5 above). This is your map, quest tracker, and character sheets. **If you skipped the Bridge:** everything below still applies — just track it by reading the GM's narration instead of a map.
3. Create your character. With the HUD: **👑 (Home) → 🪪 (ID card)**. You only *need* to fill in a Name and a Dramatis Persona — everything else is optional flavor you can leave blank or fill in later. Without the HUD, describe your character in your first reply to the GM and it'll pick that up from there.

---

## 3. The systems, briefly

**Time & Stamina.** The day runs Dawn → Morning → Afternoon → Evening. Traveling between the Motel hub and most rooms (Cubicle Wastes, the Hexagon, Cowzits, etc.) burns one time step. Stamina (0–6) isn't a flat toll on every action — training costs 2 Stamina up front, Flirting only costs Stamina on a Partial or a miss, and Fighting costs Health instead of Stamina (see below). Sleeping in **your room** resets Stamina to full and rolls the day over to Morning.

**Attributes.** You have four stats — **Force, Finesse, Focus, Flair** — each a plain number that acts as your modifier on a check. Train them at: the Hexagon (Force), the Fourth Circle (Finesse — ask Snive nicely and he'll pretend not to watch), Cowzits Academy (Focus), the Velvet Hole-in-the-Wall (Flair). Each training attempt costs 2 Stamina + 1 Gold and rolls its own small 1d6-vs-DC4 check; success adds +1 to that stat permanently.

**Checks — the Quick Action composer.** There's no fixed action menu for most things. Open **Goal & Approach** from the HUD, describe what you're attempting in your own words, then pick a **Skill** (one of the four stats, or "No check") and a **Stake** (a named wager — the composer will suggest one automatically based on the skill you picked, e.g. Force → **Fight**, Flair → **Flirt**). Submit and it resolves on **2d6 + your stat modifier** against fail-forward bands: 6 or under is a clean **Failure**, 7–9 is a **Partial** (you get it, but it costs you), 10+ is a **Strong Hit** (clean success, no cost). You can spend Luck (0–5) to add a flat bonus to the roll before you submit.

**Flirting** isn't a separate system — it's the **Flirt** Stake (defaults to Flair). Describe the approach, pick Flirt, roll: a Strong Hit raises the target's Attitude a step for free; a Partial raises it too but costs you a point of Stamina; a miss costs the Stamina and nothing moves. Attitude climbs Hostile → Unfriendly → Neutral → Friendly → Devoted, and maxing it out with a character advances your relationship with them a stage and resets Attitude so you can climb again — a slow burn, not an instant meter-fill. **Fight** works the same way but against Health, for combat.

**Travel & schedules.** Must-hav Motel is your home base — most other locations (Cubicle Wastes, the Hexagon, Velvet Hole-in-the-Wall, Waffle House, the Fourth Circle, the Lake of Fire, Cowzits Academy) branch off it, and going to one costs a time step. Depravitae's Van and Violencia's Room are free side-trips straight from the Motel — no time cost. The three other main characters move around on their own clock as the day advances:

| | Morning | Afternoon | Evening |
|---|---|---|---|
| **Depravitae** | Cubicle Wastes | Waffle House | The Hexagon |
| **Violencia** | Must-hav Motel | Cowzits Academy | Velvet Hole-in-the-Wall* |
| **Lilibeth** | The 9th Floor | The Hexagon | Velvet Hole-in-the-Wall |

---

## 4. Walkthrough: to the Chatty Grimoire

1. **Accept "You're Late!"** — it should be waiting for you as your first quest. Accepting it lets you track it; the HUD will show the path to your next destination.
2. **Travel to the Cubicle Wastes.** This is your workplace. You'll run into Depravitae here — she's your hyperactive office-mate. Talk to her, get oriented.
3. **Accept and track "Get to Work!"** Your job is janitorial pest control: head down into the sub-basements.
4. **Travel to Sub-basement D.** This is where Hairy Patters — the Rat King — lives.
5. **Fight Hairy Patters.** There's no dedicated Fight button — open **Goal & Approach**, describe your attack, pick **Force** as the skill (it'll default to the **Fight** stake), and submit. He won't go down in one hit — a Strong Hit costs you nothing and hurts him; a Partial or a miss costs *you* Health too, so watch your own bar. When his Health finally breaks, he drops dead on the spot, you collect a small bounty, and everything he was carrying (including a certain fancy letter) is left behind in the room.
6. **Pick up/track "A Fancy Letter."** The quest target will now show you the path toward Cowzits Academy.
7. **Unlock Cowzits Academy.** You have two options:
   - Use the **"Write your name over Hairy's"** action — forge the letter using his acceptance into the academy. This is the intended, free path once you have the letter.
8. **Travel: Must-hav Motel → Cowzits Academy → Cowzits Grand Library.**
9. **Pick up the Chatty Grimoire.** It's sitting in the library, waiting for someone with questionable judgment. Congratulations — you now own a sentient book bound in human flesh that whispers at you, and it comes with a Fireball spell.

That's the critical path. Everything else — training your attributes, flirting your way through the cast, side items, the Lake of Fire's beachcombing, whatever Snive is selling — is yours to wander into at your own pace.

---

*This walkthrough reflects the module as built. The story is driven by an LLM Game Master, so exact wording, pacing, and minor details will vary run to run — treat this as a guide, not a script, and don't be afraid to improvise around it.*
