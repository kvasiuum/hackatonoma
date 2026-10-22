# Motion for React — API reference (v14)

`import { ... } from "motion/react"`. Any HTML/SVG tag works as `motion.<tag>`. For custom components: `const MotionCard = motion.create(Card)` (the component must forward `ref` to a DOM element).

## Core props

| Prop | Purpose |
|---|---|
| `initial` | Start state. `initial={false}` skips the mount animation and starts at `animate`. |
| `animate` | Target state; animates whenever it changes. |
| `exit` | State to animate to before unmount (needs `AnimatePresence`). |
| `transition` | Default transition. Per-value: `transition={{ default: {...}, opacity: { duration: 0.2 } }}`. Also allowed inside any target. |
| `variants` | Named states, referenced by label in `initial`/`animate`/`while*`. Propagate to children. |
| `custom` | Data passed to function variants: `variants={{ show: (i) => ({ transition: { delay: i * 0.05 } }) }}`. |
| `style` | Accepts motion values: `style={{ x, opacity }}`. Supports independent transforms `x, y, z, scale, scaleX, rotate, rotateX, skew, originX`. |
| `whileHover`, `whileTap`, `whileFocus`, `whileDrag`, `whileInView` | Gesture states; revert when the gesture ends. |
| `viewport` | For `whileInView`: `{ once: true, amount: 0.3, margin: "0px 0px -10% 0px", root: ref }`. |
| `layout` | Animate size/position changes. `"position"`, `"size"`, or `true`. |
| `layoutId` | Shared element: different components with the same id animate between each other. |
| `layoutDependency` | Only measure layout when this value changes (perf). |
| `layoutScroll`, `layoutRoot` | Mark scrollable containers / fixed roots so layout measures correctly. |
| `onAnimationStart`, `onAnimationComplete`, `onUpdate` | Lifecycle callbacks. |
| `onHoverStart/End`, `onTap`, `onTapStart`, `onTapCancel`, `onPan*`, `onDrag*`, `onViewportEnter/Leave` | Gesture callbacks. |

Keyframes: `animate={{ x: [0, 100, 0] }}`, with `times: [0, 0.3, 1]`. `null` as first keyframe means "current value": `x: [null, 100]`.

## Transitions

```js
{ type: "spring", visualDuration: 0.4, bounce: 0.25 }   // preferred spring form
{ type: "spring", stiffness: 400, damping: 30, mass: 1 } // physics form (overrides bounce/duration)
{ type: "spring", bounce: -0.3 }                          // negative bounce = overdamped, extra soft (v13.5+)
{ duration: 0.3, ease: "easeOut" }                       // tween
{ ease: [0.22, 1, 0.36, 1], duration: 0.6 }              // cubic-bezier
{ repeat: Infinity, repeatType: "reverse" | "loop" | "mirror", repeatDelay: 1 }
{ delay: 0.2 }
{ type: "inertia", velocity: 300, power: 0.8, min, max }  // decelerate (drag release)
{ delayChildren: stagger(0.05, { from: "center" }) }      // orchestrate children (variants only)
{ when: "beforeChildren" | "afterChildren" }
{ path: arc({ strength: 0.6, rotate: true }) }            // move x/y along a curve (see below)
```

`staggerChildren` and `staggerDirection` are **deprecated**; use `delayChildren: stagger(interval, { from: "last" })`.

Defaults when no transition is set: physical values (`x`, `y`, `scale`, `rotate`) use a spring; `opacity`/colors use a tween.

Global defaults: `<MotionConfig transition={{ type: "spring", visualDuration: 0.3, bounce: 0.15 }} reducedMotion="user">`.

## `stagger(duration, options)`

```js
stagger(0.05)                                   // 0, 0.05, 0.1...
stagger(0.05, { startDelay: 0.2 })
stagger(0.05, { from: "first" | "last" | "center" | "edges" | "random" | index })
stagger(0.04, { grid: [columns, rows], from: "center" }) // 2D ripple by distance (v14.3+)
stagger(0.1, { ease: "easeOut" })               // distribute delays along an easing curve
```

## AnimatePresence

```jsx
<AnimatePresence mode="sync" | "wait" | "popLayout" initial={false} onExitComplete={fn} custom={direction}>
  {show && <motion.div key="unique" exit={{ opacity: 0 }} />}
</AnimatePresence>
```

- Direct children must have a stable, unique `key`. Changing the key = exit old + enter new (use for slideshows: `key={index}`).
- `mode="wait"`: old exits before new enters (page transitions, swapping content).
- `mode="popLayout"`: exiting element is popped out of layout immediately so siblings animate with `layout`. Good for lists and toasts.
- `custom` on `AnimatePresence` updates the value seen by exiting children's function variants (direction-aware sliders).
- `propagate` prop lets nested `AnimatePresence` children run exits when the parent is removed.
- Hooks inside children: `useIsPresent()`, `usePresence()` → `[isPresent, safeToRemove]`, `usePresenceData()`.

## Layout animations

```jsx
<motion.div layout transition={{ type: "spring", visualDuration: 0.3, bounce: 0.2 }} />
```

- Change layout through React state (class, style, order, conditional render); Motion animates from old to new box with transforms.
- Children of a `layout` element that look stretched: give them `layout` too (or `layout="position"`).
- Keep `borderRadius` and `boxShadow` in `style` (not CSS classes) so Motion can correct their distortion during scale.
- `layoutId`: tabs underline, card → expanded card, image lightbox. Wrap separate trees in `<LayoutGroup>` when siblings with `layout` should know about each other.
- Within `AnimatePresence`, a `layoutId` element exiting and another entering will crossfade/morph.

## Gestures

```jsx
<motion.button whileHover={{ scale: 1.03 }} whileTap={{ scale: 0.97 }} whileFocus={{ boxShadow: "0 0 0 3px #6366f155" }} />
```

- `whileTap` is keyboard accessible (Enter on focused element). `onTap` fires on successful release inside.
- Drag:

```jsx
<motion.div
  drag            // or "x" | "y"
  dragConstraints={containerRef /* or { left: 0, right: 300, top: 0, bottom: 0 } */}
  dragElastic={0.2}
  dragMomentum={true}
  dragTransition={{ bounceStiffness: 400, bounceDamping: 25, power: 0.3 }}
  dragSnapToOrigin
  dragDirectionLock
  whileDrag={{ scale: 1.05, cursor: "grabbing" }}
  onDragEnd={(e, info) => info.offset.x > 100 && dismiss() /* info.velocity, info.point */}
/>
```

- `useDragControls()` + `dragListener={false}` to start drag from a handle: `onPointerDown={(e) => controls.start(e)}`.

## Reorder

```jsx
<Reorder.Group axis="y" values={items} onReorder={setItems}>
  {items.map((item) => (
    <Reorder.Item key={item.id} value={item}>{item.label}</Reorder.Item>
  ))}
</Reorder.Group>
```

v13.1+: multidimensional (grids, wrap) with automatic axis detection and RTL. v14.5: `onReorder(newOrder, moved)`. v14.6: keyboard grab-and-move reordering. Auto-scrolls the nearest scroll container.

## Motion values and hooks

Motion values update outside React render — no re-renders, ideal for scroll and pointer effects.

```js
const x = useMotionValue(0)                      // x.get(), x.set(v), x.on("change", cb)
const opacity = useTransform(x, [-200, 0, 200], [0, 1, 0])        // map ranges (clamped by default)
const bg = useTransform(x, [-200, 200], ["#f43f5e", "#22c55e"])   // colors, units, complex strings work
const rotate = useTransform(() => x.get() / 10)                   // function form, auto-tracks dependencies
const smooth = useSpring(x, { stiffness: 300, damping: 30 })      // spring-follow a value or number
const follow = useFollowValue(x, { type: "tween", duration: 0.3 })// follow with any transition type
const vel = useVelocity(x)
const t = useTime()                                // ms since mount; e.g. useTransform(t, [0, 4000], [0, 360], { clamp: false })
const shadow = useMotionTemplate`0 ${y}px ${blur}px rgba(0,0,0,.2)`
useMotionValueEvent(x, "change", (v) => {})
useAnimationFrame((time, delta) => {})
```

Scroll:

```js
const { scrollY, scrollYProgress } = useScroll()                       // page
const { scrollYProgress } = useScroll({ container: ref })              // element scroll
const { scrollYProgress } = useScroll({ target: ref, offset: ["start end", "end start"] }) // element passing through viewport
```

Offsets: `"<target edge> <container edge>"`, edges `start | center | end | 0–1 | px | %`. Common: `["start end", "end start"]` (whole pass), `["start start", "end start"]` (sticky section), `["start 0.9", "start 0.4"]` (reveal window). Many offsets are hardware-accelerated via `ViewTimeline` automatically.

Other: `useInView(ref, { once, amount, margin })` → boolean. `usePageInView()` → tab visibility. `useReducedMotion()` → boolean.

## Imperative: `useAnimate`

```jsx
const [scope, animate] = useAnimate()
useEffect(() => {
  animate([
    ["h1", { opacity: [0, 1], y: [20, 0] }, { duration: 0.5 }],
    ["p", { opacity: [0, 1] }, { at: "-0.2" }],
    ["li", { opacity: [0, 1], x: [-10, 0] }, { delay: stagger(0.05), at: "<" }],
  ])
}, [])
return <section ref={scope}>...</section>
```

Selectors are scoped to `scope`. Animations are cleaned up on unmount. Returns controls: `.play() .pause() .stop() .complete() .cancel() .speed .time .then()`.

Sequence `at`: number (absolute seconds), `"+0.2"` / `"-0.2"` (relative to previous end), `"<"` (same start as previous), `"label"` (named point: add `"label"` string as a sequence item).

## `useScopeMotion` (v14.5+)

Runs vanilla APIs (`animate`, `inView`, `scroll`, `hover`, `press`, `drag`) inside a scope rooted at `ref`. Everything started inside is stopped and inline styles restored on unmount or when `deps` change. Supports a default transition and responsive `matchMedia`.

```jsx
const ref = useRef(null)
useScopeMotion(ref, ({ media }) => {
  inView("li", (el) => animate(el, { opacity: 1, y: 0 }))
  hover(".card", (el) => {
    animate(el, { y: -4 })
    return () => animate(el, { y: 0 })
  })
}, {
  transition: { type: "spring", bounce: 0.2 },
  matchMedia: { desktop: ">=800" },  // media.desktop is boolean; setup re-runs on change
  deps: [],
})
```

## AnimateView (React ≥ 19.3, `motion/react-animate-view`)

Built on React's `<ViewTransition>`. Animates enter, exit, update and shared (same `name`) transitions triggered by `startTransition`, `useDeferredValue` or Suspense.

```jsx
<AnimateView
  name="hero-image"                          // shared across routes → morph
  transition={{ type: "spring", visualDuration: 0.4, bounce: 0.1 }}
  enter={{ opacity: [0, 1], y: [20, 0] }}
  exit={{ opacity: 0 }}
  share={(types) => ({ /* types = transition types */ })}
  onAnimationComplete={(type) => {}}
>
  <img src={src} />
</AnimateView>
```

Throws on React < 19.3; use `AnimatePresence` + `layoutId` there.

## Bundle size

`<m.div>` + `<LazyMotion features={domAnimation}>` (or `domMax` for layout/drag) cuts the initial bundle. Use `LazyMotion strict` to catch accidental `motion.` usage. `motion/react-mini` exports a tiny `useAnimate` (WAAPI only).
