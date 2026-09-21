# Architecture & Onboarding Guidelines

Welcome to the Furnixo Enterprise Frontend Architecture. This document explains our folder structures, layout design patterns, dependencies, and code constraints.

---

## 1. Directory Structure

```
src/
├── app/                  # Next.js App Router Pages and Routes
├── assets/               # Assets Registry (images, icons, placeholders)
│   └── images/           # Categorized subdirectories (hero, products, brands, etc.)
├── components/           # Component Library
│   ├── layout/           # App Shell Layout Blocks (Header, Footer, Command Palette)
│   ├── system/           # Critical loading and fallback layers
│   └── ui/               # Reusable Atomic UI elements (Buttons, Inputs, Cards)
├── config/               # Data-only configurations (feature flags, SEO, menu datasets)
├── constants/            # Programmatic constants (routes, design tokens, motion)
├── hooks/                # Decoupled utility hooks (useHeaderState, useRecentSearches)
├── lib/                  # Shared utilities (classnames joiners, storage wrappers)
├── providers/            # React Context Providers (Theme, Motion, Layout)
├── services/             # API/Backend interaction adapters
└── types/                # Shared TypeScript structures
```

---

## 2. Dependency Direction Rules

To keep the bundle lightweight and maintainable, follow these strict rules:
- **Presentation Component Isolation**: Layout elements (like `Header` or `Footer`) must contain **only** view code. Business states, caches, and scroll measurements belong in custom Hooks (e.g. `useHeaderState.ts`).
- **Data Configuration Separation**: All config lists (links, feature flags) belong in `src/config/` as simple data declarations. Avoid importing JSX nodes or layout code into config files.
- **Import Rules**:
  - Internal sibling imports are allowed (e.g., `desktop-navigation.tsx` importing from the same folder).
  - External components must import ONLY from the root barrel `index.ts` of the target component folder. Never do deep sub-path imports (e.g., import from `@/components/layout/header` instead of `@/components/layout/header/header`).

---

## 3. Bundle Splitting & Performance Targets

We enforce strict performance limits:
- **Critical (Bundled Instantly)**: `Header`, `Logo`, `Navigation`.
- **Lazy Loaded (Code Split)**: Components loaded via Next.js `dynamic(..., { ssr: false })` to reduce First Load JS sizes.
  - `MegaMenu`
  - `CommandPalette` (Search Dialog)
  - `MobileNav` (Drawer)
- **Cumulative Layout Shift (CLS)**: Must remain under `0.05` during animations, images loading, or theme switches. Avoid height/width scaling animations.
