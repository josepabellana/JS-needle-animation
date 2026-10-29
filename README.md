# JS Needle Animation

A hand-sewing simulation built entirely with vanilla JavaScript and SVG — no canvas, no WebGL, no dependencies.

**[▶ View live demo](https://josepabellana.github.io/JS-needle-animation/)**

> The animation is JavaScript-driven and can't be embedded as a static image.
> **[Open the live demo](https://josepabellana.github.io/JS-needle-animation/)** to see it run.
> *(Enable GitHub Pages first — see the section at the bottom.)*

---

## What it does

A needle sews a running stitch across a woven linen surface. Every frame is computed in JavaScript and written directly to SVG path `d` attributes via `requestAnimationFrame` — no CSS keyframes, no GSAP.

- **Needle** passes under the cloth between the two holes (SVG mask)
- **Thread** forms a realistic arc in the air, pulled taut as the stitch closes
- **Loose tail** through the needle eye moves with gravity as the needle tilts
- **Completed stitches** build up to the left with a 4-layer tube rendering (dark core → mid → highlight → sheen) and fade out at the left edge
- **Shadow** is projected flat onto the cloth surface and fades as the needle lifts
- Pauses automatically when the tab is hidden or `prefers-reduced-motion` is set

## How it works

All motion is driven by monotone cubic spline interpolation over manually keyframed data points (angle, x, y for the needle; bow amount for the thread arcs). The spline resamples those sparse keyframes into 108 evenly-spaced frames per stitch cycle (1.8 s).

```js
// Sparse keyframes: [frame, angle°, butt-x, butt-y]
const NEEDLE_KEYS = [
  [0, 12.0, 150, 229],
  [30,  1.0, 194, 309],  // needle enters cloth
  [54,  0.0, 470, 314],  // needle fully through
  [60, -8.7, 508, 306],  // pulling thread through
  // ...
];
```

The thread arc uses a simple `bowed()` quadratic helper:

```js
function bowed(x0, y0, x1, y1, h) {
  const dx = x1-x0, dy = y1-y0, len = Math.sqrt(dx*dx+dy*dy) || 1;
  const k = (2 * h * Math.min(1, len/70)) / len;
  return `M${x0},${y0} Q${(x0+x1)/2 + dy*k},${(y0+y1)/2 - dx*k} ${x1},${y1}`;
}
```

## Usage

### Standalone page

Just open `index.html` — no build step, no npm.

### As a loading screen

Drop the SVG block and script into any HTML page:

```html
<!-- 1. Add the overlay div + SVG (see index.html) -->
<div id="loader"> ... </div>

<!-- 2. Run the animation, then dismiss on window load -->
<script>
  // Start rAF loop → on window load, add .done class → remove from DOM
  window.addEventListener('load', () => {
    document.getElementById('loader').classList.add('done');
  });
</script>
```

```css
#loader {
  position: fixed; inset: 0; z-index: 9999;
  transition: opacity .7s ease, visibility .7s ease;
}
#loader.done { opacity: 0; visibility: hidden; pointer-events: none; }
```

## Enable live demo (GitHub Pages)

1. Go to **Settings → Pages** in this repository
2. Set source to **Deploy from branch → main → / (root)**
3. The demo will be live at `https://josepabellana.github.io/JS-needle-animation/`
