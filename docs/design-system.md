# Design System Guidelines

This document details the colors, radii standards, shadows, typography, and spacing variables of our premium furniture design system.

---

## 1. Color Palettes (Semantic Mapping)

All CSS colors must refer to semantic CSS variables rather than hardcoded tailwind values (e.g. `text-stone-900` or `bg-white`).

### Light Mode Tones
- `bg-background`: Warm Ivory / Stone (`#faf8f6`)
- `bg-card`: Pure white container background (`#ffffff`)
- `bg-surface`: Soft background layout fills (`#f5f2ee`)
- `border-border`: Elegant light border (`#eae6e2`)
- `hover-accent`: Soft walnut tone for list item hover (`#ebdcd0`)

### Dark Mode Tones
- `bg-background`: Natural deep ebony (`#141211`)
- `bg-card`: Warm stone containers (`#1c1917`)
- `bg-surface`: Intermediate dark gray containers (`#24201e`)
- `border-border`: Subdued dark borders (`#2c2724`)
- `hover-accent`: Warm charcoal tones for item hovers (`#302926`)

### Accent Tones (Furniture Vibe)
- **Warm Walnut** (`brand-700` / `#5c3426`)
- **Oak** (`brand-300` / `#d4a994`)
- **Bronze** (`brand-500` / `#8c5a47`)
- **Gold** (`#d4af37`)
- **Amber** (`#d97706`)

---

## 2. Border Radii Standards

Ensure all UI components strictly follow these border rounding constants:
- **Buttons & Small Controls**: `12px` (`rounded-lg` / `--radius-lg`)
- **Inputs & Input Groups**: `12px` (`rounded-lg` / `--radius-lg`)
- **Cards & Outer Container blocks**: `20px` (`rounded-card` / `--radius-card`)
- **Modals, Dialogs, and Drawers**: `24px` (`rounded-2xl` / `--radius-2xl`)

---

## 3. Shadow Hierarchy

Use these semantic shadow tokens to establish visual layers:
- `shadow-flat`: Subtle 1px shadow for flat elements, inputs, and boundaries.
- `shadow-soft`: Medium shadow for standard card containers.
- `shadow-raised`: Prominent shadow for hover and active card states.
- `shadow-overlay`: Deep, dark shadow for modals, dialog overlays, and drawers.
