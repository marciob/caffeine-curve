# Caffeine Curve

An interactive chart of the caffeine in your body through one day. Drag your cups along the timeline and see how much is still in you at bed time.

**Live:** https://caffeine-curve-sigma.vercel.app

## Where this comes from

This is a working rebuild of a UI designed by **Charles Postiaux** ([@PostiauxCharles](https://x.com/PostiauxCharles)). He posted it as a 14-second video on X on 9 November 2026:

> https://x.com/PostiauxCharles/status/2119801461278146758

The post showed the design only, with no code or link. This repository recreates the look and the interactions from a frame-by-frame reading of that video, so the design can be used and tested. All design credit goes to him. The rebuild was done with Claude Code.

Differences from the original are listed at the end.

## How to use

- **Hover** over the chart to read the level at any time of day: `18:55 · 95 MG`.
- **Drag** a cup (the dot under the axis, or the bright column just after it) to move it.
- **Click** an empty spot on the chart to add a cup. Click a selected cup to remove it.
- **Reset** (top right, circular arrow) puts the three default cups back: 08:00, 11:00 and 15:30.
- **Settings** (top right, sliders) opens the parameters. They are saved in your browser. **Defaults** inside the panel resets only the parameters.
- Keyboard: `Esc` deselects, `Delete` removes the selected cup, arrow keys move it by 15 minutes.

On a phone the page uses the full screen. Touch and hold on the chart to read a value. Tap to add a cup.

## Parameters

| Parameter | Default | Range | Meaning |
|---|---|---|---|
| Cup size | 95 mg | 10 to 400 mg | Caffeine in one cup |
| Half-life | 5 h | 2 to 12 h | Time for the body to clear half of it |
| Time to peak | 65 min | 15 to 120 min | Minutes from the cup to the highest level |
| Bed time | 23:00 | day start + 10 h to + 23:45 | When the level is checked |
| Sleep limit | 50 mg | 10 to 300 mg | The dashed line. Target level at bed |
| Day starts | 06:00 | 00:00 to 12:00 | Left edge of the chart |

## The model

Each cup follows a one-compartment model with first-order absorption and elimination (the Bateman function):

```
level(t) = dose · ka / (ka − ke) · (e^(−ke·t) − e^(−ka·t))
ke = ln 2 / half-life
ka is solved from "time to peak" so that the curve peaks at that minute
```

The chart is the sum over all cups. The defaults (95 mg, 5 h, 65 min) reproduce the numbers visible in the video: 3 cups at 08:00, 11:00 and 15:30 give `67 MG STILL IN YOU` at 23:00 and `LAST CUP BY 10:45`.

`LAST CUP BY` is the latest time, on a 15-minute grid, at which the last cup could be taken so that the level stays under the sleep limit from bed time through the following 7 hours, given the other cups. If the other cups alone already cross the limit, the footer says `OVER THE LINE EVEN WITHOUT THE LAST CUP`.

## Rendering notes

Everything is drawn on one `<canvas>`. No libraries.

- Dot matrix with an 8 px pitch. Each column is one time step, each row 1/47 of the chart's scale.
- Inside a column, brightness and dot size rise from the bottom to the top.
- The top brightness of a column depends on the time since the last cup. It is bright for about one hour, then settles to a dim floor.
- A blurred copy of the shape is drawn as a faint field of small dots (the halo).
- The vertical scale is fixed at 160 mg and grows when the peak passes it.
- The selected cup's share is stacked on top and drawn in blue. The blue fades to grey about 1.5 hours after the cup.
- The column at each cup's exact time is highlighted; for unselected cups the dots appear as rings.
- The `PEAK` marker hides when it would collide with the `BED` label.

## Run locally

Open `index.html` in a browser, or serve the folder:

```
npx serve .
```

Fonts: SF Mono when the system has it, otherwise Geist Mono from Google Fonts.

## Differences from the original

- The exact font, dot sizes and fade curves are estimates from a compressed video.
- The 50 mg line follows the chart's scale. In the video it stays fixed.
- The settings panel, the phone layout, keyboard controls and browser storage are additions.

## License

Code: [MIT](LICENSE). Design: Charles Postiaux, used here as a study rebuild with credit.
