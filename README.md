
> **Live example:** this design system is used in production at
> [remy-camiguel-portfolio.vercel.app](https://remy-camiguel-portfolio.vercel.app),
> built by [Remy Camiguel](https://github.com/rxmdyrems).

---
name: minimal-monochrome-design-system
description: Use this skill whenever building a minimal, monochrome portfolio or UI — specifically for the font system, gray-ramp color tokens, halftone dot texture, and the theme toggle with circular reveal transition. Apply whenever the user asks for a "minimal design system," "dark mode toggle with transition," or "subtle dot texture."

---

# Minimal Monochrome Design System

A grayscale-only design language: no accent colors, no hue-based
emphasis. Contrast and inversion (ink-on-background, background-on-ink)
carry all visual weight. This skill documents typography, color tokens,
halftone texture, and the theme toggle + transition as one coherent
system — use the token names below exactly.

## Typography

```css
:root {
  --font: 'Geist', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-mono: 'Geist Mono', ui-monospace, SFMono-Regular, monospace;
}

body { font-family: var(--font); -webkit-font-smoothing: antialiased; }
h1, h2, h3, h4, h5, h6 { font-family: 'Geist Pixel', 'Geist Mono', 'Geist', monospace; }
.font-mono, code, kbd, pre { font-family: var(--font-mono); }
.prose, .prose p, .prose li, .prose blockquote { font-family: 'Source Serif 4', Georgia, serif; }

.eyebrow {
  font-size: 11px;
  letter-spacing: .16em;
  text-transform: uppercase;
  font-weight: 500;
}

.section-heading {
  font-size: clamp(40px, 7vw, 80px);
  line-height: .98;
  font-weight: 600;
  letter-spacing: -.04em;
  text-transform: uppercase;
}

@media (max-width: 480px) {
  .section-heading {
    font-size: clamp(34px, 11vw, 48px);
    letter-spacing: -.05em;
  }
}
```

Load fonts via `@fontsource-variable/geist`, `@fontsource-variable/geist-mono`, and `@fontsource/source-serif-4/400.css`. "Geist Pixel" is used as a heading fallback chain; if it isn't separately sourced in your project, "Geist Mono" uppercase is the practical heading font.

## Layout

```css
:root {
  --nav-h: 92px;
  --nav-h-m: 72px;
  --container: 1320px;
  --edge: clamp(24px, 5vw, 64px);
}

.container { max-width: var(--container); margin: 0 auto; padding: 0 var(--edge); }
.grid12 { display: grid; grid-template-columns: repeat(12, 1fr); column-gap: clamp(16px, 2.4vw, 28px); }
.section { padding: clamp(108px, 16vh, 184px) 0; }
```

## Color Tokens

```css
:root {
  --gray-50: #FAFAFA;  --gray-100: #F4F4F5; --gray-200: #E4E4E7;
  --gray-300: #D4D4D8; --gray-400: #A1A1AA; --gray-500: #71717A;
  --gray-600: #52525B; --gray-700: #3F3F46; --gray-800: #27272A;
  --gray-900: #18181B; --gray-950: #0A0A0A;

  --bg: #FFFFFF;
  --bg-soft: var(--gray-50);
  --ink: #0A0A0A;
  --muted: var(--gray-600);
  --muted-2: var(--gray-700);
  --line: var(--gray-200);
  --line-strong: var(--gray-300);
  --toggle-line: var(--gray-500);
  --toggle-active: var(--gray-100);
}

:root[data-theme="dark"] {
  color-scheme: dark;
  --gray-50: #F4F4F5;  --gray-100: #E4E4E7; --gray-200: #D4D4D8;
  --gray-300: #A1A1AA; --gray-400: #71717A; --gray-500: #52525B;
  --gray-600: #3F3F46; --gray-700: #27272A; --gray-800: #18181B;
  --gray-900: #0C0C0F; --gray-950: #08080A;

  --bg: #0C0C0F;
  --bg-soft: #18181B;
  --ink: #F4F4F5;
  --muted: #D4D4D8;
  --muted-2: #A1A1AA;
  --line: #3F3F46;
  --line-strong: #52525B;
  --toggle-line: #71717A;
  --toggle-active: #27272A;
}
```

## Halftone Dot Texture

Color always follows `var(--ink)`, so it adapts automatically per
theme. Opacity stays in a `.06`–`.09` range. Each placement uses a
unique position, size, and edge-fade mask — never repeat the exact
same placement across two different page types.

```css
.hero::before {
  content: "";
  position: absolute;
  z-index: 0;
  top: calc(var(--nav-h) + 34px);
  right: clamp(40px, 8vw, 112px);
  width: clamp(150px, 18vw, 220px);
  height: 190px;
  pointer-events: none;
  background-image: radial-gradient(circle, var(--ink) 1.1px, transparent 1.35px);
  background-size: 8px 8px;
  opacity: .09;
  mask-image: radial-gradient(ellipse at 70% 18%, #000 0%, rgba(0,0,0,.78) 28%, transparent 78%);
}

@media (max-width: 720px) {
  .hero::before {
    top: calc(var(--nav-h) + 4px);
    right: 18px;
    width: 88px;
    height: 62px;
    background-size: 7px 7px;
    mask-image: radial-gradient(ellipse at 80% 18%, #000 0%, transparent 76%);
  }
}
```

```html
<section class="hero"><div class="hero-stage"></div></section>
```

Reference placements used across this system (vary per surface):

| Surface | Spacing | Opacity | Fade origin |
|---|---|---|---|
| Hero | 8px | .09 | 70% 18% |
| Journal list | 10px | .07 | 20% 25% |
| Detail page | 8px | .08 | 80% 12% |
| Project page | 12px | .07 | 72% 48% |
| Nav background | 9px | .06 | 60% 60% |

## Theme Toggle + Circular Reveal Transition

3-way segmented control (system / light / dark), each option an icon
button reflecting state via `aria-pressed`.

```tsx
import { Monitor, Moon, Sun } from "lucide-react";

type ThemePreference = "system" | "light" | "dark";

const options: Array<{ value: ThemePreference; label: string; Icon: typeof Monitor }> = [
  { value: "system", label: "Use system theme", Icon: Monitor },
  { value: "light", label: "Use light theme", Icon: Sun },
  { value: "dark", label: "Use dark theme", Icon: Moon },
];

function ThemeToggle() {
  const { theme, setTheme } = useTheme();
  return (
    <div className="theme-toggle-group" role="group" aria-label="Theme preference">
      {options.map(({ value, label, Icon }) => (
        <button
          key={value}
          className="theme-toggle-option"
          aria-label={label}
          aria-pressed={theme === value}
          onClick={(e) => setTheme(value, e.currentTarget)}
        >
          <Icon aria-hidden="true" />
        </button>
      ))}
    </div>
  );
}
```

```css
.theme-toggle-group {
  display: inline-flex;
  gap: 2px;
  padding: 3px;
  border: 1px solid var(--toggle-line);
  border-radius: 6px;
  background: transparent;
  color: var(--muted);
}
.theme-toggle-option {
  width: 22px; height: 22px;
  border-radius: 100px;
  border: none; background: none;
  display: flex; align-items: center; justify-content: center;
  transition: background .2s var(--ease), color .2s var(--ease);
}
.theme-toggle-option[aria-pressed="true"] {
  background: var(--toggle-active);
  color: var(--ink);
}
.theme-toggle-option svg { width: 12px; height: 12px; stroke-width: 1.6; }
```

Circular reveal transition — origin locks to the clicked toggle's
center, expands to cover the farthest screen corner:

```tsx
function getThemeTransitionOrigin(toggle?: Element | null) {
  const bounds = toggle?.getBoundingClientRect();
  const x = bounds ? bounds.left + bounds.width / 2 : innerWidth / 2;
  const y = bounds ? bounds.top + bounds.height / 2 : innerHeight / 2;
  const radius = Math.hypot(Math.max(x, innerWidth - x), Math.max(y, innerHeight - y));
  return { x, y, radius };
}

function applyTheme(preference: ThemePreference, toggle?: Element | null) {
  const root = document.documentElement;
  const resolved = preference === "system"
    ? (matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light")
    : preference;
  const reduced = matchMedia("(prefers-reduced-motion: reduce)").matches;

  const swap = () => {
    root.dataset.theme = resolved;
    root.classList.toggle("dark", resolved === "dark");
  };

  if (!reduced && document.startViewTransition) {
    const { x, y, radius } = getThemeTransitionOrigin(toggle);
    root.style.setProperty("--theme-transition-x", `${x}px`);
    root.style.setProperty("--theme-transition-y", `${y}px`);
    root.style.setProperty("--theme-transition-radius", `${radius}px`);
    document.startViewTransition(swap);
  } else {
    swap();
  }
}
```

```css
::view-transition-old(root), ::view-transition-new(root) { animation: none; mix-blend-mode: normal; }
::view-transition-new(root) {
  z-index: 2;
  clip-path: circle(0px at var(--theme-transition-x, 50%) var(--theme-transition-y, 50%));
  animation: theme-reveal 600ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
}
@keyframes theme-reveal {
  to { clip-path: circle(var(--theme-transition-radius, 150vmax) at var(--theme-transition-x, 50%) var(--theme-transition-y, 50%)); }
}
@media (prefers-reduced-motion: reduce) {
  ::view-transition-new(root) { animation: none; clip-path: none; }
}
```

## Motion

```css
:root {
  --ease: cubic-bezier(.22, .61, .36, 1);
  --ease-soft: cubic-bezier(.16, .84, .44, 1);
}
html { scroll-behavior: smooth; }

.reveal {
  opacity: 0;
  transform: translateY(26px);
  transition: opacity .72s var(--ease-soft), transform .72s var(--ease-soft);
}
.reveal.is-in-view { opacity: 1; transform: none; }
.reveal-left { transform: translateX(-28px); }
.reveal-right { transform: translateX(28px); }
```

Staggered entrances: second item `transition-delay: .08s`, third item `.16s`.

Respect reduced motion globally:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: .01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: .01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## Rules

- No accent/hue colors, anywhere. Emphasis is inversion only.
- Halftone dots stay in the .06–.09 opacity range; never opaque enough to compete with text.
- Never repeat the identical dot placement/mask on two different page types — vary position and fade per surface.
- The theme transition duration and easing (600ms, cubic-bezier(0.16, 1, 0.3, 1)) are fixed — don't shorten for "snappiness."
- Transition origin is always the toggle's own center via getBoundingClientRect(), never a fixed point or the raw click event.
- Always guard with prefers-reduced-motion — skip straight to instant swap.
- Always persist theme choice and default to system preference on first load.