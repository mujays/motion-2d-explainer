# MOTION 2D ENGINE — Spec Reference (read fully before writing a spec)

The engine turns ONE JSON spec into a finished MP4 (default 9:16, 720x1280, 30 fps; other aspects via `format`).
**Fixed design concept (never change):** flat-vector illustration (solid fills + one shade + one highlight, rounded
2–3 px outlines), ONE continuous camera shot (no cuts, zoom-dives instead), punchy captions with one yellow keyword,
callout labels "│NAME" with leader line, comparison cards, big numbers, pulse rings, procedural sound effects,
hook title + takeaway outro.
**Free per topic:** the background/setting, the palette of the objects and what is drawn — pick what fits the topic
(space for planets, a kitchen for cooking, a body-pink set for health, a blueprint for engineering, paper/kids for
children's lessons…). You only write the spec; the engine keeps the look consistent.

## 1. Spec skeleton

```json
{
  "lang": "id",                       // "id" or "en" (affects pacing in script-only mode)
  "background": "paper",              // preset name (see §1b) or {"top":"#hex","bottom":"#hex","pattern":"grid|dots|lines","accent":"#hex"}
  "format": { "aspect": "9:16", "quality": 720, "fps": 30,
              "captions": {"show": true, "position": "bottom", "words": 3, "highlight": true,
                           "size": "sedang", "uppercase": false,
                           "mode": "spoken"},   // spoken: keyword turns yellow WHEN it is spoken (default)
                                                // karaoke: every word lights up while spoken · static: old style
              "title": true, "outro": true, "cta": "FOLLOW UNTUK LAINNYA",
              "sfx": true, "sfx_volume_db": 0, "ambience": true,
              "watermark": "@akun", "watermark_pos": "top-right" },   // all optional, these are defaults
  "design_aspect": "9:16",            // the aspect your world/camera were designed for (auto-reframe if different)
  "meta": {
    "title":  ["Kenapa laut", "asin?"],          // hook, 2 short lines (line 2 = yellow). Shown 0–3 s
    "outro":  ["Sungai bawa garam,", "matahari ambil air"],  // 1-line summary, 2 lines. Last ~3.4 s
    "cta": "FOLLOW UNTUK LAINNYA"
  },
  "script": "full narration text …",           // ALWAYS required (also in audio mode)
  "audio": "voice.mp3",                        // optional: narration file -> auto-sync
  "srt": "captions.srt",                       // optional: exact timings (e.g. CapCut export) instead of auto-sync
  "highlight": ["garam", "matahari"],          // optional: words to color yellow (1 per caption chunk). Omit = auto
  "ambience": [{"name": "rain"}],              // optional loop: rain | wind   (only if the scene has rain/wind)
  "camera": {"x": 360, "y": 640, "zoom": 1.0}, // start framing
  "world":  [ …objects… ],                     // drawn in list order (first = back)
  "beats":  [ …timed actions… ]
}
```
`"format"` may also be a path to a preset json (`"format": "preset.json"`). `"captions": false` hides subtitles
(the .srt is still written). `"cta": false` removes the CTA pill. `"reframe": false` disables auto-reframe.
Optional: `"caption_y": 0.775` (exact caption height as fraction of frame),
`"rate": 5.2` (syllables/s for script-only timing), `"tail": 1.2` (seconds after last word).

## 1b. Backgrounds (choose per topic — the UI adapts automatically)

| preset | mode | good for |
|---|---|---|
| paper, studio, notebook (lines) | light | general lessons, tips, history, language |
| kids (dots), lavender, peach, mint | light | children's content, cooking, lifestyle, health tips |
| day, sky, sunset, desert | light | outdoors, weather, geography, travel |
| lab (grid), body | light | chemistry/biology, human body, medicine |
| night, storm, dusk, deep | dark | weather at night, mystery, nature at night |
| space | dark | astronomy, planets |
| ocean, forest | dark | sea life, ecosystems |
| blueprint (grid), dark (dots), neon (grid) | dark | engineering, tech, how-things-work, gaming |
| chalkboard | dark | maths, school-style explanations |

Custom: `"background": {"top": "#FFF4D6", "bottom": "#FFD9E6", "pattern": "dots", "accent": "#7A5CFA"}`
(`pattern`: grid | dots | lines | omitted). Light/dark mode is detected from the colours and switches the theme tokens:

**Theme colour tokens** (use them as colour names anywhere, including inside SVG): `outline` (object outlines),
`ink` (in-world text, rulers, lines), `line` (label leaders), `pulse`, `hl` (travel dots/accents), `accent`.
On light backgrounds they are dark navy/orange; on dark backgrounds they are light/yellow.
Indoor scenes: put a `room` object first (wall + floor, optional window) — it covers the background.
`"theme"` is still accepted as an alias of `"background"` (old specs keep working).

## 2. Coordinates & camera

* World units = pixels of a 720x1280 canvas. Camera `{x,y}` is the world point at screen center; `zoom` 1 shows
  x 0…720, y 0…1280 around it. A world point appears on screen at
  `screenY = 640 + (worldY − camY) × zoom` (same for X with 360).
* **Standard layout (use it):** horizon (ground or sea surface) at world y ≈ 820–910. Sky things (sun, cloud) at
  y 150–350. Ground/sea fills the rest down to y 1130+.
* **Indoor layout:** `room` with floor ≈ 950–1000, tables/props standing on it, wall items (window, clock, chart)
  at y 250–500.
* **Caption safe zone:** screen y 930–1060 is reserved for captions. Keep important objects/labels so their screen
  y ≤ 900. When you zoom to z, choose camY ≥ horizonY − 215/z if the horizon must stay above the captions.
* **Title zone:** screen y 0–260 (portrait) is covered by the title during the first 3 s and by the outro in the last 3.4 s —
  keep the opening/closing framing calm (zoom 1, nothing important at the very top).
* **Layouts per aspect.** Scale is area-preserving, so at zoom 1 the visible world (W×H units) around the camera is:
  9:16 → 720×1280 · 4:5 → 858×1073 · 1:1 → 960×960 · 16:9 → 1280×720.
  General formula: `screenY = H/2 + (worldY − camY) × zoom × S` with S = sqrt(W·H / (720·1280)).
  Design for the chosen aspect and set `design_aspect` to it:
  - 9:16: the standard layout above.
  - 4:5 / 1:1: horizon at world y ≈ camY + 200 (e.g. camera y 640 → horizon ≈ 840); sky items within ±400 of camY;
    spread objects wider (x −100…820). Captions sit at ≈80–83 % of frame height.
  - 16:9: a wide panorama — place the story steps side by side along x (e.g. cloud/lightning at x 100–400,
    house at x 900–1100, sea at x 1200+), horizon at camY + 150…200, camera y ≈ horizon − 180. Pan along x
    instead of up/down. Captions sit at ≈85 % height; keep things above ≈75 %. Cards are 640 px wide — fine.
  - Captions off → you may use the whole lower part of the frame. Title/outro off → you may use the top.
* Zoom range 1.0–3.5 for normal moves; 4–7 for "dive into the microscopic world" (see particles + `min_zoom`).

## 3. World objects (`type` + params). Every object may also have `"id"`, `"hidden": true` (invisible until a `show`
action), `"min_zoom": z` (only visible when camera zoom ≥ z, fades in), `"screen": true` (HUD, ignores camera),
`"scale"` (1 = normal; animate with `set` to grow/shrink anything), and `"idle"`: idle life while on screen —
`float` (default for svg/prop), `breathe` (default for sun/planet), `sway` (plants, flags), `bob`, `pulse`, `spin`
(+`spin_speed` deg/s), `none`. Spec-level `"life": false` turns all idle motion off.

| type | params (defaults) | notes / anchor |
|---|---|---|
| ground | y=1000, color grass\|dirt\|sand\|"#hex" | flat land to the bottom; grass gets tufts |
| sea (alias water) | y=850, bottom=y+260, amp=4, wave=90, color, sparkle=true | animated waves + drifting sparkles |
| mountain | x, y(base), w=560, h=520, snow=true, color | anchor = (x,y) |
| river | points [[x,y]…], width=26, speed=40, dot_color | flowing dotted stream |
| rocks | x, y, n=4, s=1, bits=0 (falling salt bits), bits_color | |
| cloud | x, y, s=1, dark=false, rain=false, rain_to, density | gentle bob; `dark` for storms |
| rain | area [x0,y0,x1,y1], density=1, speed=1.6, color | slanted falling streaks |
| sun / moon | x, y, r=42; moon: shade (angle of its dark half, 0=right 90=down 180=left), shade_k | glowing disk; `shade` = moon phase / night side |
| lightning | from [x,y], to [x,y], amp=28, segments=7, seed, hot=0 | `hot` 0→1 turns it orange with sparks |
| house | x, y(base), s=1, lit=0, color, roof | `lit` 0→1 lights the window |
| tree | x, y, s=1, color | |
| building | x, y(base), w=90, h=220, lit=0.5, color | city skyline block |
| person | x, y(feet), s=1, color, shirt | simple character |
| planet | x, y, r=80, color, land=false, ring=false, glow=true, shade=35 (night-side angle), shade_k=0.35 | earth: land=true |
| stars | n=90, height=0.6, seed | screen-space twinkles (space/night) |
| vent | x, y(base), h=70, w=46, smoke=false | dark rock spire / hydrothermal vent |
| vapor | x, y, h=220, n=3, spread=50, color | rising wavy steam lines |
| ruler | x, y, len=300, ticks=4, highlight=0 | `highlight` 0→1 fills it yellow (distance/measure) |
| particles | area [x0,y0,x1,y1], groups:[{shape, color, n, r, motion, speed, show, lift, lift_frac}] | see below |
| circle | x, y, r, fill, stroke, sw, glow | generic |
| rect | x, y (center), w, h, radius, fill, stroke | generic |
| poly | points, fill, stroke | generic (any custom shape) |
| line | points, color, width, dash, arrow, draw (0→1 animates drawing) | generic path / graph line |
| arrow | from, to, color=orange, width=6 | |
| text | x, y, text, size=40, color (default ink), font ("title" for heavy) | in-world text that zooms with the world |
| **prop** | prop name, x, y (centre), w=120, rotate, flip, bob (idle float px), wobble (deg), recolor {"#hex": colour} | built-in flat icons: atom battery book car cell chart clock coin cup drop food gear heart house2 laptop leaf lightbulb magnifier person2 phone rocket star sun2 tree2 |
| **svg** | svg "<svg viewBox='0 0 100 100'>…</svg>", x, y (centre), w=120, rotate, flip, bob, wobble, recolor | ANY custom illustration (see §3b). Animate x, y, w, rotate with `set` |
| ellipse | x, y, rx, ry, rotate, fill, stroke, sw | |
| path (alias blob) | points [[x,y]…], closed=true, fill, stroke (default outline), sw, draw (open paths) | smooth Catmull-Rom curve: organs, hills, roads, waves, graph curves |
| room | floor=900 (world y of floor line), wall, floor_color, window [x,y,w,h] or true, window_sky, frame, boards | indoor set; put it first in `world` |
| table | x, y (floor), w=380, h=150, color | furniture; put objects on y − h |

**Particles** (molecules, ions, cells, dust, crowd dots): shapes `water` (red O + 2 white H), `plus`, `minus`,
`dot`, `square`, `ring`. motion `jiggle` (default) \| `rise` \| `fall` \| `flow`. `show` 0…1 = fraction visible
(animate with `set` → `groups.1.show`). `lift` = rise distance for a random `lift_frac` (0.5) of the group
(evaporation). For a micro dive: put a small particle area inside the macro scene (r≈2–3), give it
`"min_zoom": 2.5`, then `camera zoom 5–6` onto it.

Colors: named `yellow gold white blue lightblue sky navy red pink green purple orange grey dark brown rock
mountain grass dirt sand cyan`, theme tokens `outline ink line pulse hl accent`, or `"#RRGGBB"` / `[r,g,b]`.

### 3b. Drawing custom SVG in the house style
Use `svg` objects for anything not in the catalog (organs, tools, animals, food, machines, buildings, characters).
* viewBox `0 0 100 100` (or any), object centred at x,y, scaled to `w` world units.
* Flat only: 1 base fill + 1 darker shade + 1 small light highlight stroke (`stroke-linecap="round"`). No gradients,
  no filters, no text, no images.
* Outline every main shape with `stroke="outline" stroke-width="3" stroke-linejoin="round"` (theme-aware).
* Palette: friendly saturated colours — reds #EF5A6F, oranges #FFB02E, yellows #FFD25A, greens #5CC27A/#4CAF6A,
  blues #4FA8F0/#7FC4FF, purples #B05AA8, skin #F2C4A0, wood #8A5A3C/#D9954A, greys #9AA3C2/#2C3350, white #F2F4FB.
* Supported: path (M L H V C S Q T A Z), rect (rx), circle, ellipse, polygon, polyline, line, g + transform.
* Keep it simple (≤ 12 shapes). Big readable silhouettes beat detail — the camera zooms in anyway.
* Motion: animate with `set` on x / y / w / rotate (e.g. dough rising = w 220→320, rocket = y 900→200),
  `show`/`hide` to swap states (raw dough → baked bread), `bob`/`wobble` for idle life.

## 4. Beats (timeline)

```json
{"at": "phrase from the script", "do": [ action, action … ]}
```
* `at` = words copied from the script (searched in order, case/punctuation-insensitive). Beat fires at the
  START of the first word. `"^phrase"` = at the END of the phrase. Suffix `"+0.4s"`/`"-0.3s"` shifts.
  Also `"start"`, `"end"`, a number of seconds, or `"#12"` (word index).
* List beats in script order. Each action can have `"delay": s` and `"sfx": "name"` / `"sfx": null`.

| action | fields | auto SFX |
|---|---|---|
| camera | focus: [x,y] or object id, zoom, dur=1.2, ease io\|out\|back\|linear | whoosh |
| show | id or list of ids, dur=0.45, stagger=0.12, pop=true | pop |
| hide | id or list, dur=0.4 | – |
| set | id, props {any numeric param: x, y, hot, lit, highlight, draw, "groups.0.lift"…}, dur=0.8 | none (add `sfx`) |
| label | label "TEXT", target id or [x,y], point [dx,dy] (offset from target anchor), offset [sx,sy] screen px (default [50,-50]; negative x = box on the left), hold=2.6 or until "phrase" | typing |
| pulse | target, r=110, dur=1.6, color | blip |
| card | rows [{label, icon wave\|box\|dot, value 0–1, color, bar, text}], title, y=0.28 (screen fraction), hold=3.4 | swish + rise + ding |
| strike | id (lightning), count=2, flash=0.35, thunder_delay | thunder |
| flash | strength=0.4, dur=0.3 | – |
| travel | from, to (id or [x,y]), color=yellow, n=9, dur=1.4, arc=0.15, r=6, hold=0.6 | sparkle / swish |
| burst | target, h=500, w=120, n=4, color | boom |
| bigtext | bigtext "3,5%", y=0.42 or target, size=92, color=yellow, hold=1.8 | ding |
| shake | amp=9, dur=0.45 | – |
| sfx | sfx name, gain (dB) | – |

**Animation actions (use these a lot — they are what makes the video feel alive):**

| action | fields | use for |
|---|---|---|
| show + `anim` | anim: pop (default) \| grow (from its base, svg) \| slide-left/right/up/down (+distance) \| drop (falls + bounces) \| spin \| wipe (reveals left→right) \| fade; dur, stagger | every entrance — vary them |
| emphasize (alias punch) | id or list, scale=1.2, stagger | "this one!" — punch-scale an object when it is named |
| wiggle | id or list, dur=0.9 | wrong / alive / excited (svg rotates back and forth) |
| move | id, to [x,y] or object id, or path [[x,y]…] (smooth), dur=1.4, ease, orient (rotate along path) | objects travelling: food down a stem, car on a road, bee to a flower, molecule into a cell |
| stream | path [[x,y]…] or from/to (+bend), shape dot\|drop\|bubble\|arrow\|ray\|text (+text "CO2")\|prop (+prop), color, n=7, speed=160, size=8, dur=3 | continuous flows: water up roots, gas in/out, sunlight rays, blood, electricity, money, data |
| arrow | from, to (id or [x,y]), bend=0.25, color=accent, width=7, dash, draw=0.6, hold=1.8, pad=30 | cause → effect, "becomes", process steps |
| count | from=0, to, fmt "{:.0f}%" / "{:,.0f}", dur=1.4, hold, y or target, size, color | any number that matters: counts up with ticks + ding |
| glow | target, r=120, color, dur=2 | energy, attention, "it's this part" |
| set | id, props {scale, x, y, w, rotate, lit, hot, highlight, draw, "groups.N.show/lift"…} | gradual change: grow, heat up, fill, light up |

SFX names: pop tick typing whoosh swish ding chime sparkle bubble thunder boom riser blip rise.
Loops (ambience): rain wind. Title gets whoosh+pop, outro gets chime automatically.

## 4b. Motion rules (the video must never look static)
* Every sentence of the narration gets at least ONE object animation (show-anim, move, stream, set, emphasize,
  wiggle, arrow, count, glow) — camera moves and labels alone do not count.
* Never more than ~3 s without an object animation; aim for ≥ 0.5 animated beats per second.
* Processes are shown as processes: things flow (`stream`), travel (`move`), transform (`hide` A + `show` B with a
  flash), grow (`set scale`/`w`), and cause → effect gets an `arrow`.
* Name → punch: when the narration names an object, `emphasize` it (or `glow`/`pulse`) on that word.
* Numbers → `count`. Comparisons → `card`. Lists → staggered `show` with different anims.
* Use at least 3 different kinds among stream / move / arrow / emphasize / wiggle / count in every video.
* `--check` prints a motion report and warns about still stretches and low density — fix every warning.

## 5. Style rules (non-negotiable = consistency)

1. One world, one continuous shot: build the whole topic as ONE scene and move the camera between its parts.
   Never hard-cut; use zoom-dives for micro views.
2. Something changes every 1.5–3 s (see §4b): a camera move every 4–8 s, object animations in between.
3. Labels: 1–2 words, they are auto-UPPERCASE, max 2 on screen, put them next to the thing (offset ~40–80 px),
   and in the language of the narration.
4. Captions are automatic (2–4 words, one yellow keyword). Give `highlight` with the key nouns/verbs of the topic.
5. Title = the hook question (≤ 3 words per line). Outro = the one-line takeaway, then CTA pill.
6. Use cards for "A vs B" amounts, bigtext for a single striking number, ruler for distance, travel for
   "something moves from A to B", burst for explosions/expansion, strike for lightning/sparks.
7. Pick the background for the topic (§1b). Use built-in objects/props where they fit; draw everything else as
   `svg` in the house style (§3b). No photos, no invented gradients, no emoji, no real logos or known characters.
8. Start wide (zoom 1 showing the whole world), end wide (camera back to the opening framing before the outro).

## 6. Run

```bash
E=<skill folder>/engine/explainer.py           # see SKILL.md step 0
python $E <work>/spec.json --check                          # anchors, ids, beat gaps (fix warnings!)
python $E <work>/spec.json --preview 1,6,12,20,30,40        # contact sheet PNG — LOOK at it
python $E <work>/spec.json -o <work>/video.mp4              # full render (≈0.5× realtime)
# if a single run risks timing out: --part 1/3, --part 2/3, --part 3/3, then --finalize 3
```
Outputs next to the video: `video.srt` (captions) and `video.words.json` (word timings).
Paths in the spec (audio/srt) are relative to the spec file.

## 7. Worked example (excerpt of an older, simpler spec) — for complete, RICH specs read examples/fotosintesis.json (main model), roti.json, tatasurya.json

```json
"world": [
  {"id":"sun","type":"sun","x":600,"y":200},
  {"id":"cloud","type":"cloud","x":190,"y":190},
  {"id":"rain","type":"rain","area":[80,240,330,520],"hidden":true},
  {"id":"mtn","type":"mountain","x":250,"y":830,"w":600,"h":560},
  {"id":"river","type":"river","points":[[250,420],[205,520],[255,600],[330,700],[420,800]]},
  {"id":"sea","type":"sea","y":820,"bottom":1130},
  {"id":"mol","type":"particles","min_zoom":2.5,"area":[470,900,620,1010],"groups":[
     {"shape":"water","color":"red","n":46,"r":2.2},
     {"shape":"plus","color":"purple","n":9,"r":2.6,"show":0}]},
  {"id":"ground","type":"ground","y":1128,"color":"dirt"}
],
"beats": [
  {"at":"Semua berawal","do":[{"type":"camera","focus":[250,470],"zoom":1.5},{"type":"show","id":"rain"}]},
  {"at":"mengikis batuan","do":[{"type":"set","id":"rocks","props":{"bits":10},"sfx":"bubble"},
                                {"type":"label","label":"Batuan","target":"rocks","offset":[80,-70]}]},
  {"at":"Garamnya","do":[{"type":"camera","focus":[545,955],"zoom":6,"dur":1.6},
                         {"type":"set","id":"mol","props":{"groups.1.show":1},"dur":0.6,"sfx":"pop"}]},
  {"at":"ikut terbang","do":{"type":"set","id":"mol","props":{"groups.0.lift":140},"dur":3,"sfx":"rise"}}
]
```
