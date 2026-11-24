# Animation System

Tools: **GSAP** for UI motion and scroll, **three.js** (via `@react-three/fiber`) for one signature 3D moment, **React Bits** for ready-made animated components.

The motion personality is calm, precise and premium: slow easing, small distances, no bounce.

## 1. Principles

1. **Purposeful.** Motion explains change (appearing, moving, completing), never just decorates.
2. **Quiet.** One hero moment per page, everything else is subtle.
3. **Fast.** UI feedback under 200 ms, entrances 400 to 700 ms.
4. **Safe.** Respect `prefers-reduced-motion`. Never block content on animation.
5. **Cheap.** Animate only `transform` and `opacity`.

## 2. Motion Tokens

```js
// src/lib/motion.js
export const DUR = { instant: 0.12, fast: 0.2, base: 0.4, slow: 0.7, hero: 1.1 };
export const EASE = {
  out:    'power3.out',      // entrances
  inOut:  'power2.inOut',    // transitions
  expo:   'expo.out',        // hero reveals
  soft:   'sine.inOut',      // ambient loops
};
export const STAGGER = { tight: 0.04, base: 0.08, loose: 0.14 };
export const DIST = { sm: 8, md: 16, lg: 32 };  // px
```

## 3. GSAP Setup

```bash
npm install gsap @gsap/react
```

```js
// src/lib/gsap.js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';

gsap.registerPlugin(ScrollTrigger, useGSAP);

export const prefersReducedMotion = () =>
  typeof window !== 'undefined' && window.matchMedia('(prefers-reduced-motion: reduce)').matches;

export { gsap, ScrollTrigger, useGSAP };
```

Always use `useGSAP` so animations and ScrollTriggers are scoped and cleaned up when components unmount.

## 4. Reusable Patterns

### Reveal on scroll

```jsx
import { useRef } from 'react';
import { gsap, useGSAP, prefersReducedMotion } from '../lib/gsap';
import { DUR, EASE, STAGGER, DIST } from '../lib/motion';

export function RevealGroup({ children, className }) {
  const root = useRef(null);

  useGSAP(() => {
    if (prefersReducedMotion()) return;
    gsap.from('[data-reveal]', {
      y: DIST.md, opacity: 0, duration: DUR.slow, ease: EASE.out, stagger: STAGGER.base,
      scrollTrigger: { trigger: root.current, start: 'top 80%', once: true },
    });
  }, { scope: root });

  return <div ref={root} className={className}>{children}</div>;
}
// usage: <div data-reveal>...</div> inside RevealGroup
```

### Hero intro timeline

```js
useGSAP(() => {
  if (prefersReducedMotion()) return;
  const tl = gsap.timeline({ defaults: { ease: EASE.expo } });
  tl.from('.hero-overline', { y: 12, opacity: 0, duration: DUR.base })
    .from('.hero-title',    { y: 32, opacity: 0, duration: DUR.hero }, '-=0.2')
    .from('.hero-sub',      { y: 16, opacity: 0, duration: DUR.slow }, '-=0.7')
    .from('.hero-cta',      { y: 12, opacity: 0, duration: DUR.base, stagger: STAGGER.base }, '-=0.5')
    .from('.hero-canvas',   { opacity: 0, duration: DUR.hero }, '-=0.9');
}, { scope: heroRef });
```

### Counter (wallet and stats)

Use React Bits `CountUp`, or:

```js
gsap.to(obj, { value: target, duration: 1.2, ease: 'power2.out', onUpdate: () => setText(Math.round(obj.value)) });
```

### Card hover

Prefer CSS for hover (cheaper):

```css
.card { transition: transform .25s cubic-bezier(.22,1,.36,1), box-shadow .25s; }
.card:hover { transform: translateY(-4px); box-shadow: var(--shadow-md); }
```

### Page transitions

- Route change: fade out current (`opacity 1 to 0`, 150 ms), fade in next with `y: 8 to 0` (300 ms).
- Keep layout stable, never animate `height` or `width` of large blocks.

### Credit movement (signature micro-interaction)

When credits are earned, a gold `CreditBadge` floats up 12 px, fades, and the wallet figure counts up. When spent, the figure counts down and the badge slides left. Total under 900 ms.

### Session state change

On `session:updated`, the status pill cross-fades (`opacity` and `y: 4`), and the card gets a 600 ms soft `--accent-soft` flash.

## 5. Where Each Tool Is Used

| Area | Tool |
|---|---|
| Hero 3D scene | three.js (R3F) |
| Headline text reveal | React Bits `SplitText` / `BlurText` |
| Section reveals, timelines, pinned storytelling | GSAP + ScrollTrigger |
| Stat counters | React Bits `CountUp` |
| Cards hover | CSS, optional `SpotlightCard` |
| Buttons | CSS + `Magnet` / `ClickSpark` on primary CTAs only |
| Lists | GSAP stagger or `AnimatedList` |
| Modals and sheets | GSAP (`y`, `opacity`) with focus management |

## 6. three.js Hero Scene

**Concept:** two softly lit orbs orbit and exchange small glowing particles, representing two people swapping skills. Calm, slow, light-themed, on a transparent canvas so the page background shows through.

```bash
npm install three @react-three/fiber @react-three/drei
```

```jsx
// src/components/three/ExchangeScene.jsx
import { Canvas, useFrame } from '@react-three/fiber';
import { useRef, useMemo } from 'react';
import * as THREE from 'three';

function Orb({ radius, speed, color, phase }) {
  const ref = useRef();
  useFrame(({ clock }) => {
    const t = clock.getElapsedTime() * speed + phase;
    ref.current.position.set(Math.cos(t) * radius, Math.sin(t * 0.8) * 0.4, Math.sin(t) * radius);
  });
  return (
    <mesh ref={ref}>
      <sphereGeometry args={[0.55, 48, 48]} />
      <meshStandardMaterial color={color} roughness={0.35} metalness={0.05} />
    </mesh>
  );
}

function Flow({ count = 60 }) {
  const ref = useRef();
  const positions = useMemo(() => {
    const a = new Float32Array(count * 3);
    for (let i = 0; i < count; i++) {
      a[i * 3] = (Math.random() - 0.5) * 3;
      a[i * 3 + 1] = (Math.random() - 0.5) * 1.2;
      a[i * 3 + 2] = (Math.random() - 0.5) * 3;
    }
    return a;
  }, [count]);
  useFrame(({ clock }) => { ref.current.rotation.y = clock.getElapsedTime() * 0.08; });
  return (
    <points ref={ref}>
      <bufferGeometry>
        <bufferAttribute attach="attributes-position" array={positions} count={count} itemSize={3} />
      </bufferGeometry>
      <pointsMaterial size={0.04} color="#B7791F" transparent opacity={0.8} sizeAttenuation />
    </points>
  );
}

export default function ExchangeScene() {
  return (
    <Canvas
      dpr={[1, 1.5]}
      camera={{ position: [0, 0, 5], fov: 45 }}
      gl={{ alpha: true, antialias: true, powerPreference: 'low-power' }}
      frameloop="always"
    >
      <ambientLight intensity={0.9} />
      <directionalLight position={[3, 4, 5]} intensity={1.2} />
      <Orb radius={1.3} speed={0.45} color="#4F46E5" phase={0} />
      <Orb radius={1.3} speed={0.45} color="#FFFFFF" phase={Math.PI} />
      <Flow />
    </Canvas>
  );
}
```

Load lazily so it never blocks first paint:

```jsx
const ExchangeScene = lazy(() => import('../components/three/ExchangeScene'));

<Suspense fallback={<div className="hero-canvas-fallback" />}>
  {!prefersReducedMotion() && isDesktop && <ExchangeScene />}
</Suspense>
```

Optional: tie the camera or rotation to scroll progress using a GSAP ScrollTrigger that updates a ref read inside `useFrame`.

## 7. Performance Budget

| Item | Budget |
|---|---|
| three.js scene | Under 150 KB gzipped extra, lazy loaded, no external HDRI or textures |
| Frame rate | 60 fps on mid-range laptop, drop to static image if sustained under 40 fps |
| Draw calls | Under 50 |
| Canvas DPR | Capped at 1.5 |
| Animated properties | `transform`, `opacity` only |
| Mobile | No three.js, use static gradient or SVG fallback |
| LCP | Hero text must paint before any animation or 3D loads |

Safeguards:
- Pause rendering when the canvas is off-screen (IntersectionObserver, set `frameloop="demand"`).
- Pause when the tab is hidden.
- Dispose geometries and materials on unmount (R3F does this for declarative objects).
- Kill ScrollTriggers on route change (handled by `useGSAP` cleanup).
- Call `ScrollTrigger.refresh()` after fonts and images load.

## 8. Accessibility and Reduced Motion

- If `prefers-reduced-motion: reduce`: skip timelines, show final state, disable the 3D scene, replace counters with final numbers.
- No flashing content, no parallax that moves large areas.
- Animated headlines keep readable text in the DOM.
- Motion never carries meaning alone (state changes also have text or icon changes).

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; scroll-behavior: auto !important; }
}
```

## 9. QA Checklist

- [ ] Content visible with JavaScript animations disabled
- [ ] Reduced-motion mode tested
- [ ] No layout shift from animation (reserve space)
- [ ] Lighthouse performance 90+ on landing (mobile profile)
- [ ] No console warnings about unmounted ScrollTriggers
- [ ] Hero scene stops rendering when off-screen
