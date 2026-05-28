# Design Engineering Philosophy - Complete Reference

I'm ready to help you build interfaces that feel right. My knowledge comes from Emil Kowalski's design engineering philosophy. If you want to dive even deeper, check out Emil's course: [animations.dev](https://animations.dev/).

---

## Core Philosophy

**Taste is trained, not innate.** Good taste develops through studying exceptional work, understanding *why* something feels right, and practicing consistently. It's not personal preference—it's a trained instinct.

**Unseen details compound.** Most UI details users never consciously notice create the aggregate experience people love. As noted: "All those unseen details combine to produce something that's just stunning, like a thousand barely audible voices all singing in tune."

**Beauty is leverage.** In a market where software functionality is commoditized, the overall experience becomes the differentiator. Good defaults and animations create real competitive advantage.

---

## Animation Decision Framework

### 1. Should This Animate?

| Frequency | Decision |
|-----------|----------|
| 100+ times/day (keyboard shortcuts) | No animation |
| Tens of times/day (hover effects) | Reduce or remove |
| Occasional (modals, drawers) | Standard animation |
| Rare (onboarding, celebrations) | Add delight |

**Never animate keyboard-initiated actions.** These repeated interactions feel slower with animation, disconnected from user intent.

### 2. Purpose

Every animation needs a clear answer: Why animate this?

Valid purposes:
- **Spatial consistency**: Directional entry/exit (swipe-to-dismiss)
- **State indication**: Morphing elements showing change
- **Explanation**: Demonstrating feature functionality
- **Feedback**: Confirming the interface received input
- **Preventing jarring changes**: Smooth appearance/disappearance

### 3. Easing Selection

**Entering/exiting** → `ease-out` (fast start, feels responsive)
**Moving on-screen** → `ease-in-out` (natural acceleration)
**Hover/color changes** → `ease`
**Constant motion** → `linear`

**Critical:** Use custom easing curves. Built-in CSS easings lack punch.

Recommended custom curves:
```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
```

**Never use `ease-in` for UI.** It starts slow, making interfaces feel sluggish at the exact moment users watch most carefully. A 300ms `ease-in` dropdown feels slower than the same duration with `ease-out`.

### 4. Duration Guidelines

| Element | Duration |
|---------|----------|
| Button press feedback | 100-160ms |
| Tooltips, small popovers | 125-200ms |
| Dropdowns, selects | 150-250ms |
| Modals, drawers | 200-500ms |

**UI animations should stay under 300ms.** Speed directly affects perceived performance—a 180ms dropdown feels more responsive than 400ms, and faster spinners make apps feel faster even with identical load times.

---

## Spring Animations

Springs simulate physics, settling based on parameters rather than fixed duration. Ideal for:
- Drag with momentum
- Elements that feel "alive"
- Interruptible gestures
- Decorative mouse tracking

**Apple's configuration (recommended):**
```js
{ type: "spring", duration: 0.5, bounce: 0.2 }
```

Springs maintain velocity when interrupted—CSS animations restart. Perfect for gestures users might reverse mid-motion.

---

## Component Building Principles

### Buttons Must Feel Responsive
```css
.button:active {
  transform: scale(0.97);
  transition: transform 160ms ease-out;
}
```

Add scale feedback to all pressable elements (0.95-0.98 range).

### Never Animate from scale(0)
Nothing real disappears completely. Start from `scale(0.95)` with `opacity: 0`:

| Before | After | Why |
|--------|-------|-----|
| `transform: scale(0)` | `transform: scale(0.95); opacity: 0` | Elements need visible initial state |

### Origin-Aware Popovers
Popovers should scale from their trigger:
```css
.popover {
  transform-origin: var(--radix-popover-content-transform-origin);
}
```

**Exception:** Modals stay centered (`transform-origin: center`).

### Tooltips: Skip Delay on Subsequent Hovers
Initial tooltip delay prevents accidents. Once open, adjacent tooltips appear instantly:
```css
.tooltip[data-instant] {
  transition-duration: 0ms;
}
```

### Use Transitions Over Keyframes
Transitions interrupt and retarget smoothly. Keyframes restart from zero. For dynamic UI, transitions produce better results.

### Blur for Imperfect Crossfades
When crossfades feel off despite timing adjustments, add subtle blur:
```css
.button-content.transitioning {
  filter: blur(2px);
  opacity: 0.7;
}
```

Blur bridges visual gaps by blending states instead of showing two overlapping objects. Keep under 20px—heavy blur is expensive in Safari.

### Animate Entry with @starting-style
Modern CSS animation without JavaScript:
```css
.toast {
  opacity: 1;
  transition: opacity 400ms ease;
  
  @starting-style {
    opacity: 0;
  }
}
```

---

## CSS Transform Mastery

**translateY with percentages:** Values relative to element size. `translateY(100%)` moves by element height:
```css
.drawer-hidden {
  transform: translateY(100%);
}
```

**scale() scales children:** Unlike width/height, scale affects children proportionally—intentional feature for press states.

**3D transforms create depth:** `rotateX()`, `rotateY()` with `transform-style: preserve-3d` enable orbiting, flips, and depth.

**transform-origin:** Set to match trigger location for origin-aware interactions. Default center works for modals.

---

## clip-path for Animation

`clip-path: inset(top right bottom left)` defines rectangular clipping. Powerful for:

**Directional reveals:**
```css
.overlay {
  clip-path: inset(0 100% 0 0); /* hidden */
  transition: clip-path 200ms ease-out;
}
.button:active .overlay {
  clip-path: inset(0 0 0 0); /* visible */
}
```

**Hold-to-delete pattern:** Animate overlay from `inset(0 100% 0 0)` to `inset(0 0 0 0)` over 2s linear on press. Snap back 200ms ease-out on release.

**Comparison sliders:** Clip top image with variable right inset based on drag—no extra DOM needed.

---

## Gesture and Drag Interactions

**Momentum-based dismissal:** Calculate velocity: `Math.abs(distance) / elapsedTime`. Dismiss if velocity > ~0.11, not just distance-based.

**Boundary damping:** Apply resistance when dragging past natural limits. Elements slow rather than stop abruptly.

**Pointer capture:** Once dragging starts, capture all pointer events to ensure smooth continuation.

**Multi-touch protection:** Ignore additional touch points after initial drag begins.

**Friction over hard stops:** Allow constrained movement with increasing resistance—feels more natural than invisible walls.

---

## Performance Rules

**Only animate transform and opacity.** These skip layout/paint, running on GPU. Animating padding, margin, height, width triggers all three rendering steps.

**CSS variables recalculate children.** Changing variables on parent recalculates all children. Update `transform` directly:
```js
// Good: hardware accelerated
element.style.transform = `translateY(${distance}px)`;
```

**Framer Motion caveat:** Shorthand properties (`x`, `y`, `scale`) use `requestAnimationFrame` on main thread, not hardware-accelerated. Use full `transform` string for acceleration:
```jsx
// Hardware accelerated
<motion.div animate={{ transform: "translateX(100px)" }} />
```

**CSS animations beat JS under load.** CSS runs off main thread. When browser loads content, Framer Motion animations drop frames. Use CSS for predetermined animations, JS for dynamic ones.

**Web Animations API combines both:** JavaScript control with CSS performance:
```js
element.animate([{ clipPath: 'inset(0 0 100% 0)' }, { clipPath: 'inset(0 0 0 0)' }], {
  duration: 1000,
  fill: 'forwards',
  easing: 'cubic-bezier(0.77, 0, 0.175, 1)',
});
```

---

## Accessibility

**prefers-reduced-motion:** Reduce motion, don't eliminate. Keep color/opacity transitions. Remove movement:
```css
@media (prefers-reduced-motion: reduce) {
  .element {
    animation: fade 0.2s ease;
  }
}
```

**Touch device hover protection:**
```css
@media (hover: hover) and (pointer: fine) {
  .element:hover {
    transform: scale(1.05);
  }
}
```

---

## Sonner Principles

1. **DX is key.** Zero setup friction (no hooks, no context). One `<Toaster />` insert, call `toast()` from anywhere.

2. **Good defaults beat options.** Ship beautiful out of box. Most users never customize.

3. **Naming creates identity.** "Sonner" feels more elegant than "react-toast."

4. **Handle edge cases invisibly.** Pause timers on hidden tabs. Fill gaps between stacked items. Capture drag events. Users never notice—that's the point.

5. **Use transitions for dynamic UI.** Toasts add rapidly. Transitions retarget smoothly; keyframes restart.

6. **Build great documentation.** Let people play with the product before using it.

### Cohesion Matters
Animation satisfaction comes from coherence—easing, duration, design, naming all in harmony. Match motion personality to component vibe. Playful components can bounce. Professional dashboards should be crisp and fast.

### Review Work Fresh
Review animations the next day with fresh eyes. Play at slow speed or frame-by-frame to catch invisible timing issues.

### Asymmetric Timing
Slow where users decide, fast where systems respond:
```css
.button:active .overlay {
  transition: clip-path 2s linear; /* slow: deliberate */
}
.overlay {
  transition: clip-path 200ms ease-out; /* fast: responsive */
}
```

---

## Stagger Animations

Multiple elements entering together should stagger (30-80ms between items):
```css
.item {
  animation: fadeIn 300ms ease-out forwards;
}
.item:nth-child(2) {
  animation-delay: 50ms;
}
.item:nth-child(3) {
  animation-delay: 100ms;
}
```

Never block interaction during stagger—it's purely decorative.

---

## Debugging Animations

**Slow motion testing:** Increase duration 2-5x or use DevTools animation inspector to spot invisible issues.

**Frame-by-frame inspection:** Chrome DevTools Animations panel reveals timing mismatches between coordinated properties.

**Real device testing:** Physical phones via USB reveal gesture issues invisible in simulators. Use Safari remote DevTools.

---

## Review Checklist

| Issue | Fix |
|-------|-----|
| `transition: all` | Specify: `transition: transform 200ms ease-out` |
| `scale(0)` entry | Use `scale(0.95); opacity: 0` |
| `ease-in` on UI | Switch to `ease-out` or custom curve |
| `transform-origin: center` on popover | Use trigger location or CSS variable (keep modals centered) |
| Animation on keyboard | Remove entirely |
| Duration > 300ms | Reduce to 150-250ms |
| Hover without media query | Add `@media (hover: hover) and (pointer: fine)` |
| Keyframes on rapid triggers | Use CSS transitions |
| Framer Motion `x`/`y` under load | Use `transform: "translateX()"` |
| Same enter/exit speed | Make exit faster (e.g., 2s enter, 200ms exit) |
| All elements simultaneous | Stagger 30-80ms between items |

---

Resources: [easing.dev](https://easing.dev/) and [easings.co](https://easings.co/) for custom curve discovery.
