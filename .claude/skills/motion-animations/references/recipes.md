# Recipes — polished effects, ready to adapt

React unless marked **Vanilla**. Imports assume `motion/react` (React) or `motion` (vanilla). Tune values to the design; keep the taste rules from SKILL.md.

Shared presets used below:

```js
export const spring = { type: "spring", visualDuration: 0.4, bounce: 0.15 }
export const snappy = { type: "spring", visualDuration: 0.25, bounce: 0.1 }
export const bouncy = { type: "spring", visualDuration: 0.5, bounce: 0.35 }
export const elegant = { duration: 0.7, ease: [0.22, 1, 0.36, 1] }
```

---

## 1. Hero entrance (blur-up cascade)

```jsx
const container = { show: { transition: { delayChildren: stagger(0.08, { startDelay: 0.1 }) } } }
const item = {
  hidden: { opacity: 0, y: 20, filter: "blur(8px)" },
  show: { opacity: 1, y: 0, filter: "blur(0px)", transition: elegant },
}

<motion.section variants={container} initial="hidden" animate="show">
  <motion.p variants={item} className="eyebrow">New release</motion.p>
  <motion.h1 variants={item}>Ship faster</motion.h1>
  <motion.p variants={item}>Subtitle…</motion.p>
  <motion.div variants={item}><Button /></motion.div>
</motion.section>
```

## 2. Split-text reveal (words rising through a mask)

```jsx
function RevealText({ text }) {
  const words = text.split(" ")
  return (
    <motion.h1 initial="hidden" whileInView="show" viewport={{ once: true }}
      transition={{ delayChildren: stagger(0.05) }} aria-label={text}>
      {words.map((w, i) => (
        <span key={i} aria-hidden style={{ display: "inline-block", overflow: "hidden", verticalAlign: "top" }}>
          <motion.span style={{ display: "inline-block" }}
            variants={{ hidden: { y: "110%" }, show: { y: "0%", transition: elegant } }}>
            {w}&nbsp;
          </motion.span>
        </span>
      ))}
    </motion.h1>
  )
}
```

## 3. Scroll reveal for sections and cards

```jsx
<motion.div
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, amount: 0.25 }}
  transition={spring}
/>
```

**Vanilla:**

```js
inView(".reveal", (el) => {
  animate(el, { opacity: [0, 1], y: [40, 0] }, { type: "spring", visualDuration: 0.5, bounce: 0.1 })
}, { amount: 0.25 })
```

## 4. Grid ripple (staggered from center)

```js
// Vanilla, or inside useScopeMotion in React
const cols = 6
animate(".tile",
  { opacity: [0, 1], scale: [0.6, 1] },
  { delay: stagger(0.04, { grid: [cols, Math.ceil(tiles.length / cols)], from: "center" }), type: "spring", bounce: 0.3 })
```

React variant: put `delayChildren: stagger(0.04, { grid: [6, 4], from: "center" })` on the parent's `show` transition.

## 5. Card hover lift with tilt

```jsx
function TiltCard({ children }) {
  const x = useMotionValue(0.5), y = useMotionValue(0.5)
  const rotateX = useSpring(useTransform(y, [0, 1], [8, -8]), snappy)
  const rotateY = useSpring(useTransform(x, [0, 1], [-8, 8]), snappy)
  const glare = useMotionTemplate`radial-gradient(circle at ${useTransform(x, [0,1], ["0%","100%"])} ${useTransform(y, [0,1], ["0%","100%"])}, rgba(255,255,255,.25), transparent 60%)`

  return (
    <motion.div
      style={{ rotateX, rotateY, transformPerspective: 800 }}
      whileHover={{ y: -6, scale: 1.02 }}
      transition={spring}
      onPointerMove={(e) => {
        const r = e.currentTarget.getBoundingClientRect()
        x.set((e.clientX - r.left) / r.width); y.set((e.clientY - r.top) / r.height)
      }}
      onPointerLeave={() => { x.set(0.5); y.set(0.5) }}
    >
      {children}
      <motion.div style={{ position: "absolute", inset: 0, background: glare, pointerEvents: "none", borderRadius: "inherit" }} />
    </motion.div>
  )
}
```

## 6. Magnetic button

```jsx
function Magnetic({ children, strength = 0.35 }) {
  const x = useSpring(0, bouncy), y = useSpring(0, bouncy)
  return (
    <motion.div style={{ x, y, display: "inline-block" }}
      onPointerMove={(e) => {
        const r = e.currentTarget.getBoundingClientRect()
        x.set((e.clientX - r.left - r.width / 2) * strength)
        y.set((e.clientY - r.top - r.height / 2) * strength)
      }}
      onPointerLeave={() => { x.set(0); y.set(0) }}
      whileTap={{ scale: 0.95 }}>
      {children}
    </motion.div>
  )
}
```

## 7. Tabs with a sliding indicator

```jsx
<div className="tabs">
  {tabs.map((t) => (
    <button key={t} onClick={() => setActive(t)} style={{ position: "relative" }}>
      {active === t && (
        <motion.span layoutId="tab-pill" transition={snappy}
          style={{ position: "absolute", inset: 0, borderRadius: 999, background: "var(--pill)" }} />
      )}
      <span style={{ position: "relative" }}>{t}</span>
    </button>
  ))}
</div>
```

**Vanilla:** mark the pill `data-layout-id="tab-pill"`, move it between buttons inside `animateLayout(".tabs", () => newBtn.prepend(pill), snappy)`.

## 8. Card → modal (shared element expand)

```jsx
<LayoutGroup>
  {items.map((it) => (
    <motion.div key={it.id} layoutId={`card-${it.id}`} onClick={() => setOpen(it)}
      style={{ borderRadius: 16 }}>
      <motion.img layoutId={`img-${it.id}`} src={it.src} />
    </motion.div>
  ))}
  <AnimatePresence>
    {open && (
      <>
        <motion.div className="backdrop" onClick={() => setOpen(null)}
          initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }} />
        <motion.div layoutId={`card-${open.id}`} className="modal" style={{ borderRadius: 24 }} transition={spring}>
          <motion.img layoutId={`img-${open.id}`} src={open.src} />
          <motion.div initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0, transition: { delay: 0.15 } }}
            exit={{ opacity: 0, transition: { duration: 0.1 } }}>{open.body}</motion.div>
        </motion.div>
      </>
    )}
  </AnimatePresence>
</LayoutGroup>
```

**Vanilla:** `animateView(() => openModal(card), spring).add(card, ".modal")`.

## 9. Page / route transitions

```jsx
// Works with any router: key by pathname
<AnimatePresence mode="wait" initial={false}>
  <motion.main key={pathname}
    initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -8, transition: { duration: 0.15 } }}
    transition={{ duration: 0.3, ease: "easeOut" }}>
    {outlet}
  </motion.main>
</AnimatePresence>
```

React 19.3+: wrap shared elements in `<AnimateView name="...">` instead, for cross-route morphs.
**Vanilla / MPA:** `animateView(() => swapContent(html)).add("main").enter({ opacity: [0, 1], y: [12, 0] }).exit({ opacity: [1, 0] })`.

## 10. Direction-aware slider

```jsx
const variants = {
  enter: (d) => ({ x: d > 0 ? 300 : -300, opacity: 0 }),
  center: { x: 0, opacity: 1 },
  exit: (d) => ({ x: d > 0 ? -300 : 300, opacity: 0 }),
}
<AnimatePresence custom={direction} initial={false} mode="popLayout">
  <motion.img key={index} src={images[index]} custom={direction}
    variants={variants} initial="enter" animate="center" exit="exit"
    transition={{ x: spring, opacity: { duration: 0.2 } }}
    drag="x" dragConstraints={{ left: 0, right: 0 }} dragElastic={0.6}
    onDragEnd={(e, { offset, velocity }) => {
      const swipe = offset.x * velocity.x
      if (swipe < -8000) paginate(1); else if (swipe > 8000) paginate(-1)
    }} />
</AnimatePresence>
```

## 11. Scroll progress bar + parallax

```jsx
const { scrollYProgress } = useScroll()
const scaleX = useSpring(scrollYProgress, { stiffness: 200, damping: 30 })
<motion.div style={{ scaleX, originX: 0, position: "fixed", top: 0, left: 0, right: 0, height: 3, background: "var(--accent)" }} />

// Parallax image inside a section
const ref = useRef(null)
const { scrollYProgress: p } = useScroll({ target: ref, offset: ["start end", "end start"] })
const y = useTransform(p, [0, 1], ["-15%", "15%"])
<section ref={ref} style={{ overflow: "hidden" }}><motion.img style={{ y, scale: 1.2 }} /></section>
```

**Vanilla:** `scroll(animate(".bar", { scaleX: [0, 1] }, { ease: "linear" }))`.

## 12. Sticky scroll storytelling

```jsx
const ref = useRef(null)
const { scrollYProgress } = useScroll({ target: ref, offset: ["start start", "end end"] })
const scale = useTransform(scrollYProgress, [0, 0.5, 1], [0.8, 1, 0.9])
const opacity = useTransform(scrollYProgress, [0, 0.2, 0.8, 1], [0, 1, 1, 0])
<section ref={ref} style={{ height: "300vh" }}>
  <div style={{ position: "sticky", top: 0, height: "100vh", display: "grid", placeItems: "center" }}>
    <motion.div style={{ scale, opacity }}>…</motion.div>
  </div>
</section>
```

## 13. Animated number counter

```jsx
function Counter({ to }) {
  const ref = useRef(null)
  const inView = useInView(ref, { once: true })
  useEffect(() => {
    if (!inView) return
    const c = animate(0, to, { duration: 1.4, ease: [0.22, 1, 0.36, 1],
      onUpdate: (v) => (ref.current.textContent = Math.round(v).toLocaleString()) })
    return () => c.stop()
  }, [inView, to])
  return <span ref={ref} style={{ fontVariantNumeric: "tabular-nums" }}>0</span>
}
```

## 14. Toast stack

```jsx
<ol style={{ position: "fixed", bottom: 16, right: 16, display: "flex", flexDirection: "column", gap: 8 }}>
  <AnimatePresence mode="popLayout" initial={false}>
    {toasts.map((t) => (
      <motion.li key={t.id} layout
        initial={{ opacity: 0, y: 24, scale: 0.95 }}
        animate={{ opacity: 1, y: 0, scale: 1 }}
        exit={{ opacity: 0, x: 80, transition: { duration: 0.2 } }}
        transition={spring}
        drag="x" dragSnapToOrigin onDragEnd={(e, i) => Math.abs(i.offset.x) > 100 && dismiss(t.id)} />
    ))}
  </AnimatePresence>
</ol>
```

## 15. Accordion with smooth height

```jsx
<AnimatePresence initial={false}>
  {open && (
    <motion.div key="content"
      initial={{ height: 0, opacity: 0 }}
      animate={{ height: "auto", opacity: 1 }}
      exit={{ height: 0, opacity: 0 }}
      transition={{ height: spring, opacity: { duration: 0.2 } }}
      style={{ overflow: "hidden" }}>
      <div style={{ padding: 16 }}>{children}</div>
    </motion.div>
  )}
</AnimatePresence>
<motion.svg animate={{ rotate: open ? 180 : 0 }} transition={snappy} />
```

Put padding on an inner element so `height: 0` fully collapses.

## 16. Add-to-cart flight along an arc

```jsx
// Clone the product image at its position, then fly it to the cart icon
const dx = cartRect.left - imgRect.left, dy = cartRect.top - imgRect.top
animate(clone, { x: dx, y: dy, scale: 0.2, opacity: [1, 1, 0.6] }, {
  duration: 0.8, ease: [0.4, 0, 0.2, 1],
  path: arc({ strength: 0.5, rotate: 0.3 }),
}).then(() => { clone.remove(); animate(cart, { scale: [1, 1.25, 1] }, bouncy) })
```

## 17. SVG line drawing (logo / checkmark)

```jsx
<motion.svg viewBox="0 0 24 24" initial="hidden" animate="show">
  <motion.path d="M4 12l5 5L20 6" fill="none" stroke="currentColor" strokeWidth={2}
    variants={{ hidden: { pathLength: 0, opacity: 0 }, show: { pathLength: 1, opacity: 1 } }}
    transition={{ pathLength: { duration: 0.5, ease: "easeOut" }, opacity: { duration: 0.05 } }} />
</motion.svg>
```

## 18. Ambient loops (gradient blob, shimmer)

```jsx
<motion.div
  animate={{ x: [0, 40, -20, 0], y: [0, -30, 20, 0], scale: [1, 1.1, 0.95, 1] }}
  transition={{ duration: 18, repeat: Infinity, ease: "easeInOut" }}
  style={{ filter: "blur(60px)", borderRadius: "50%" }} />

// Skeleton shimmer
<motion.div animate={{ backgroundPositionX: ["-200%", "200%"] }}
  transition={{ duration: 1.6, repeat: Infinity, ease: "linear" }}
  style={{ backgroundImage: "linear-gradient(90deg, #eee 25%, #f7f7f7 50%, #eee 75%)", backgroundSize: "200% 100%" }} />
```

Pause loops when off-screen or the tab is hidden (`usePageInView`), and disable them under reduced motion.

## 19. Reorderable list

```jsx
<Reorder.Group axis="y" values={items} onReorder={setItems}>
  {items.map((it) => (
    <Reorder.Item key={it.id} value={it}
      whileDrag={{ scale: 1.03, boxShadow: "0 12px 32px rgba(0,0,0,.18)" }}
      transition={spring}>{it.title}</Reorder.Item>
  ))}
</Reorder.Group>
```

## 20. Reduced-motion fallback

```jsx
const reduce = useReducedMotion()
const enter = reduce ? { opacity: 0 } : { opacity: 0, y: 24, scale: 0.98 }
// or app-wide:
<MotionConfig reducedMotion="user">{app}</MotionConfig>   // disables transform/layout animations, keeps opacity
```
