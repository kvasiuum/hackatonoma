---
name: motion-animations
description: Build beautiful, polished web animations with the Motion library (motion.dev, formerly Framer Motion) in React or vanilla JS. Use when the user asks for animations, transitions, micro-interactions, hover/tap effects, scroll effects, page or modal transitions, staggered lists, layout animations, drag, or wants a UI to "feel alive" or "look smooth".
---

# Motion animations

Motion is the animation library at motion.dev (npm `motion`, formerly `framer-motion`). This skill targets **Motion 14.x** (October 2026). APIs were verified against the source; prefer them over older blog posts and memory.

## Imports

```bash
npm install motion
```

| Context | Import |
|---|---|
| React | `import { motion, AnimatePresence, stagger } from "motion/react"` |
| React Server Components file | `import * as motion from "motion/react-client"` |
| Vanilla JS / any framework | `import { animate, scroll, inView, hover, press, stagger } from "motion"` |
| Smallest vanilla bundle (WAAPI only) | `import { animate } from "motion/mini"` |
| React View Transitions (React ≥ 19.3) | `import { AnimateView } from "motion/react-animate-view"` |
| No bundler | `<script src="https://cdn.jsdelivr.net/npm/motion@14/dist/motion.js"></script>` → `const { animate } = Motion` |

Never import from `framer-motion` in new code; `motion/react` is the same thing.

## Pick the right tool

| Effect wanted | React | Vanilla |
|---|---|---|
| Animate on mount / prop change | `<motion.div initial animate>` | `animate(el, {...})` |
| Exit animation on unmount | `<AnimatePresence>` + `exit` | animate, then remove in `.then()` |
| Hover / tap / focus feedback | `whileHover`, `whileTap`, `whileFocus` | `hover()`, `press()` |
| Reveal on scroll into view | `whileInView` + `viewport={{ once: true }}` | `inView()` |
| Scroll-linked (progress bar, parallax) | `useScroll` + `useTransform` | `scroll(animation)` |
| Element changes size/position | `layout` prop | `animateLayout()` + `data-layout` |
| Element moves between places (tabs underline, card → modal) | `layoutId` | `data-layout-id` or `animateView().add(a, b)` |
| Staggered list/grid | `delayChildren: stagger(0.05)` in variants | `delay: stagger(0.05, { grid })` |
| Multi-step choreography | `useAnimate` + sequence array | `animate([...sequence])` |
| Follow the cursor smoothly | `useSpring` / `useFollowValue` | `springValue()` |
| Drag, swipe, reorder | `drag` prop, `<Reorder>` | `drag()` |
| Curved flight path (to cart, etc.) | `transition={{ path: arc() }}` | same in `animate` options |
| Page / route transitions | `AnimatePresence mode="wait"` or `AnimateView` | `animateView()` |
| Clean up everything in a component | `useScopeMotion(ref, setup)` | `scopeMotion(root, setup)` |

Detailed API: `references/react.md` and `references/vanilla.md`. Ready-made effects: `references/recipes.md`. Bugs and gotchas: `references/pitfalls.md`. Read the relevant file before writing non-trivial code.

## What makes an animation look good

Follow these defaults unless the user's design calls for something else.

1. **Springs for anything physical.** Movement, scale, drag release and layout should use a spring. Prefer the designer-friendly form: `{ type: "spring", visualDuration: 0.35, bounce: 0.2 }`. Bounce 0–0.15 for UI chrome, 0.2–0.35 for playful elements, never above 0.5 in product UI.
2. **Tweens for opacity and color.** `{ duration: 0.2, ease: "easeOut" }`. Opacity bouncing looks broken.
3. **Short.** Micro-interactions 0.1–0.2 s, element enter 0.25–0.45 s, page/modal 0.3–0.5 s. Exits ~30% faster than enters.
4. **Small distances.** Enter from `y: 8–24`, `scale: 0.95–0.98`, not from off-screen. Combine with `opacity: 0` and optionally `filter: "blur(4px)"` for a premium feel.
5. **Animate transforms and opacity.** `x`, `y`, `scale`, `rotate`, `opacity`, `clipPath`, `filter` are cheap. Avoid animating `width`, `height`, `top`, `left`, `margin`; use `layout` instead.
6. **Stagger, but tightly.** 0.03–0.08 s between items; cap total stagger around 0.5 s so long lists don't crawl. Use `stagger(0.04, { from: "center" })` or `grid` for grids.
7. **Every interactive element gets feedback.** `whileHover={{ y: -2 }}` / `scale: 1.02` and `whileTap={{ scale: 0.97 }}`. Subtle beats dramatic.
8. **Continuity.** When something moves from A to B, use `layoutId` so it travels instead of disappearing and reappearing.
9. **Respect reduced motion.** Wrap the app in `<MotionConfig reducedMotion="user">`, or branch on `useReducedMotion()` to swap movement for a fade.
10. **One hero animation per screen.** Let one thing be expressive; keep the rest quiet.

Easing cheat sheet: `"easeOut"` for enters, `"easeIn"` for exits, `[0.22, 1, 0.36, 1]` (expo-out-ish) for elegant reveals, `"linear"` only for scroll-linked or infinite loops.

## Minimal patterns

```jsx
// Enter + exit
<AnimatePresence>
  {open && (
    <motion.div
      key="panel"
      initial={{ opacity: 0, y: 12, filter: "blur(4px)" }}
      animate={{ opacity: 1, y: 0, filter: "blur(0px)" }}
      exit={{ opacity: 0, y: 8, transition: { duration: 0.15 } }}
      transition={{ type: "spring", visualDuration: 0.35, bounce: 0.15 }}
    />
  )}
</AnimatePresence>

// Staggered list
const list = { show: { transition: { delayChildren: stagger(0.05) } } }
const item = { hidden: { opacity: 0, y: 16 }, show: { opacity: 1, y: 0 } }
<motion.ul variants={list} initial="hidden" animate="show">
  {items.map((i) => <motion.li key={i.id} variants={item} />)}
</motion.ul>
```

```js
// Vanilla: reveal cards as they scroll in
inView(".card", (el) => {
  animate(el, { opacity: [0, 1], y: [24, 0] }, { type: "spring", bounce: 0.15 })
}, { amount: 0.3 })
```

## Workflow

1. Identify React vs vanilla from the user's project (look for `package.json`, `.jsx/.tsx`).
2. Choose the tool from the table above; read the matching reference.
3. Start from a recipe in `references/recipes.md` when one fits; adapt values to the design.
4. Apply the taste rules; check `references/pitfalls.md` for the APIs used (especially `AnimatePresence` keys, `layout` distortion, and SSR).
5. Add reduced-motion handling for anything that moves more than a few pixels or loops.
