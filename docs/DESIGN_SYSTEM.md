# Design System

**Direction:** premium, minimal, warm light. A beige canvas, cream surfaces, soft pink and butter-yellow tints, deep ink text and one confident berry accent. **No pure white and no pure black anywhere.** Contrast comes from dark warm ink, a deep accent and visible borders, not from a white background.

## 1. Principles

1. **Light only, never pure white.** Beige canvas, cream surfaces. No dark mode in v1.
2. **One accent.** Deep berry for action. Gold only for credits. Pink and yellow are tints, not accents.
3. **Layered warm surfaces.** Depth comes from beige, cream, blush and butter layers plus clear borders.
4. **Strong contrast.** Near-black warm ink, deep berry buttons, 3:1 minimum on input borders.
5. **Section rhythm.** Alternate beige, blush and butter bands to break long pages without adding decoration.
6. **Motion with purpose.** See ANIMATION.md.
7. **Accessible.** Ratios below are computed. Re-check any new pair before shipping.

## 2. Color Tokens

```css
:root {
  /* Canvas (beige) and tinted bands */
  --bg:            #F5EEE0;  /* page background, warm beige */
  --bg-subtle:     #EDE3D0;  /* deeper beige: sections, input fill, hover */
  --bg-blush:      #FBE9E4;  /* light pink band */
  --bg-butter:     #FFF6D6;  /* light yellow band */

  /* Surfaces (cream, never pure white) */
  --surface:       #FFFAF0;  /* cards, modals, inputs */
  --surface-hover: #FFF5E4;
  --surface-pink:  #FDEFEC;  /* highlighted cards */

  /* Ink (warm near-black) */
  --ink:           #1C1512;  /* primary text */
  --ink-2:         #4A3D35;  /* secondary text */
  --ink-muted:     #6B5D52;  /* captions, placeholders */
  --ink-faint:     #A09284;  /* disabled only, never for text that must be read */

  /* Lines */
  --border:        #DDD0BA;  /* cards, dividers (decorative) */
  --border-strong: #C4B396;  /* hover, emphasis */
  --border-input:  #8C7B65;  /* inputs, checkboxes (3:1+ against surfaces) */

  /* Brand (deep berry) */
  --brand:         #8E2554;
  --brand-hover:   #741B44;
  --brand-soft:    #F9DDE2;  /* light pink tint: chips, selected rows */
  --brand-ink:     #5E1238;  /* text on brand-soft */
  --on-brand:      #FFFAF0;  /* text on brand (cream, not white) */

  /* Credits (light yellow + amber) */
  --credit:        #8A5A00;
  --credit-soft:   #FDEFB8;

  /* Status */
  --success:       #1F6B45;  --success-soft: #E4EFD9;
  --warning:       #8F4708;  --warning-soft: #FCE8C8;
  --danger:        #B3261E;  --danger-soft:  #FBE0DC;

  /* Effects (warm shadows, not grey) */
  --focus-ring:    0 0 0 3px rgba(142, 37, 84, 0.28);
  --shadow-sm:     0 1px 2px rgba(60, 35, 20, 0.08);
  --shadow-md:     0 6px 18px rgba(60, 35, 20, 0.10);
  --shadow-lg:     0 18px 48px rgba(60, 35, 20, 0.14);
}
```

Usage rules:
- Page background is `--bg`. Cards are `--surface` with a 1px `--border` and `--shadow-sm`, so they lift visibly off the beige.
- Alternate full-width bands between `--bg`, `--bg-blush` and `--bg-butter`. Never two tinted bands back to back.
- Primary buttons use `--brand` with `--on-brand` text. Secondary use `--surface` with `--border-strong`.
- `--credit` and `--credit-soft` appear only on credit amounts, wallet and earn/spend states.
- Pink (`--brand-soft`, `--bg-blush`) marks selection and warmth. Yellow (`--bg-butter`, `--credit-soft`) marks money, highlights and tips.
- Never use `#FFFFFF` or `#000000`. Text on dark fills uses `--on-brand`.
- The only gradient allowed is a soft radial wash of `--bg-blush` into `--bg-butter` behind the hero.

### Contrast Check (WCAG 2.1, computed)

| Pair | Ratio | Passes |
|---|---|---|
| `--ink` on `--bg` | 15.6:1 | AAA |
| `--ink` on `--surface` | 17.3:1 | AAA |
| `--ink` on `--bg-blush` / `--bg-butter` | 15.3:1 / 16.7:1 | AAA |
| `--ink-2` on `--bg` | 9.1:1 | AAA |
| `--ink-muted` on `--bg` / `--surface` | 5.5:1 / 6.1:1 | AA |
| `--ink-muted` on `--bg-subtle` | 5.0:1 | AA |
| `--brand` on `--bg` / `--surface` (links) | 7.2:1 / 7.9:1 | AAA |
| `--on-brand` on `--brand` (buttons) | 7.9:1 | AAA |
| `--on-brand` on `--brand-hover` | 10.2:1 | AAA |
| `--brand-ink` on `--brand-soft` | 10.2:1 | AAA |
| `--credit` on `--credit-soft` | 5.1:1 | AA |
| `--success` / `--warning` / `--danger` on their soft fills | 5.4 / 5.7 / 5.2:1 | AA |
| `--border-input` on `--surface` / `--bg` | 3.9:1 / 3.5:1 | Passes 3:1 for UI components |
| `--border` on `--surface` | 1.5:1 | Decorative only, never the sole edge of an input |
| `--ink-faint` on `--bg` | 2.6:1 | Disabled state only |

## 3. Typography

| Role | Font | Notes |
|---|---|---|
| Display | **Instrument Serif** | Hero and section headlines, elegant contrast |
| UI and body | **Inter** | Clean, highly legible |
| Numbers / code | **JetBrains Mono** | Credit amounts, timestamps (optional) |

```css
:root {
  --font-display: 'Instrument Serif', Georgia, serif;
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;
}
```

| Token | Size / Line height | Weight | Use |
|---|---|---|---|
| `display-xl` | clamp(3rem, 6vw, 5.5rem) / 1.02 | 400 serif | Hero |
| `display-lg` | clamp(2.25rem, 4vw, 3.5rem) / 1.08 | 400 serif | Section titles |
| `h3` | 1.5rem / 1.3 | 600 | Card titles |
| `body-lg` | 1.125rem / 1.65 | 400 | Lead paragraphs |
| `body` | 1rem / 1.6 | 400 | Default |
| `small` | 0.875rem / 1.5 | 500 | Labels, meta |
| `caption` | 0.75rem / 1.4 | 500 | Overlines (uppercase, 0.08em tracking) |

Headlines use `letter-spacing: -0.02em`. Max reading width 65ch. One headline word may use italic Instrument Serif in `--brand` for emphasis.

## 4. Spacing, Radius, Layout

- 4px base grid. Scale: 4, 8, 12, 16, 24, 32, 48, 64, 96, 128.
- Section vertical padding: 96px desktop, 64px mobile.
- Container: max 1200px, 24px side padding (16px mobile).
- Radius: `sm 8px`, `md 12px`, `lg 20px`, `xl 28px`, `pill 999px`.
- Grid: 12 columns desktop, 4 mobile, 24px gutters.
- Breakpoints: `sm 640`, `md 768`, `lg 1024`, `xl 1280`.

## 5. Components

| Component | Spec |
|---|---|
| Button primary | Height 44px, radius pill, `--brand` fill, `--on-brand` text, hover `--brand-hover`, focus `--focus-ring` |
| Button secondary | `--surface` fill, 1.5px `--border-strong`, `--ink` text, hover `--bg-subtle` |
| Button ghost | Transparent, `--ink` text, hover `--bg-subtle` |
| Input | Height 44px, `--surface`, 1.5px `--border-input`, radius 12px, focus border `--brand` + `--focus-ring` |
| Card | `--surface`, 1px `--border`, radius 20px, `--shadow-sm`, hover lift to `--shadow-md` and `--border-strong` |
| Feature card (highlight) | `--surface-pink`, 1px `--brand-soft` border |
| Chip (skill) | Pill, `--brand-soft` bg, `--brand-ink` text, 28px height |
| Credit badge | Pill, `--credit-soft` bg, `--credit` text, mono numerals |
| Avatar | Circle, 1px `--border-strong`, initials on `--bg-subtle` |
| Rating | Five stars, filled with `--credit`, empty with `--border-strong` |
| Modal | `--surface`, radius 28px, `--shadow-lg`, scrim `rgba(28, 21, 18, 0.45)` with blur 8px |
| Toast | `--ink` bg with `--on-brand` text, radius 14px |
| Tabs | Underline style, 2px `--brand` indicator, inactive `--ink-muted` |
| Table | Header `--bg-subtle`, rows `--surface`, 1px `--border` dividers, hover `--surface-hover` |
| Skeleton | `--bg-subtle` with soft shimmer (disabled under reduced motion) |

States: every interactive element has default, hover, focus-visible, active, disabled. The focus ring is always visible for keyboard users.

## 6. Key Screens

| Screen | Layout notes |
|---|---|
| Landing | Beige hero with blush-to-butter wash, large serif headline, subtle three.js scene, "Teach, Earn, Learn" on a blush band, social proof on butter, closing CTA on beige |
| Auth | Cream card on beige, single column, inline validation |
| Onboarding | Stepper: basics, skills offered, skills wanted, availability |
| Dashboard | Wallet summary card on `--credit-soft`, upcoming sessions, top matches |
| Discover | Filter bar + responsive card grid of matches with score and mutual badge (pink chip) |
| Profile | Blush hero header, skills chips, availability grid, reviews |
| Booking | Side sheet: skill, date/time in both timezones, duration, credit cost preview |
| Wallet | Large credit figure on butter, ledger table with earn/spend color coding |
| Messages | Two-pane list + thread, own bubbles `--brand-soft`, other bubbles `--surface` |

## 7. Imagery and Icons

- Icons: `@phosphor-icons/react`, weight `regular` (or `light` for large sizes). **No lucide-react.**
- Illustration: minimal line shapes in `--ink` and the three.js scene, no stock photography.
- Avatars: user uploads or initials.

## 8. Voice

Warm, plain and confident. Short sentences. Examples: "Teach an hour, learn an hour." "You have 2.5 credits."

## 9. Tailwind Mapping

```js
// tailwind.config.js (v3 style; in v4 declare in @theme)
// Names avoid collisions with shadcn: use brand (not accent) and ink-muted (not muted).
export default {
  theme: {
    extend: {
      colors: {
        bg: 'var(--bg)', 'bg-subtle': 'var(--bg-subtle)', 'bg-blush': 'var(--bg-blush)', 'bg-butter': 'var(--bg-butter)',
        surface: 'var(--surface)', 'surface-pink': 'var(--surface-pink)',
        ink: 'var(--ink)', 'ink-2': 'var(--ink-2)', 'ink-muted': 'var(--ink-muted)',
        border: 'var(--border)', 'border-strong': 'var(--border-strong)', 'border-input': 'var(--border-input)',
        brand: 'var(--brand)', 'brand-hover': 'var(--brand-hover)', 'brand-soft': 'var(--brand-soft)', 'brand-ink': 'var(--brand-ink)', 'on-brand': 'var(--on-brand)',
        credit: 'var(--credit)', 'credit-soft': 'var(--credit-soft)',
      },
      fontFamily: { display: 'var(--font-display)', sans: 'var(--font-sans)', mono: 'var(--font-mono)' },
      borderRadius: { md: '12px', lg: '20px', xl: '28px' },
      boxShadow: { sm: 'var(--shadow-sm)', md: 'var(--shadow-md)', lg: 'var(--shadow-lg)' },
    },
  },
};
```