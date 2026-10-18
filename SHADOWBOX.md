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

## Rebuilding it

The generated HTML is **not** a source. Edit the prototype or the builder, never the output.

The builder lives with the renderer it inlines, in `warband-live`:

```
node scripts/build-shadowbox.mjs \
  --app-ref   prototype/shadowbox \
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

1. The Google Fonts `<link>` is dropped.
2. Two CSS rules redirect the renderer's two font families onto locally installed stacks.
   The renderer names fonts in SVG `font-family` **presentation attributes**, which any CSS
   rule outranks, so this needs no edit to `kit.js` or `sb.js`. A locally installed Inter or
   Lilita One is first in each stack and still wins, so a machine that has them renders
   exactly as the prototype did.
3. The five `<script src>` tags become five inline `<script>` blocks in the same order.
4. The `<title>` says the build is experimental.
5. A dismissible notice and a permanent provenance stamp are added **over** the page, so the
   renderer's own muster screen and start button are untouched. The notice is removed from
   the DOM when dismissed, so it cannot swallow a click meant for the board.

## Self-containment

No externally loaded JavaScript, stylesheet, font, image, XHR, socket or backend. Verified
by opening the file in Chromium **with every URL blocked and the connection emulated
offline**, playing a complete game, and recording that no off-origin request was even
attempted. The single `http` string in the file is the anchor in the notice that links back
to the established playtest — a hyperlink, not a loaded resource.

`?seed=N` works as it does in the prototype, and defaults to 28.

## A match cannot be finished yet — read this before publishing

**This is the reason not to merge.** On both requested seeds the game freezes at turn 7 and
cannot continue:

```
seed 28   phase=play turn=7 active=1 pools=0/0 against=45/46  (one point from a break)
seed  3   phase=play turn=7 active=1 pools=4/6 against=42/54
```

It is the bot's turn in both cases. The page is **idle, not busy and not looping** — a
repaint counter sat at 101 with no change over sixteen seconds — and no error reaches the
console. On seed 3 the bot still has five live units with `acted: 0` and costs of 1, 3, 1, 2
and 1 against a pool of 6, so it is not short of legal moves; it simply never takes one. The
renderer's click handler returns early unless `active === HUMAN`, so **a player has no way
to recover**: the match is dead where it stands.

**It is not this build.** Brian's page, byte-identical and served from its own directory
with its real webfonts and five separate script files, settles into the same state on the
same turn with the same damage and the same exercised tally. The fault is in PR #4's
turn-driving glue, not in inlining.

The root cause is **not** established here, and three plausible explanations were each
killed by measurement rather than argument — an empty bot pool (false on seed 3), a turn cap
(the rules set `max_turns` to 10), and `afterMove()` spinning on a null from `wbBotChoose`
(false; the page does not repaint at all). Pinning it down belongs with the renderer's
author. Until it is fixed, a playtester invited to this page will reach turn 7 and stop.

## Known limitations, reported rather than designed around

- **Desktop only, measured.** The page is one 1920×1080 canvas scaled to the viewport, so a
  small screen shrinks everything rather than reflowing it. Nothing clips and no scrollbar
  appears at any size, but the type shrinks with the board:

  | viewport | scale | smallest label | labels under 10px |
  |---|---|---|---|
  | 1920×1080 | 1.00 | 22.0px | 0 of 12 |
  | 1366×768 | 0.71 | 15.6px | 0 of 12 |
  | 640×360 | 0.33 | **7.3px** | **7 of 12** |

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

## Licensing and permissions

- The rules are MIT, copyright Frank Trevor Gilson, from `warband-prototype`. MIT asks that
  the notice travel with substantial portions; the existing `index.html` carries none, so
  this is a standing question for both pages rather than something introduced here.
- **The Shadow-Box renderer is `brianhschneider`'s work**, in `Everyone-Loves-Beacon/warband-live`,
  a private repository in another organisation **with no declared licence**. Publishing it
  from this public repository needs his explicit agreement. That agreement is not recorded
  anywhere this build can see, so it is the open question in front of this change.
