---
name: rovi-design
description: "UI/UX design system and visual philosophy. Auto-loads when building frontend, creating components, designing layouts, styling pages, working with CSS, animations, hero sections, or UI in React, Next.js, or Vue. Enforces smooth transitions, solid colors, rounded abstract cuts, framer-motion animations, CSS variables, and lazy loading."
---

# ROVI — Design System

## Visual Philosophy

Clean, fluid, solid. No visual noise. Every element has purpose and breathing room.

---

## Colors

- **Solid colors only.** No gradients as decoration — gradients are only for smooth transitions between sections with different backgrounds.
- **No flashy or neon colors.** Muted, professional palettes. Think dark tones, warm neutrals, subtle accents.
- **Multiple tones per color.** Define a full scale in CSS variables (50-950 or similar). Never hardcode a color anywhere.
- **Section transitions:** If one section is dark and the next is light, always add a transition element (a div with a CSS gradient) between them. No hard cuts between contrasting backgrounds.

```css
:root {
  --color-primary-50: #f0f4ff;
  --color-primary-100: #dbe4ff;
  --color-primary-500: #4c6ef5;
  --color-primary-900: #1a237e;

  --color-neutral-50: #fafafa;
  --color-neutral-100: #f5f5f5;
  --color-neutral-800: #262626;
  --color-neutral-900: #171717;
  --color-neutral-950: #0a0a0a;
}
```

---

## Typography

- **Primary font:** `Onest`
- **Secondary font:** `IBM Plex Sans`
- Define font sizes, weights, and line-heights as CSS variables. Never hardcode font properties.

```css
:root {
  --font-primary: 'Onest', sans-serif;
  --font-secondary: 'IBM Plex Sans', sans-serif;

  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --text-5xl: 3rem;
  --text-6xl: 3.75rem;
  --text-7xl: 4.5rem;
}
```

---

## Shapes & Borders

- **No perfect circles.** No `border-radius: 50%` on containers or cards.
- **Abstract corner cuts.** Use asymmetric border-radius to create unique, non-uniform shapes. Different radius per corner.
- **Innovate on shape.** Each section or card can have its own personality through abstract rounding.

```css
/* Abstract cuts — not uniform, not circular */
.card {
  border-radius: 2rem 0.5rem 2rem 0.5rem;
}

.hero-image {
  border-radius: 3rem 1rem 4rem 0.5rem;
}

.section-feature {
  border-radius: 0 2.5rem 0 2.5rem;
}
```

---

## Layout & Sections

### Navbar

- **Clean, on light background** (white or `--color-neutral-50`).
- **Logo left, navigation links center, CTA right.**
- Navigation links use geometric/clean typography, spaced evenly.
- CTA button is a solid accent color (not outlined), with rounded corners.
- Navbar sits **above** the hero — it does not overlap or blend into it.
- Minimal height, generous horizontal padding, no visual clutter.

### Hero Sections

- **Large text + visual side by side.** Two-column layout: headline + subtitle on the left, image/3D element/illustration on the right.
- Text should be big and bold. Use `text-5xl` to `text-7xl` for the main headline.
- **Dark background** that contrasts with the light navbar above.
- **Not full-width.** The hero has horizontal margin/padding so the light body background is visible on both sides. The hero looks like a dark elevated card floating over the page, not an edge-to-edge block. This reinforces the 3D depth effect.
- **Geometric depth on the hero container.** The hero should not be a flat rectangle. Use CSS `perspective`, `clip-path`, or asymmetric shapes to create a 3D beveled or cut edge — especially on the top-left or top corners — giving the illusion of depth where the navbar meets the hero.
- The visual element (image, 3D object, illustration) should have abstract border-radius, never a plain rectangle.

```css
/* Hero: contained, not full-width, with geometric 3D cut */
.hero {
  margin-inline: var(--space-6);
  clip-path: polygon(4% 8%, 100% 0%, 100% 100%, 0% 100%);
  background: var(--color-neutral-900);
  border-radius: 1.5rem;
}

/* Alternative: perspective transform for depth illusion */
.hero-container {
  margin-inline: var(--space-6);
  perspective: 1200px;
}
.hero-inner {
  transform: rotateY(-1deg) rotateX(0.5deg);
  transform-origin: center center;
  border-radius: 1.5rem;
  overflow: hidden;
}
```

### Section Transitions

- **Never a hard background color cut.** If section A is `--color-neutral-900` and section B is `--color-neutral-50`, insert a gradient transition div between them.

```css
.section-transition {
  height: 6rem;
  background: linear-gradient(to bottom, var(--from-color), var(--to-color));
}
```

---

## Animations & Motion

- **Always use `framer-motion`** (React/Next.js) or equivalent motion library.
- **Animate on viewport entry.** Every section and meaningful element should animate in as it becomes visible. Use `whileInView` with `viewport={{ once: true }}`.
- **No abrupt appearances.** Elements fade in, slide in, or scale in. Never just pop into existence.
- **Stagger children.** When multiple items appear together (cards, list items), stagger their entrance.

```tsx
<motion.div
  initial={{ opacity: 0, y: 40 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true }}
  transition={{ duration: 0.6, ease: 'easeOut' }}
>
  {content}
</motion.div>
```

```tsx
/* Staggered children */
<motion.div
  initial="hidden"
  whileInView="visible"
  viewport={{ once: true }}
  variants={{
    visible: { transition: { staggerChildren: 0.15 } },
  }}
>
  {items.map((item) => (
    <motion.div
      key={item.id}
      variants={{
        hidden: { opacity: 0, y: 30 },
        visible: { opacity: 1, y: 0 },
      }}
    />
  ))}
</motion.div>
```

---

## Smooth Scrolling & Navigation

- **`scroll-behavior: smooth`** on `html` always.
- **Lazy loading** on all images and heavy components. Use `loading="lazy"` on images, dynamic imports with `React.lazy` or `next/dynamic` for components.
- **Scroll-to-section** navigation should be fluid, never instant jumps.

```css
html {
  scroll-behavior: smooth;
}
```

---

## CSS Architecture

- **One `styles/global.css`** (or equivalent) that defines all CSS variables.
- **CSS variables for everything:** colors, fonts, sizes, spacing, border-radius, shadows. Zero hardcoded values in component styles.
- **Fully modular.** Each component has its own styles (CSS Modules, scoped styles, or styled-components). No global class soup.
- **Consistent spacing scale** via variables.

```css
:root {
  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;
  --space-16: 4rem;
  --space-24: 6rem;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);

  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-base: 300ms ease;
  --transition-slow: 500ms ease;
}
```

---

## Storybook

- **Always offer to set up Storybook** when creating UI components.
- Each component should have a `.stories.tsx` file alongside it.
- Stories should cover: default state, loading state, error state, and edge cases (empty data, long text, etc.).

---

## Modals & Overlays

- **Modals must be perfectly centered** — both horizontally and vertically. Use `position: fixed` + `inset: 0` + flexbox centering. No manual `top: 50%; transform: translateY(-50%)` hacks that break on different viewports.
- **Backdrop always present.** Semi-transparent overlay behind the modal. Click on backdrop closes the modal.
- **Content must not overflow.** If modal content is long, add `max-height` with `overflow-y: auto` on the body. The modal itself stays centered.
- **Close button always visible** — top-right corner, always accessible.

```css
/* Pattern: modal centering — always this way */
.modal-overlay {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.5);
  z-index: 50;
}

.modal-content {
  max-height: 85vh;
  overflow-y: auto;
}
```

---

## UI Quality Baseline

**No basic UI errors.** Every interface must pass these checks before being considered done:

- **Alignment:** Elements that should be aligned are aligned. No off-by-pixel text, no misaligned buttons, no uneven spacing.
- **Overflow:** Text does not overflow its container. Long text truncates with ellipsis or wraps properly.
- **Responsiveness:** Layouts do not break on common viewport sizes. Flex/grid layouts adapt correctly.
- **Spacing consistency:** Use the spacing scale (`--space-*`). No arbitrary pixel values creating visual inconsistency.
- **Interactive states:** Buttons have hover/active/disabled states. Links have hover states. Inputs have focus states.
- **Empty states:** Lists and tables have a proper empty state — not a blank void.
- **Loading states:** Async content shows a loading indicator, not a layout jump.
- **Z-index sanity:** Modals, dropdowns, and tooltips stack correctly. Nothing hidden behind something it should be above.

These are not optional polish — they are the minimum bar. An interface with these errors is not done.

---

## Hard Rules

### Never
- Use `border-radius: 50%` on containers or cards.
- Hardcode colors, fonts, or spacing in component files.
- Put emojis in the UI as decorative elements.
- Create abrupt background transitions between sections.
- Let elements appear without animation.
- Use flashy, neon, or overly saturated colors.
- Ship a modal that is not perfectly centered.
- Leave misaligned elements, overflowing text, or broken layouts.

### Always
- Define all design tokens in `global.css` as CSS variables.
- Use `framer-motion` (or equivalent) for viewport-triggered animations.
- Add gradient transition divs between contrasting sections.
- Use abstract, asymmetric border-radius — not uniform circles.
- Lazy load images and heavy components.
- Set `scroll-behavior: smooth` globally.
- Prefer `Onest` as primary font, `IBM Plex Sans` as secondary.
- Stagger animations when multiple elements enter together.
- Center modals with `position: fixed` + `inset: 0` + flexbox.
- Verify alignment, overflow, spacing, and interactive states before finishing any UI work.
