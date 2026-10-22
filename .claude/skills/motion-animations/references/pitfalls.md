# Pitfalls and fixes

## AnimatePresence

- **Exit doesn't play.** The conditional must be a *direct child* of `AnimatePresence`, with a stable `key`. `AnimatePresence` itself must stay mounted (don't put it inside the conditional).
- **Using array index as key** in a list breaks exits and reorders. Use stable ids.
- **Swapping content without exit**: change the `key` (e.g. `key={step}`) so React remounts.
- **`mode="popLayout"`** requires the child to forward `ref` to a DOM node (custom components: use `forwardRef` or React 19 ref prop).
- **`mode="wait"`** with long exits feels laggy; keep exits ≤ 0.2 s.

## Layout animations

- **Content stretches/squishes during layout animation.** Add `layout` (or `layout="position"`) to the child elements, or wrap text in an element with `layout="position"`.
- **Border radius or shadow distorts.** Set them in the `style` prop, not via CSS classes.
- **Element jumps when page is scrolled or inside a scroll container.** Add `layoutScroll` to the scrollable ancestor; `layoutRoot` to `position: fixed` containers.
- **Unrelated siblings don't animate.** Wrap them in `<LayoutGroup>`.
- **Expensive re-measures.** Use `layoutDependency={value}` so measurement only happens when it changes.
- **Animating `width`/`height` directly** is janky; toggle the layout and use `layout` instead. Exception: `height: "auto"` accordions (recipe 15) are fine.

## Performance

- Animate `transform`, `opacity`, `filter`, `clipPath`. Avoid `top/left/width/height/margin/box-shadow` on many elements at once.
- For pointer/scroll-driven values use motion values (`useMotionValue`, `useTransform`, `useScroll`), never React state — state causes a re-render every frame.
- Large `filter: blur()` on big surfaces is costly; keep blur radius small or limit to small elements.
- Loops off-screen still cost; gate with `useInView` / `usePageInView`.
- `will-change` is added automatically for animated transforms; don't sprinkle it manually on everything.

## React specifics

- **Server Components / Next.js App Router**: files using `motion/react` need `"use client"`, or import `* as motion from "motion/react-client"` for simple motion elements in server files.
- **Hydration flash**: set `initial={false}` when the element should render in its final state on first paint (e.g. remembered UI state).
- **`motion.create(Component)`** must be called outside render, once. The component must forward `ref`.
- **Custom props leaking to DOM** (styled-components etc.): since v13 the `@emotion/is-prop-valid` auto-detection is removed; pass `<MotionConfig isValidProp={isPropValid}>` explicitly.
- **`staggerChildren` deprecated**: use `delayChildren: stagger(0.05)`. Variants must be used on children (labels propagate); setting `animate` on a child with an object breaks the orchestration.
- **`whileInView` re-triggers every time** unless `viewport={{ once: true }}`.
- **Strict Mode double effects** with `useAnimate`: return cleanup (`return () => controls.stop()`) or use `useScopeMotion`, which cleans up automatically.
- **`AnimateView` throws** on React < 19.3.

## Vanilla specifics

- In SPAs, wrap setup in `scopeMotion()` and call `scope.cancel()` on teardown; otherwise `inView`, `scroll`, `hover` listeners leak.
- `animate(selector)` only matches elements that exist at call time. For later elements, call again or use `inView`.
- `scroll()` with an `animate` controls object needs `ease: "linear"` on that animation, or the scroll mapping looks wrong.
- Transforms set by `animate` live in inline `style`; CSS classes that also set `transform` will conflict. Use independent `x`, `y`, `scale` values.

## Design smells

- Everything bouncing: limit bounce to playful elements.
- Durations > 0.6 s on UI controls: users wait for the interface.
- Enter animations on every scroll-in of long lists: use `once: true` and cap stagger.
- Animating on every hover of dense lists: keep it to color/opacity, not movement.
- No reduced-motion path for parallax, loops, large movements.
