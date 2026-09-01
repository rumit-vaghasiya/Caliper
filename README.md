# Caliper

Fluid type sizing for CSS and Webflow. Pick a desktop size, get a sensible mobile size, see how it behaves across the whole viewport range, and copy the `clamp()`.

One HTML file. No build step, no dependencies, no network calls. Open it in a browser and it works, including offline.

<!-- Drop a screenshot here as screenshot.png and uncomment:
![Caliper](screenshot.png)
-->

## Quick start

1. Download `caliper.html`.
2. Open it in any browser, or drag it onto a browser tab.
3. Set your viewport range once (defaults are 320px to 1440px).
4. Type a desktop size. Copy the output.

To keep it handy, bookmark the local file or drop it on a static host.

## Two modes

### Single size

For the everyday case: one heading, one value.

Enter the desktop size. Caliper suggests the mobile size by keeping your body size as a floor and removing half the difference above it, so a 4rem heading suggests 2.5rem, 3rem suggests 2rem, and 1rem stays 1rem. The shrink slider adjusts how aggressive that is. Turn off **Suggest** to type your own.

### Type scale

For a full H1 to H6 set from a modular ratio.

**Build the scale from** decides which end is fixed:

- **H1, scaling down.** You set the H1 and everything divides down by the ratio. Good when the hero size is a design decision you already made. Watch the bottom of the scale: a large H1 with a small ratio leaves H6 oversized.
- **Body text, scaling up.** You set the body size and the ratio climbs from there. H1 lands wherever it lands. This keeps small headings sensible and is the safer default for a system you will reuse.

Desktop and mobile get separate ratios. A tighter mobile ratio (1.125 or 1.2) lets the H1 come down hard without crushing H5 and H6.

Every row in the hierarchy table is editable. Override any min or max and the output follows. **Reset to ratio** clears all overrides.

## Output formats

| Format | What you get |
| --- | --- |
| `clamp()` | The bare value, ready to paste into a size field or a variable |
| Declaration | `font-size: clamp(...);` plus optional line-height and letter-spacing |
| Fluid formula | The `clamp()` written with `var()` viewport variables, so one change updates every size |
| CSS variables | A `:root` block with `--font-size--h1` and friends |
| CSS rules | Selector blocks per heading level, for pasting into custom code |

Copy button on the main output, plus a copy button on every row of the hierarchy table.

## Using it with Webflow

The plain `clamp()` output pastes straight into a font size field as a custom value, or into a Webflow variable of type Size.

The fluid formula needs two unitless numbers available in CSS:

```css
:root {
  --site--viewport-min: 20;   /* 320px at a 16px root */
  --site--viewport-max: 90;   /* 1440px at a 16px root */
}
```

Put that in site-wide custom code. They are unitless on purpose, since the formula divides by them. Once they exist, every size using the formula responds to a change in your viewport range without regenerating anything.

Rename the prefix in the **Variable names** field if your project uses a different convention. It updates the output live.

The breakpoint table uses Webflow's breakpoints, so you can check a size at 991px before you find it in the Designer.

## The maths

All of it in rem, where `vwMin` and `vwMax` are your viewport range converted to rem:

```
slope     = (maxSize - minSize) / (vwMax - vwMin)
intersect = minSize - slope * vwMin

font-size: clamp(minSize rem, intersect rem + slope * 100 vw, maxSize rem);
```

The result is a straight line between your two anchor points, flat outside them. At the min viewport it equals your mobile size exactly. At the max viewport it equals your desktop size exactly.

**Round sizes to whole pixels** snaps every value to your root size so you get 2.5rem instead of 2.4931rem. Turn it off if you want the ratio respected exactly.

## Suggestions it makes

**Mobile size**, when Suggest is on:

```
mobile = body + (desktop - body) * (1 - shrink)
```

**Line-height**, by size band. Larger type gets tighter leading:

| Desktop size | Line-height |
| --- | --- |
| under 1.2rem | 1.6 |
| under 1.6rem | 1.45 |
| under 2.2rem | 1.3 |
| under 3rem | 1.2 |
| under 4.2rem | 1.1 |
| 4.2rem and up | 1.05 |

**Letter-spacing**, negative tracking on large sizes only: 0 below 1.5rem, then -0.01em, -0.02em, and -0.03em above 4rem.

Both are off by default. Turn on **Add line-height and letter-spacing** in Extras to include them in the output.

## Warnings

Caliper flags three things:

- **Inverted viewport range.** Min is not smaller than max, so the maths breaks down.
- **Mobile size larger than desktop.** The text shrinks as the screen grows. CSS locks it to the mobile value. Fine if you meant it.
- **May not survive browser zoom.** If the rem part of the preferred value is zero or negative, the text can stop growing at 200% zoom, which fails WCAG 1.4.4. Usually caused by a mobile size that is too small for the viewport range. Raise the mobile size or narrow the range.

## Settings reference

| Setting | Default | Notes |
| --- | --- | --- |
| Units | rem | Switches every size and viewport field between rem and px |
| Root font size | 16px | Used for every rem to px conversion |
| Body size | 1rem | The floor for the mobile suggestion, and the anchor for body-up scales |
| Viewport min | 320px | Below this, the size is locked to the mobile value |
| Viewport max | 1440px | Above this, the size is locked to the desktop value |
| Round to whole pixels | on | Snaps sizes to your root size |
| Desktop ratio | 1.25 | Major third |
| Mobile ratio | 1.125 | Major second |
| Levels | H1 to H6 | Down to H1 to H3 |

Ratios available: 1.067, 1.125, 1.2, 1.25, 1.333, 1.414, 1.5, 1.618. For a six level scale, 1.2 and 1.25 are the practical range. Anything above 1.333 pushes H6 below body text.

## Making it yours

**Rename it.** Change the `BRAND` constant on the first line of the script. It updates the page title, the header, and the footer.

**Recolor it.** Every colour is a custom property in the `:root` block at the top of the stylesheet. `--accent` drives the chart curve, the logo, and every active state.

**The logo** is the clamp curve itself, flat then ramping then flat. It is inline SVG in three places: the favicon, the Apple touch icon, and the header. All three use the same path, `M5 22h5l12-12h5`.

## Browser support

The `clamp()` output works in every browser that supports `clamp()`, which is all of them since 2020.

The fluid formula divides by a `var()`. Browsers resolve that at computed value time, which modern Chrome, Safari, Firefox and Edge all handle. If you need to support something older, use the plain `clamp()` output instead.

Keyboard accessible throughout, with visible focus rings. Respects `prefers-reduced-motion`.

## Not built yet

Ideas worth adding:

- Saved presets, so your standard viewport range loads on open
- A spacing scale on the same maths, since fluid padding has the same problem
- Paste an existing `clamp()` and have it reverse engineer the min, max, and viewport range
- A column for your own variable names so the CSS block matches project naming

## License

MIT. Copyright (c) 2026 Rumit Vaghasiya.
