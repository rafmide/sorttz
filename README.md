# Sorttz

Sorttz is a sorting algorithm visualizer I built because I wanted one place where I could watch an algorithm run, read its code in whatever language I happen to be using, and see what it costs, without hopping between three tabs.

Live: https://sorttz.vercel.app/

## What it does

- Visualizes 16 sorting algorithms: bubble, selection, insertion, shell, merge, quick, heap, counting, radix, bucket, comb, cocktail shaker, gnome, cycle, tim and intro sort.
- Shows the code for each one in 12 languages: TypeScript, JavaScript, Python, Go, Rust, C, C++, C#, Java, Kotlin, Swift and PHP.
- Shows best, average and worst case time, space, stability and whether it sorts in place.
- Races up to 4 algorithms side by side on the exact same input, and ranks them when they finish.
- Counts comparisons, writes and array accesses, and has a timer that runs while it plays.
- Lets you play, pause, step forward and back one operation at a time, scrub through the whole run, and change the speed from 1x to 1024x.
- Optional sound, one pitch per bar value. It is off by default.
- Benchmarks the real JavaScript implementations in your browser and draws growth curves from n = 64 to 4096.
- Remembers your settings and a history of your finished runs.

## Seeds

The numbers are a shuffled list of 1 to n, driven by a seeded generator (xmur3 into mulberry32). The same seed and the same size always give the same array, on any machine. Type your own seed or hit the shuffle button for a random one.

The link button copies a URL that reproduces the exact run:

```
https://sorttz.vercel.app/?algo=quick,merge&n=64&seed=sorttz&speed=8&lang=go
```

| Param   | Meaning                         | Values                                |
| ------- | ------------------------------- | ------------------------------------- |
| `algo`  | Lanes, comma separated, 1 to 4  | any algorithm id, for example `heap`  |
| `n`     | Number of bars                  | 8 to 256                              |
| `seed`  | Seed text                       | up to 24 letters, digits, space, `.-` |
| `speed` | Playback speed multiplier       | 1, 2, 4 ... 1024                      |
| `lang`  | Language shown in the code panel | `ts`, `go`, `rust`, `python` ...      |

Everything in the URL is validated and clamped before it is used. A bad value falls back to a default instead of breaking the page.

## How it works

Each algorithm runs once, up front, against a tracer. The tracer records every compare, swap and write into a flat `Int32Array`, then a replay builds a snapshot of the array every 256 steps. That gives playback a few useful properties:

- Scrubbing and stepping backwards are cheap. Seek restores the nearest snapshot and replays at most 256 steps.
- Every lane in a race advances at the same operations per second, so the algorithm that needs fewer operations really does finish first.
- Counters are exact, because they come from the same trace that is drawn.
- Each trace is checked against a sorted result before it is allowed to play.

The animated timer is wall clock time of the animation, so a higher speed gives a shorter time. It is not a measurement of how fast the algorithm is. For that, use the Benchmark section. It runs the same JavaScript that appears in the code panel on your machine and reports microseconds per sort.

The bucket sort animation uses ceil(sqrt(n)) buckets so you can see the work inside each bucket. The code panel uses n buckets, which is the textbook version.

## Where the code is tested

I wrote all 192 snippets so each one is self contained, in place, and sorts ascending integers. I ran every snippet against a reference sort on empty input, one and two elements, sorted, reversed, all equal, heavy duplicates, and a couple of hundred random arrays.

| Language   | Status                                                              |
| ---------- | ------------------------------------------------------------------- |
| TypeScript | Compiled with `tsc --strict`, all 16 pass                           |
| JavaScript | All 16 pass on Node 22                                              |
| Python     | All 16 pass on 3.13                                                 |
| Go         | `gofmt`, `go vet` and tests clean, all 16 pass                      |
| Rust       | Compiled with `-D warnings`, all 16 pass                            |
| C          | gcc 13 with `-Wall -Wextra -Werror` and ASan/UBSan, all 16 pass     |
| C++        | g++ 13 with the same flags, all 16 pass                             |
| Java       | `javac -Xlint:all -Werror`, all 16 pass                             |
| Kotlin     | Compiled with the Kotlin compiler, all 16 pass                      |
| PHP        | PHP 8.3 with strict types, all 16 pass                              |
| C#         | **Not compiled.** No .NET SDK was available. Reviewed by hand.      |
| Swift      | **Not compiled.** No Swift toolchain was available. Reviewed by hand. |

The code panel marks C# and Swift with a small "Not compiled yet" tag. If you have the toolchains, run them through a test harness and open a PR with fixes.

Two simplifications to know about. Radix sort handles non negative integers only. Tim sort is the simplified version (fixed runs of 32, insertion sorted, then merged bottom up) without run detection or galloping.

## Stack

One file, no build step, no framework, no server.

- HTML and plain JavaScript in `index.html`
- Tailwind CSS 4.3.3 through the browser build on the CDN
- Anime.js 4.5.0 for entrances, the finish sweep and the bar intro
- Chart.js 4.6.1 for the charts
- Lucide 1.61.0 for icons
- Inter Tight and JetBrains Mono from Google Fonts

All four scripts are pinned to an exact version and loaded with Subresource Integrity hashes, so a changed file on the CDN is refused instead of run.

The visualizer is drawn on a canvas, scaled for device pixel ratio, and only repaints when something changed.

Every control (the dropdowns, sliders, switch, segmented tabs, tooltips and toasts) is custom built. There is no native `<select>`, range input or checkbox. They support keyboard use and expose the right ARIA roles. Motion respects `prefers-reduced-motion`.

## Storage

No database, no account, nothing sent anywhere. Sorttz uses IndexedDB in your browser for:

- your last settings (algorithms, size, seed, speed, language, sound)
- the history of finished runs, capped at 200
- the last growth curve benchmark

If IndexedDB is blocked, for example in some private windows, it falls back to memory for that session and keeps working.

## Keyboard shortcuts

| Key         | Action                        |
| ----------- | ----------------------------- |
| `Space`     | Play or pause                 |
| `←` / `→`   | Step back or forward          |
| `Shift` + arrows | Step 25 at a time        |
| `R`         | Restart                       |
| `S`         | New random seed               |
| `M`         | Toggle sound                  |

## Run it locally

Open `index.html` in a browser. That is the whole setup. It needs internet access for the CDN scripts and fonts. If you prefer a server:

```sh
npx serve .
```

## Deploy

It is a static site. Put `index.html` and `og.png` in the same folder and deploy that folder to Vercel with no framework and no build command. `og.png` is the 1200 x 630 social preview the meta tags point to at `https://sorttz.vercel.app/og.png`.

## Files

```
index.html   the whole app: markup, styles, scripts
og.png       social preview image
README.md    this file
```

## Known limits

- Sizes go from 8 to 256 bars. Above that, a quadratic algorithm produces more steps than is pleasant to scrub.
- The native benchmark measures JavaScript. Timings for other languages would need those languages, so the code panel shows the code and the complexity, not a timing.
- Browser timers are coarse. Very fast sorts are batched and averaged, but treat small numbers as rough.
- Sorting is on distinct integers. Duplicates are covered in the snippet tests, not in the animation.
