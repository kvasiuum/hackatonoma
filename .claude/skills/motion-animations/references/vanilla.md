# Motion for vanilla JS — API reference (v14)

`import { animate, scroll, inView, hover, press, drag, stagger, animateLayout, animateView, scopeMotion, spring } from "motion"`

Works with any framework (Vue, Svelte, Astro, plain HTML). Selectors, elements, NodeLists and arrays are all accepted where `el` appears.

## `animate`

```js
// Elements
const controls = animate(".box", { x: 100, opacity: 1, rotate: 45 }, { duration: 0.5 })
animate(el, { backgroundColor: "#22c55e", "--accent": "#f00" })   // colors and CSS variables
animate(el, { x: [0, 100], scale: [1, 1.2, 1] }, { times: [0, 0.5, 1] })
animate(el, { clipPath: ["inset(0 100% 0 0)", "inset(0 0% 0 0)"] })

// Springs
animate(el, { y: 0 }, { type: "spring", visualDuration: 0.4, bounce: 0.25 })
animate(el, { y: 0 }, { type: spring, stiffness: 300, damping: 20 })

// Stagger across matched elements
animate("li", { opacity: [0, 1], y: [12, 0] }, { delay: stagger(0.05) })
animate(".tile", { scale: [0.8, 1], opacity: [0, 1] }, { delay: stagger(0.03, { grid: [6, 4], from: "center" }) })

// Plain values and objects
animate(0, 100, { duration: 1, onUpdate: (v) => (counter.textContent = Math.round(v)) })
animate(motionValue, 200)
animate(objectOrThreeMesh, { x: 10 })

// SVG path drawing (pathLength 0–1)
animate("path", { pathLength: [0, 1] }, { duration: 1.2, ease: "easeInOut" })
```

Controls: `.then()` / `await`, `.play()`, `.pause()`, `.stop()`, `.cancel()`, `.complete()`, `.time`, `.speed`, `.duration`.

Options: `duration`, `delay`, `ease`, `type`, `repeat`, `repeatType`, `repeatDelay`, `times`, `onUpdate`, `onComplete`, `path`, plus per-value overrides: `{ default: {...}, opacity: { duration: 0.2 } }`.

Hardware acceleration: `opacity`, `transform`, `filter`, `clipPath`, and (v14.2+) independent transforms `x`/`y`/`scale` and (v12.43+) `backgroundColor` run on the compositor where the browser supports it — smooth even when the main thread is busy.

## Sequences

```js
animate([
  [".hero h1", { opacity: [0, 1], y: [30, 0] }, { duration: 0.6, ease: [0.22, 1, 0.36, 1] }],
  "afterTitle",
  [".hero p", { opacity: [0, 1] }, { at: "-0.3" }],
  [".hero .btn", { scale: [0.9, 1], opacity: [0, 1] }, { at: "afterTitle", type: "spring", bounce: 0.4 }],
  [".hero li", { opacity: [0, 1], x: [-12, 0] }, { delay: stagger(0.06), at: "<" }],
], { defaultTransition: { duration: 0.4 } })
```

## `stagger`

`stagger(duration, { startDelay, from: "first" | "last" | "center" | "edges" | "random" | index, ease, grid: [cols, rows] })`

## `arc` — curved motion paths (v12.40+)

```js
import { animate, arc } from "motion"
animate(el, { x: 400, y: -100 }, {
  duration: 0.8,
  path: arc({ strength: 0.6, peak: 0.5, direction: "cw", rotate: true }),
})
```

- `strength`: bulge as a fraction of distance (default 0.5).
- `peak`: where along the path (0–1) the arc is highest.
- `direction`: `"cw" | "ccw"`; unset picks a stable side.
- `rotate`: `true` or 0–1 — tilt along the tangent.
Works with springs and in React via `transition={{ path: arc() }}`.

## `scroll`

```js
// Scroll-linked animation (hardware accelerated where possible)
scroll(animate(".progress", { scaleX: [0, 1] }, { ease: "linear" }))

// Element passing through viewport
scroll(animate(img, { y: [-60, 60] }, { ease: "linear" }), {
  target: section,
  offset: ["start end", "end start"],
})

// Callback form
scroll((progress, info) => { /* info.y.current, info.y.velocity */ }, { container: el, axis: "y" })
```

Returns a cleanup function.

## `inView`

```js
const stop = inView(".reveal", (el, entry) => {
  animate(el, { opacity: 1, y: 0 })
  return () => animate(el, { opacity: 0, y: 20 })   // optional: on leave
}, { amount: 0.3, margin: "0px 0px -10% 0px", root })
```

Without the returned function it fires once per element.

## Gestures

```js
hover(".card", (el, startEvent) => {
  const a = animate(el, { y: -4, boxShadow: "0 12px 30px rgba(0,0,0,.15)" })
  return (endEvent) => animate(el, { y: 0, boxShadow: "0 2px 8px rgba(0,0,0,.08)" })
})

press("button", (el, startEvent) => {
  animate(el, { scale: 0.96 })
  return (endEvent, { success }) => animate(el, { scale: 1 })
})
```

Both filter out emulated touch hovers, are keyboard accessible (`press` via Enter), and return a cleanup function.

### `drag` (v14.4+)

```js
const stop = drag(".sheet", {
  axis: "y",
  constraints: { top: 0, bottom: 400 },     // or a container Element
  elastic: 0.2,                              // 0–1, per edge allowed
  momentum: true,
  transition: { power: 0.3, timeConstant: 200 },
  snapToOrigin: false,                       // true | "x" | "y"
  directionLock: false,
  onStart: (e, info) => {},
  onMove: (e, info) => {},
  onEnd: (e, info) => { if (info.velocity.y > 500) close() },
})
```

## `animateLayout` (v14.1+)

FLIP layout animations without React. Mark elements with `data-layout` (size/position) or `data-layout-id="name"` (shared element moving between parents).

```js
await animateLayout(".container", () => {
  // mutate the DOM here: reorder, toggle classes, move an indicator
  list.prepend(newItem)
}, { type: "spring", visualDuration: 0.35, bounce: 0.15 })
```

Signature: `animateLayout(scopeOrUpdate, updateOrOptions?, options?)`. Returns a builder that is awaitable.

## `animateView` — View Transitions API with Motion transitions

```js
await animateView(() => {
  // update the DOM (sync or async); e.g. swap page content
  main.innerHTML = nextPageHtml
}, { type: "spring", visualDuration: 0.4, bounce: 0.1 })
  .add(".card.active", ".modal")          // morph one element into another (shared element)
  .layout({ type: "spring", bounce: 0.2 }) // transition for size/position of current subject
  .add(".title").crop(false)              // text: don't crop
  .enter({ opacity: [0, 1], y: [20, 0] }) // layers only in the new snapshot
  .exit({ opacity: [1, 0] })              // layers only in the old snapshot
```

Builder methods apply to the most recently `.add()`ed subject: `.add(subject, newSubject?)`, `.layout(options)`, `.enter(kf, opts)`, `.exit(kf, opts)`, `.new(kf, opts)`, `.old(kf, opts)`, `.crop(bool)`, `.group(bool)`, `.class(name)`. `view-transition-name` is generated and removed automatically. Group layers auto-crop and match DOM hierarchy. Falls back to an instant update where View Transitions are unsupported.

## `scopeMotion` (v14.5+)

Collect animations, scroll/inView listeners, gestures and resize observers so they can be torn down together — essential in SPA components.

```js
const scope = scopeMotion(".page", ({ media }) => {
  animate("h1", { x: media.desktop ? 100 : 0 })
  inView(".reveal", (el) => animate(el, { opacity: 1 }))
}, {
  transition: { duration: 0.5 },              // default for every animate() inside
  matchMedia: { desktop: ">=800" },          // re-runs setup when conditions change
})

scope.stop()     // stop everything, leave styles
scope.cancel()   // stop and restore inline styles
scope.refresh()  // cancel, then re-run setup
button.onclick = scope.bind(() => animate(".x", { rotate: 90 }))  // collected into scope
```

## Motion values outside React

```js
import { motionValue, springValue, transformValue, styleEffect, mapValue } from "motion"
const x = motionValue(0)
const smoothX = springValue(x, { stiffness: 300, damping: 30 })
const rotate = transformValue(() => smoothX.get() / 10)
styleEffect(el, { x: smoothX, rotate })     // binds values to styles, batched per frame
window.addEventListener("pointermove", (e) => x.set(e.clientX))
```

`attrEffect`, `svgEffect`, `propEffect` bind to attributes, SVG and object properties. `motion/three` exports `threeEffect` for Three.js objects and uniforms.

## Frameloop

`frame.read(cb)`, `frame.update(cb)`, `frame.render(cb)`, `cancelFrame(cb)` batch DOM reads/writes to avoid layout thrashing. Pass `true` as the second argument to keep the callback running every frame.
