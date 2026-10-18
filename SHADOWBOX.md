# shadowbox.html — an experimental Shadow-Box playtest

A second, experimental playtest page served beside the established one. It is **not** a
replacement for `index.html`, which is untouched by this change and remains the supported
build.

```
                        this page                    index.html
  renderer              Shadow-Box (experimental)    the canonical interface
  opponent              the engine's sparring bot     the engine's sparring bot
  players               one human                     one human
  layout                1920×1080 canvas, scaled      responsive
  interaction           preview, then Take            direct
  status                experiment                    supported
```

## What it is built from

Two pinned commits, and nothing else.

| part | repository | commit |
|---|---|---|
| canonical rules | `frankgilson-lab/warband-prototype` | `31ed7774cef58a545066301845271646d3143030` |
| Shadow-Box renderer | `Everyone-Loves-Beacon/warband-live`, PR #4 | `0da6f6b2314122851acbbd819bd016573071498d` |

Both SHAs are stamped into the page: in full in the opening notice and in the `title` of
the stamp at the bottom right, in short form in the stamp itself. Feedback on this page can
therefore be tied to an exact build.

The three rules files are carried byte for byte:

| file | bytes | sha256 |
|---|---|---|
| `config.js` | 68,105 | `ba35f20ab33e4e401ae88b9f139ffbf38e911d0c5ce0a05694331599ddd54f38` |
| `core.js` | 122,593 | `b175ded9f00c4b029beb83acc04a60749e2bbeee84788bd785709c0183cb4790` |
| `engine.js` | 508,065 | `ae1ce0780b2e6dfca52738ca71e69329adb3e65822257b3d6b9ecc5703153f42` |

And the artifact they make: **800,497 bytes**, sha256
`7d4e605da43402da6cfa8a176d5d326a89e31d0b905cb4c9a730dc5b13a2c439`.

## Two works, two sets of terms

This matters more than its position in a README suggests, so it is stated before the build
instructions rather than after the limitations.

- **The canonical Warband rules engine is MIT-licensed**, Copyright (c) 2026 Frank Trevor
  Gilson. MIT asks that the copyright notice and the permission notice accompany every copy
  and every substantial portion, and a page that inlines the whole engine is a substantial
  portion. So the licence is **embedded verbatim, twice**: as an HTML comment at the top of
  the file, where it can never be dismissed or restyled away, and as a readable panel in the
  opening notice. Both copies are `31ed777:LICENSE` byte for byte — 1,076 bytes, sha256
  `f736741c65c17618a6767290792c4c94368bbd60e415889725f18186f9a84289`.
- **The Shadow-Box renderer and its graphics are Brian Schneider's work** (@brianhschneider),
  published here **with his permission**. Permission is not a licence: they are **not** MIT,
  and nothing in this repository grants anyone the right to reuse them. The page says so in
  the same words, in the notice and in the permanent stamp.

The builder reads the licence out of the same commit as the rules and **refuses to build**
if it is missing or is not the MIT text — so the notice travels with every future build
rather than with the one artifact somebody remembered to paste it into. Both refusals are
exercised in `scripts/shadowbox-acceptance.mjs`'s evidence; a licence containing `--` is
also refused, because it could not be carried verbatim inside an HTML comment.

## Rebuilding it

The generated HTML is **not** a source. Edit the prototype or the builder, never the output.

The builder lives with the renderer it inlines, in `warband-live`:

```
node scripts/build-shadowbox.mjs \
  --app-ref   0da6f6b \
  --rules-repo ../warband \
  --rules-ref  31ed777 \
  --out        ../warband-playtest/shadowbox.html
```

It reads both inputs out of git rather than the working tree, so a dirty checkout cannot
leak into a build. It then **re-derives the three rules files from the upstream commit and
refuses to build if they differ by a byte** — `rules/MIRROR.json` is deliberately not
trusted, because a file that records its own hashes cannot attest to them. `--explain`
prints the list of changes applied.

## What the builder changes, and the list is complete

The renderer is copied verbatim. `index.html`, `kit.js` and `sb.js` are inlined as they
are; gameplay, balance, the sparring bot and every interaction decision are untouched.
**Preview-then-Take stays an experiment of this prototype's** and is labelled as such on
the page — it is not adopted for the canonical interface.

1. The verbatim MIT licence of the rules goes in at the top of the file, with the
   renderer's separate terms stated beside it.
2. The Google Fonts `<link>` is dropped.
3. Two CSS rules redirect the renderer's two font families onto locally installed stacks.
   The renderer names fonts in SVG `font-family` **presentation attributes**, which any CSS
   rule outranks, so this needs no edit to `kit.js` or `sb.js`. A locally installed Inter or
   Lilita One is first in each stack and still wins, so a machine that has them renders
   exactly as the prototype did.
4. The five `<script src>` tags become five inline `<script>` blocks in the same order, each
   headed by a comment naming its source commit and its terms.
5. The `<title>` says the build is experimental.
6. A dismissible notice — now carrying the licence and the attribution — and a permanent
   provenance stamp are added **over** the page, so the renderer's own muster screen and
   start button are untouched. The notice is removed from the DOM when dismissed, so it
   cannot swallow a click meant for the board; the stamp stays, and carries the attribution
   in words rather than only in two short SHAs.

## Self-containment

No externally loaded JavaScript, stylesheet, font, image, XHR, socket or backend. Verified
by opening the file in Chromium **with every URL blocked and the connection emulated
offline**, playing both games to their verdicts, and recording that no off-origin request
was even attempted. The single `http` string in the file is the anchor in the notice that
links back to the established playtest — a hyperlink, not a loaded resource.

`?seed=N` works as it does in the prototype, and defaults to 28.

## Both seeds play to a verdict

Both requested seeds were played end to end **with every URL blocked and the connection
emulated offline**, by a driver that clicks marks the page painted and never computes
legality. The verdict is read from `wbState()`, the engine's own accessor — **not** from the
view-model handed to `SB.paint`, which stops updating once `S.phase === 'over'` and would
show a finished match as a frozen one. That mistake produced a wrong defect report against
the renderer in an earlier round; it is why the check is written this way.

```
seed  3   over at turn 7    against 54/12 of 54/48   Company A broke    alive A 4  B 5
seed 28   over at turn 10   against 48/8  of 46/51   Company A broke    alive A 2  B 5
```

For both: `wbLegalMoves(wbState())` returns **0** at the ending, the **Chronicle is on
screen**, **zero console errors**, and **no off-origin request was even attempted**. Between
them the two runs exercised the draft, the colours, deployment, movement, plain strikes,
Pass, Rally, Brace, Anchor and a Marking Shot, and no click that changed nothing was counted
as an action.

`scripts/shadowbox-acceptance.mjs`, in `warband-live`, is the run:

```
node scripts/shadowbox-acceptance.mjs ../warband-playtest/shadowbox.html   --rules-repo ../warband --rules-ref 31ed777
```

61 checks, all passing.

## Known limitations, reported rather than designed around

- **Desktop only, measured during play.** The page is one 1920×1080 canvas scaled to the
  viewport, so a small screen shrinks everything rather than reflowing it. Nothing clips and
  no scrollbar appears at any size, but the type shrinks with the board:

  | viewport | scale | smallest label | labels under 10px |
  |---|---|---|---|
  | 1920×1080 | 1.00 | 13.2px | 0 of 18 |
  | 1366×768 | 0.71 | 9.4px | 1 of 18 |
  | 640×360 | 0.33 | **4.4px** | **9 of 18** |

  **An earlier version of this table was measured on the wrong screen.** It reported 22.0px,
  15.6px and 7.3px across 12 labels, and said no label fell under 10px at 1366×768. Those
  numbers came from the muster screen: the measuring run clicked the start button without
  first dismissing the notice that covers it, so the game never began and the table
  described a title card. The figures above are from a game in progress, which is what a
  playtester will be looking at.

  At 640×360 most of the interface is unreadable. There is no mobile layout, and adding one
  would be a redesign of Brian's prototype rather than a build step, so it is reported here
  instead. The added notice does adapt — it scrolls and compacts below 640px of height, so
  its text and its button stay reachable — because a label nobody can read is not a label.
- **No accessibility parity** with `index.html`. The board is SVG driven by pointer
  hit-testing on coordinates, with no keyboard path to a hex, no focus order and no
  accessible names for pieces. The added notice is keyboard-reachable and dismissible with
  Escape, which is the one accessible thing this build contributes; it does not extend to
  the game itself.
- **One human against the bot.** No hotseat, no second human, no audience.
- **Not Warband Live.** The streamer-versus-audience prototype is separate and is not part
  of this page.
