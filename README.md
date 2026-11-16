# Nana's Training

A phone-friendly weekly gym and running plan for two people, **Nana** and **Neel**, on one page with a name switch at the top. Every day shows each exercise as a photo card (start and finish positions), the sets and reps, a short coaching cue, and a "Watch video" link. Nana and Neel tick off each day as they finish it.

- **Live site:** https://neel-m20.github.io/nana-training/
- **Repo:** https://github.com/neel-m20/nana-training (moved from `neelm20/nana-training` in Nov 2026; the old `neelm20.github.io` link no longer works, see [Hosting](#hosting))
- **Direct links:** `#nana`, `#neel`, or a specific day such as `#neel-d20261124`
- Addressed "Hi Nana," and signed "By Neel, with love". Keep that tone.

---

## What's in the repo

| Path | What it is |
|---|---|
| `index.html` | **The whole site.** HTML, CSS and JavaScript in one file, with no build step and no dependencies. All plan data lives in its `<script>`. |
| `img/` | About 150 exercise photos named `<Exercise_ID>_0.jpg` (start) and `<Exercise_ID>_1.jpg` (finish). |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are. Keep it. |
| `README.md` | This file. |

## How to edit and publish

1. Edit `index.html`. You can preview it by opening the file in a browser, and you can add `#neel` or `#nana-d20261201` to the address to jump to a person or day.
2. Commit and push to `main`. GitHub Pages rebuilds in about 30–60 seconds.
3. Hard-refresh the phone if it still shows the old version.

Before pushing, a quick syntax check catches most mistakes:

```bash
python3 -c "import re;s=open('index.html').read();open('/tmp/chk.js','w').write('\n'.join(re.findall(r'<script>(.*?)</script>',s,re.S)))" && node --check /tmp/chk.js
```

---

## How the plan data works

Everything is in the `<script>` in `index.html`, in this order:

1. **Helpers** (`T`, `E`, `IMG`, `X`, `SS`, `M`, …). These are the building blocks for exercise cards.
2. **`PLAN_NANA`**: Nana's days, keyed by ISO date (`"2026-11-30"`). The older weeks are written out by hand.
3. **Neel's helpers** (`XI`, `nWarm`, `easyRun`, `n4x4`, `nLong`, `nRest`) and **`PLAN_NEEL`**: Neel's hand-written older days.
4. **Weekly generators**: the `NANA_WEEKS` and `NEEL_WEEKS` arrays. Each entry is one week (keyed by its Monday), and a `forEach` builds all 7 days from a template. **This is where new weeks go.**
5. **One-off overrides**: an IIFE after the generators that changes specific days (for example, Neel away 19–23 Nov). It runs after the generators, so it wins.
6. **`PEOPLE`**: per-person settings. These are which plan to use, the localStorage prefix, the greeting and the sign-off.
7. **Rendering**: day strip, week scrolling, cards, and done-ticks. You rarely need to touch this.

### Adding new weeks (the usual job)

Append an entry to `NANA_WEEKS` or `NEEL_WEEKS`; don't hand-write days.

```js
// Nana
{mon:"2027-01-04", heavy:true, light:false, run:14,
 lift:"Note shown on lifting days…",
 r1:"Cue for the Wednesday 4×4", r1rx:"4 × 4 min",          // r1rx optional
 r2:"40 min non-stop", r2note:"<b>Saturday note…</b>"},

// Neel
{mon:"2027-01-04", wk:1, light:false, main:"squat",        // "squat" | "deadlift", alternates weekly
 mainRx:"4 × 5 @ 75 kg", inc:2, long:"70 min"},           // inc: false | true | 2 = upper-body weight step
```

- `heavy` (Nana) means the main lifts are 4 × 8.
- `light` means a lighter week: 2 sets, 3 × 4 intervals, shorter runs.
- `holiday:true` swaps in the New Year wording.

### One-off changes (travel, sickness, a swapped day)

Add or replace dates in the override IIFE, which sits just before `const PEOPLE`. The pattern is to copy the generated day and change only what differs:

```js
PLAN_NEEL["2026-11-25"] = Object.assign({}, PLAN_NEEL["2026-11-24"], {note:"…"});
```

Watch the order. If you copy a day *after* you've overwritten it, you get the new version.

### Day fields

| Field | Meaning |
|---|---|
| `t` | `"lower"`, `"upper"`, `"run"` or `"rest"`. Sets the colour and icon. |
| `title`, `mins`, `note` | Heading, duration chip, and coach note (HTML allowed). |
| `items` | The list of exercise cards. |
| `done:true` | Pre-ticked. Use this only for days already completed before the site existed. |
| `rest`, `restHead` | A rest-day card instead of exercises. |
| `sick:true` | The get-well card. |
| `date:"Neel"` / `head` / `sub` / `then:null` | The picture card for runs together (Cubbon Park, breakfast and coffee). |
| `after` | Overrides the message shown after the day is ticked. |

### Exercise items

- `X(name, rx, cue, videoQuery, extra)` builds a card. The photo comes from the `IMG` map, or from `{im:"Exercise_ID"}`.
- `XI(name, rx, cue, imageId, videoQuery)` does the same with the image ID passed directly (used for Neel).
- `SS(name, rx, cue, q, M(...), M(...))` builds a superset, with two moves and a photo each.
- `M(name, reps, imageId)` is one move inside a superset or interval block.
- `E.warm`, `E.abs`, `E.core`, `E.stretch`, `E.flow` are shared blocks.

"Watch video" opens a YouTube **search** for `q`, so it never breaks.

### Photos

The images are public domain, from [yuhonas/free-exercise-db](https://github.com/yuhonas/free-exercise-db). To add an exercise:

```bash
ID=Barbell_Deadlift
for n in 0 1; do curl -sL "https://raw.githubusercontent.com/yuhonas/free-exercise-db/main/exercises/$ID/$n.jpg" -o "img/${ID}_$n.jpg"; done
sips -Z 480 -s formatOptions 72 img/${ID}_0.jpg img/${ID}_1.jpg   # resize ONLY the new files
```

The exercise IDs are listed in that repo's `dist/exercises.json`.

### Behaviour worth knowing

- **Ticks** are stored in the browser's `localStorage` (Nana `chk-<date>`, Neel `chk-neel-<date>`). They are per device and per site address. Moving the site to a new address starts with no ticks.
- **Future days are locked.** Today and past days can be ticked. Today never starts ticked.
- **Day strip:** it shows at most 2 weeks at a time (default last week + this week). You can scroll sideways or use ‹ Earlier / Later ›. Days without a plan show as faded chips.
- The design is a soft blue palette, warm on the eyes, with dark mode supported. Keep it phone-first.

---

## The training logic (read before changing plans)

The plans were written as a coach would write them, so every change needs a whole-week check (recovery, soreness, what comes the next day), not just a literal edit.

**Nana** (25, lean). Goals: muscle, a little fat loss, strength, flexibility, running stamina.
- Week: **Mon Lower A · Tue Upper A · Wed Norwegian 4×4 · Thu Lower B · Fri Upper B · Sat easy run · Sun rest.**
- Runs always come after an upper day, never after legs.
- The warm-up is mobility only: arm circles, shoulder circles, head rotations, hip rotations, back swings, toe touches, hamstring stretch, leg swings. No rowing machine or cross-trainer.
- Interval day is real running at 8/10 effort, not jogging. Power-walking up an incline is allowed for the hard reps.
- Saturday run progression: run/walk → 30 min non-stop (week 6) → 35 → 40 min. The running-plan table on the page tracks this.
- Calves are trained twice a week, she gets box jumps weekly, and Bulgarian split squats are on Lower B.
- Every 4th week is lighter. Heavier blocks run 4 × 8 on the main lifts.

**Neel** (endurance runner, ex-hiker, rebuilding the muscle he lost to running).
- Week: **Mon Legs · Tue Upper A + optional easy run · Wed Zone 2 + durability · Thu football or 4×4 · Fri Upper B heavy · Sat long run · Sun rest.**
- Leg day **alternates squat week and deadlift week**, adding 2.5–5 kg each time a lift comes round. These stay every week: single-leg RDL, hip thrust, Bulgarian split squat, slow leg curl, Smith calves, hip abduction/adduction superset, box jumps.
- Deadlifts stay on Monday only (never before the Thursday sprints or the long run).
- His gym has **no barbell bench** (use DB press) and **no Nordic curl setup** (use the slow-lowering leg curl).
- Rebuild phase (19 Oct 2026 – 3 Jan 2027): the long run is capped at 70 min, protein is about 2 g/kg, and he eats about 200–300 kcal above maintenance.
- Every 4th week is lighter. Time off counts as the lighter week.
- He logs his sessions in the Motra app. Set the next weights from those logs.

Saturday long runs are often together at Cubbon Park, Bengaluru, followed by breakfast. Those days use the picture card.

**Planned so far:** both people through Sun 3 Jan 2027. The next block from 4 Jan is not written yet.

---

## Hosting

The site is a static GitHub Pages site served from **`main`, folder `/` (root)**.

Owner: **neel-m20** (transferred from **neelm20**).

After the transfer (or any future one):

1. Check **Settings → Pages**. The source should be "Deploy from a branch", `main`, `/ (root)`. Re-enable it if it's off.
2. The URL becomes `https://neel-m20.github.io/nana-training/`. GitHub redirects git URLs, but **not** Pages URLs, so share the new link and update any phone home-screen shortcut. Ticks saved on the old link don't carry over.
3. Point local clones at the new remote: `git remote set-url origin https://github.com/neel-m20/nana-training.git`
4. Pushing needs git signed in as **neel-m20** (or neelm20 added as a collaborator with write access).
