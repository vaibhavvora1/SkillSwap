# UI Components

Component strategy: **React Bits** for expressive, animated pieces; **Watermelon UI** for ready-made structural UI (components, blocks, dashboards); small custom primitives for everything else (buttons, inputs, modals). Icons come from **Phosphor**. **lucide-react is not used anywhere.**

| Library | Use it for | Avoid it for |
|---|---|---|
| React Bits | Hero text, backgrounds, counters, one "wow" effect per viewport | Core app UI like forms and tables |
| Watermelon UI | Dashboard layout, tables, forms, navigation, dialogs, landing blocks | Hero effects (React Bits already covers them) |
| Custom `ui/` primitives | Anything used on 3+ pages that must match SkillSwap tokens exactly | Reinventing what a library already gives you |

Never use two libraries for the same job on the same screen.

## 1. Setup

React Bits is a copy-paste library: you add the components you need into your own codebase instead of installing a package. Pick the **TypeScript + Tailwind** (or JavaScript + Tailwind) variant to match the project, and check <https://reactbits.dev> for the current install command and the exact component names and props before you copy anything.

```bash
# one-time
npm install gsap @gsap/react three @react-three/fiber @react-three/drei
npm install @phosphor-icons/react clsx tailwind-merge
```

Put copied components in:

```
client/src/components/reactbits/    # untouched upstream copies (React Bits)
client/src/components/watermelon/   # untouched upstream copies (Watermelon UI)
client/src/components/ui/           # your own wrappers with SkillSwap tokens
```

Rules:
1. Do not edit files in `reactbits/` or `watermelon/` heavily. Wrap them in `ui/` and pass SkillSwap colors as props.
2. Override colors with tokens from DESIGN_SYSTEM.md. Most React Bits components accept color props or CSS variables.
3. Each animated component respects `prefers-reduced-motion` (wrap or disable).
4. Use at most one "attention" effect per viewport.

## 2. Icons (replacing lucide-react)

```bash
npm install @phosphor-icons/react
```

```jsx
import { Coins, CalendarCheck, ChatCircleDots, Star, UserCircle } from '@phosphor-icons/react';

<Coins size={20} weight="regular" />
```

| Need | Phosphor icon |
|---|---|
| Wallet / credits | `Coins`, `Wallet` |
| Session / calendar | `CalendarCheck`, `Clock` |
| Chat | `ChatCircleDots` |
| Rating | `Star` (weight `fill` when active) |
| Profile | `UserCircle` |
| Match | `Handshake`, `ArrowsLeftRight` |
| Search / filter | `MagnifyingGlass`, `Funnel` |
| Notifications | `Bell` |
| Safety | `ShieldCheck`, `Flag`, `Prohibit` |
| Navigation | `ArrowRight`, `CaretDown`, `List`, `X` |

Add an ESLint rule so `lucide-react` can never be imported:

```js
// .eslintrc
"no-restricted-imports": ["error", { "paths": [{ "name": "lucide-react", "message": "Use @phosphor-icons/react" }] }]
```

## 3. React Bits Component Map

Names below are the components React Bits provides at the time of writing. Confirm availability on the site, and swap for an equivalent if one is renamed.

| Screen | Component | Use |
|---|---|---|
| Landing hero | **SplitText** or **BlurText** | Headline reveal |
| Landing hero | **Aurora** or **Particles** (very subtle, light colors) | Soft background wash, optional |
| Landing CTA | **Magnet** | Primary button pull effect |
| Landing steps | **AnimatedContent** / **FadeContent** | Scroll reveals |
| Landing stats | **CountUp** | Members, hours exchanged |
| Landing features | **SpotlightCard** | Feature cards with soft light follow |
| Landing testimonials | **Marquee**-style scroller | Quiet social proof |
| Navbar | **Dock** (optional, desktop) or **GooeyNav** / **PillNav** | Navigation |
| Discover | **TiltedCard** or **GlareHover** | Match cards (use sparingly) |
| Discover | **AnimatedList** | Staggered results |
| Profile | **ProfileCard** | Header card |
| Onboarding | **Stepper** | Multi-step form |
| Wallet | **CountUp** | Credit balance animation |
| Wallet | **StarBorder** | Highlight earn CTA |
| Buttons | **ClickSpark** | Tiny feedback on primary actions |
| Loading | **Shuffle** / **DecryptedText** (brand moments only) | Splash |
| Empty states | **FadeContent** | Gentle entry |

Avoid for this theme: heavy dark/neon effects (Silk, Liquid Ether, Lightning, etc.) unless recolored to the light palette and tested for performance.

## 4. Watermelon UI

[Watermelon UI](https://ui.watermelon.sh) is a React component registry for **Tailwind CSS and shadcn/ui**. Its catalog covers animated components, landing-page blocks, dashboards and templates, and it is built on React, Tailwind v4, Radix UI and Motion. Directory listings describe it as free and MIT-licensed. Confirm the license and what is free on the site before shipping, because some items in a registry can be paid.

### Setup

Watermelon runs on the shadcn toolchain, so initialise shadcn once:

```bash
npx shadcn@latest init
```

In the generated `components.json`, point the `ui` alias at the Watermelon folder. Otherwise the CLI drops files into `components/ui`, which is reserved for SkillSwap's own wrappers.

```json
{
  "aliases": {
    "components": "@/components",
    "ui": "@/components/watermelon",
    "utils": "@/lib/utils"
  }
}
```

Add each component with the exact install command shown on its page at ui.watermelon.sh. Do not guess component names. Or copy the source by hand into `components/watermelon/`.

Optional: connect the Watermelon MCP server so Claude Code can browse the registry (command from the Watermelon repo README):

```bash
claude mcp add --transport http watermelon https://mcp.watermelon.sh/mcp
```

### Where to Use It

Look in these catalog categories. Names vary, so pick the closest match on the site.

| Screen | Look for |
|---|---|
| Dashboard | Dashboard layout, sidebar, stat cards |
| Wallet | Data table, badges |
| Discover | Cards, filters, tabs, search input |
| Onboarding | Form components, steps |
| Booking | Dialog or sheet, calendar and date picker, select |
| Messages | List, avatar, input, scroll area |
| Landing | Landing-page blocks for features, FAQ, footer |
| Feedback | Toast, alert, skeleton, empty state |

### Rules for Using It

1. **Icons:** shadcn-style components often import `lucide-react`. After adding anything, run `grep -r "lucide-react" src` and swap each icon for a Phosphor one. The ESLint rule from section 2 fails the build if one slips through. Newer shadcn versions have an `iconLibrary` setting in `components.json`. If yours offers Phosphor, choose it, otherwise swap by hand.

   | Lucide | Phosphor |
   |---|---|
   | `Check` | `Check` |
   | `X` | `X` |
   | `ChevronDown` / `ChevronRight` | `CaretDown` / `CaretRight` |
   | `Search` | `MagnifyingGlass` |
   | `Menu` | `List` |
   | `Loader2` | `SpinnerGap` |
   | `MoreHorizontal` | `DotsThree` |

2. **Two animation engines:** Watermelon animates with Motion, SkillSwap uses GSAP. Leave Watermelon's internal Motion animations alone. Use GSAP for page-level motion. Never animate the same element with both. Do not import Motion in SkillSwap's own code.
3. **Reduced motion:** wrap the app once so Motion-based components respect the user setting.

   ```jsx
   import { MotionConfig } from 'motion/react';
   <MotionConfig reducedMotion="user"><App /></MotionConfig>
   ```

4. **Dark mode:** delete the generated `.dark` block. SkillSwap is light only.
5. **Review what you copy:** read the source before committing, remove unused dependencies, and check the bundle size after each addition.
6. **Wrap, don't scatter:** import Watermelon components through a wrapper in `ui/` when SkillSwap needs a fixed look or default props.

## 5. Theming Watermelon to SkillSwap Tokens

shadcn components read CSS variables such as `--primary` and `--muted`. Point those at the SkillSwap tokens from DESIGN_SYSTEM.md so every Watermelon component matches the premium minimal light theme:

```css
/* styles/tokens.css: load after the CSS that shadcn init generates */
:root {
  --background: var(--bg);
  --foreground: var(--ink);
  --card: var(--surface);          --card-foreground: var(--ink);
  --popover: var(--surface);       --popover-foreground: var(--ink);
  --primary: var(--brand);         --primary-foreground: #FFFFFF;
  --secondary: var(--bg-subtle);   --secondary-foreground: var(--ink);
  --muted: var(--bg-subtle);       --muted-foreground: var(--ink-muted);
  --accent: var(--brand-soft);     --accent-foreground: var(--brand-ink);
  --destructive: var(--danger);
  --input: var(--border-strong);
  --ring: var(--brand);
  --radius: 0.75rem;
  /* --border is already defined by SkillSwap tokens, do not redefine it */
}
```

**Name collisions:** shadcn already uses `--accent`, `--ring`, `--muted` and `--border`. To avoid clashes, the SkillSwap indigo token is named `--brand` (with `--brand-hover`, `--brand-soft`, `--brand-ink`), the focus shadow is `--focus-ring`, and the Tailwind utilities are `brand`, `brand-soft` and `ink-muted`. In SkillSwap's own code use `bg-brand`, not `bg-accent`.

Adoption checklist:
- [ ] `components.json` alias points to `components/watermelon`
- [ ] No `lucide-react` imports remain
- [ ] `.dark` block removed, `tokens.css` loads after the shadcn CSS
- [ ] Buttons, inputs and dialogs visually match DESIGN_SYSTEM.md
- [ ] Keyboard and screen reader check on every added dialog, menu and form
- [ ] Reduced-motion check with `MotionConfig`

## 6. Custom Primitives (`components/ui`)

| Component | Notes |
|---|---|
| `Button` | Variants: primary, secondary, ghost, danger. Sizes: sm, md, lg. Loading state |
| `Input`, `Textarea`, `Select` | Label, hint, error slots, accessible ids |
| `SkillChip` | Level indicator, removable variant |
| `CreditBadge` | Formats minutes to hours (`150` becomes `2.5`) |
| `Avatar` | Image with initials fallback |
| `Rating` | Read-only and input modes |
| `Modal`, `Sheet` | Focus trap, ESC to close, scroll lock |
| `Toast` | Queue, auto-dismiss, `aria-live="polite"` |
| `Tabs`, `Tooltip`, `Skeleton`, `EmptyState` | Standard patterns |
| `Page`, `Section`, `Container` | Layout wrappers using spacing tokens |

Credit formatting helper:

```js
export const formatCredits = (minutes) => {
  const hours = minutes / 60;
  return Number.isInteger(hours) ? `${hours}` : hours.toFixed(1);
};
```

## 7. Example: Themed React Bits Wrapper

```jsx
// components/ui/Headline.jsx
import SplitText from '../reactbits/SplitText';

export default function Headline({ children }) {
  return (
    <h1 className="font-display text-[clamp(3rem,6vw,5.5rem)] leading-[1.02] tracking-[-0.02em] text-ink">
      <SplitText text={children} delay={40} />
    </h1>
  );
}
```

Props differ per component version. Match them to the copy you downloaded.

## 8. Accessibility Checklist

- All interactive elements are reachable and operable with the keyboard.
- Visible `:focus-visible` ring on every control.
- Icon-only buttons have `aria-label`.
- Animated text keeps the full string in the DOM for screen readers (`aria-label` on the wrapper when splitting into spans).
- Color is never the only signal (earn/spend also use +/- and text).
- Touch targets are at least 44 by 44 px.