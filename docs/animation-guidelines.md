# Animation & Motion Guidelines

We enforce a consistent visual language. All transitions and hover states must remain subtle and elegant, avoiding excessive bounces or rapid size changes.

---

## 1. Unified Duration Scales

- **Hover Micro-interactions**: `150ms` (standard ease).
- **Dropdowns & Navigation Menus**: `180ms` (Framer spring or standard ease).
- **Dialogs & Modals**: `220ms` (spring scale transitions).
- **Drawers (Slide-ins)**: `250ms` (spring slide transitions).
- **Tooltips**: `120ms` (quick ease).

---

## 2. Spring Constant Registry

Define standard spring constants for animations in `src/constants/motion.ts`:
- **Normal Spring** (Standard panels and components):
  - Stiffness: `300`
  - Damping: `25`
  - Mass: `1`
- **Bounce Spring** (Interactive toggles):
  - Stiffness: `400`
  - Damping: `15`
  - Mass: `1`
- **Slow Spring** (Large drawers / screens transition):
  - Stiffness: `100`
  - Damping: `15`
  - Mass: `1`

---

## 3. Approved Framer Motion Variants

Avoid custom inline frames. Utilize global variants defined in `src/constants/motion.ts`:
- `fade`: standard opacity overlay.
- `slideInLeft` / `slideInRight`: drawer panels transition.
- `scaleUp`: modal container animation.
- `dropdown`: popup menu reveal.
