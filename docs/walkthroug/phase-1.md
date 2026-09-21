# Tasks - Phase 1: Enterprise Design System

- [x] **Phase 1.1 — Foundation**
  - [x] Bootstrap Next.js 15 Project in current directory
  - [x] Install dependencies (`framer-motion`, `@tanstack/react-query`, `@tanstack/react-form`, `lucide-react`)
  - [x] Configure Jest and React Testing Library setup
  - [x] Set up Semantic Design Tokens in CSS under Tailwind v4 `@theme`
  - [x] Implement custom Hydration-safe, Flash-free `ThemeProvider` and transitions lock
  - [x] Implement `AppProviders` wrapper containing Theme, Query, and Motion config
  - [x] Create proxy `icon.tsx` wrapper for `lucide-react`
  - [x] Create shared types in `src/types/component.ts` (e.g. `BaseComponentProps`)
  - [x] Create shared utility hooks in `src/hooks/` (`useLocalStorage`, `useFocusTrap`, `useClickOutside`, `useMediaQuery`, `useMounted`, `useBoolean`, `useToggle`, `useDebounce`, `usePrevious`)
  - [x] Create shared helper libraries in `src/lib/` (`cn.ts`, `storage.ts`, `debounce.ts`, `throttle.ts`, `sleep.ts`, `generate-id.ts`)
  - [x] Implement styled `<Heading>`, `<Text>`, and `<Code>` typography components
  - [x] Pre-create empty folders for future scalability (`services`, `mock`, `assets`, `config`)
  - [x] Verify setup via Jest execution and Next.js build compilation
- [x] **Phase 1.2 — Basic Components**
  - [x] Button
  - [x] Input & Textarea
  - [x] Badge
  - [x] Avatar
  - [x] Spinner & Skeleton
  - [x] Card
  - [x] Breadcrumb
- [x] **Phase 1.3 — Form Components**
  - [x] Checkbox
  - [x] Radio
  - [x] Switch
  - [x] Native Select Wrapper
- [x] **Phase 1.4 — Overlay Components**
  - [x] Modal
  - [x] Drawer
  - [x] Tooltip
  - [x] Toast
- [x] **Phase 1.5 — Navigation, Display & Polish**
  - [x] Tabs
  - [x] Accordion
  - [x] Pagination
  - [x] Error & Loading Boundaries
  - [x] Playground sub-routes
  - [x] Design System README
# Implementation Plan (Final RFC) — Phase 1: Enterprise Design System

This plan details the phased execution of **Phase 1** for the **Furnixo** e-commerce frontend. The scope is broken down into small, logical sub-phases to ensure high code quality, robust unit tests, and seamless review.

---

## Architectural & Coding Standards

### Project Structure Rules
- **Feature Isolation**: Feature modules must never import from other feature modules.
- **Shared Availability**: Shared modules (`components/ui`, `hooks`, `lib`, etc.) may be imported anywhere.
- **Path Aliases**: Imports must always use the `@/*` alias (no deep relative imports like `../../../`).
- **Circular Imports**: Circular dependencies are strictly prohibited.

### Dependency Direction Flow
Lower levels cannot import from higher levels:
```
app (Routing & Layouts)
   ↓
features (Business Page Features)
   ↓
components (UI & Common Layout Elements)
   ↓
hooks (Stateful React Hooks)
   ↓
lib (Stateless Helpers & Utilities)
   ↓
types (Typescript Declarations)
```

### Naming Conventions
- **Files & Directories**: `kebab-case` (e.g., `theme-provider.tsx`, `use-local-storage.ts`)
- **Hooks**: `camelCase` (e.g., `useLocalStorage`, `useFocusTrap`, `useClickOutside`, `useMediaQuery`, `useMounted`, `useBoolean`, `useToggle`, `useDebounce`, `usePrevious`)
- **Components**: `PascalCase` (e.g., `Button`, `CardHeader`)
- **Types & Interfaces**: `PascalCase` (e.g., `ButtonProps`)
- **Constants**: `UPPER_CASE` (e.g., `MOTION_PRESETS`, `THEME_CONFIG`)

### Component Location Rule
- **Simple Components**: Implemented directly as files (e.g., `src/components/ui/button.tsx`) with their test file adjacent (e.g., `src/components/ui/button.test.tsx`).
- **Complex Components**: Implemented in folders (e.g., `src/components/ui/modal/`) containing:
  - `index.ts` (Barrel export)
  - `modal.tsx` (Component logic)
  - `modal.test.tsx` (Adjacent tests)
  - `modal.types.ts` (Specific typings)
  - `modal.styles.ts` (Variant/style definitions, optional)

### Future-Proof Component Rules
- Never import business modules.
- Never import feature modules.
- UI components may depend only on:
  - React (and React standard APIs)
  - Shared utilities (`src/lib/*`)
  - Shared hooks (`src/hooks/*`)
  - Icon Wrapper (`src/components/ui/icon.tsx`)
  - Motion presets and framer-motion library

---

## Git Standards & Conventions
We will structure commit messages strictly using Conventional Commits guidelines:
- `feat: [message]` — Initial implementation of a component or setup
- `fix: [message]` — Resolving accessibility gaps, styling glitches, or bugs
- `refactor: [message]` — Code cleanup, separating files, improving hooks
- `test: [message]` — Adding or updating Jest/RTL coverage
- `docs: [message]` — Updates to READMEs or JSDoc comments
- `chore: [message]` — Tooling configuration updates or dependencies

---

## Component API Standards

All components must extend standard attributes. Shared interfaces are defined in `src/types/component.ts`:
```typescript
export interface BaseComponentProps {
  className?: string;
  children?: React.ReactNode;
  id?: string;
  "data-testid"?: string;
}
```

Components extending this standard must expose stable forward references (`React.forwardRef`) where applicable, handle ARIA properties gracefully, and preserve custom element interfaces.

---

## Design System Presets & Constants

### Centralized CSS Variables
We will map semantic tokens inside `@theme` in `src/app/globals.css`:
- **Theme Mappings**:
  - `bg-background`: Main background colors.
  - `text-foreground`: Primary text color.
  - `bg-primary`, `bg-secondary`, `bg-muted`, `bg-accent`, `bg-destructive`.
  - Border variables and focus ring definitions (`focus-visible`).

### Motion Guideline (Constants)
To ensure uniform animation behavior, `src/constants/motion.ts` will expose:
```typescript
export const DURATIONS = {
  hover: 0.15,
  press: 0.1,
  accordion: 0.2,
  modal: 0.25,
  drawer: 0.35,
  page: 0.3,
};

export const EASING = {
  standard: [0.4, 0, 0.2, 1], // ease-in-out
  decelerate: [0.0, 0, 0.2, 1], // ease-out
  accelerate: [0.4, 0, 1, 1], // ease-in
};

export const SPRINGS = {
  normal: { type: "spring", stiffness: 300, damping: 25, mass: 1 },
  bounce: { type: "spring", stiffness: 400, damping: 15, mass: 1 },
  slow: { type: "spring", stiffness: 100, damping: 15, mass: 1 },
};

export const MOTION_PRESETS = {
  fade: {
    initial: { opacity: 0 },
    animate: { opacity: 1 },
    exit: { opacity: 0 },
  },
  scaleUp: {
    initial: { opacity: 0, scale: 0.95 },
    animate: { opacity: 1, scale: 1 },
    exit: { opacity: 0, scale: 0.95 },
  }
};
```

---

## Accessibility Standards
Every component must fully comply with:
- **WCAG 2.2 AA Guidelines**
- **Semantic HTML**: Appropriate tag selection (`<button>`, `<main>`, `<nav>`, etc.).
- **Keyboard-first Navigation**: Tab indices, visual focus rings (`focus-visible:ring-2`), and triggers (Space, Enter, Esc, Arrow Keys).
- **Screen Reader Friendly**: Proper ARIA states (`aria-expanded`, `aria-checked`, `aria-describedby`).
- **Reduced Motion Support**: Disables animations if `prefers-reduced-motion: reduce` is configured.
- **High Contrast Compatible**: Border contrasts conform to minimum limits.

---

## Performance Rules
- Utilise `React.memo` only when justified by parent re-renders.
- Wrap computation logic in `useMemo` and callbacks in `useCallback` only when measured and needed.
- Lazy load overlays where possible to decrease bundle weights.
- Maintain stable references (avoid inline array/object creations inside components).
- Enforce stable `keys` and `refs` mappings.

---

## 1. Incremental Phased Scope

### Phase 1.1 — Foundation (Current Sub-phase)
* **Project Setup**: Setup Next.js App Router (v15), react-query, react-form, typescript (strict), eslint, and jest.
* **Semantic Design Tokens**: CSS bindings under Tailwind CSS v4 `@theme`.
* **Theme System**: Hydration-safe `ThemeProvider` + transition locking on load.
* **Global Providers**: Composed `AppProviders` containing:
  - `ThemeProvider`
  - `QueryClientProvider`
  - `ToastProvider`
  - `MotionProvider`
* **Typography**: Implement styled `<Heading>`, `<Text>`, and `<Code>` components.
* **Icon Wrapper**: Proxy `src/components/ui/icon.tsx` using `lucide-react`.

### Phase 1.2 — Basic Components
* **Button**: 11 variants (Primary, Secondary, Outline, Ghost, Danger, Success, Warning, Brand, Soft, Icon, Link), 5 sizes, loading states, and icon layouts.
* **Input & Textarea**: Native elements styled with validation states, helpers, and auto-resize textareas.
* **Badge**: Colored status indicator.
* **Avatar**: Circular imagery with fallback initials and status indicator.
* **Spinner & Skeleton**: Loader controls.
* **Card**: Compound styling (`Card`, `CardHeader`, `CardTitle`, `CardDescription`, `CardContent`, `CardFooter`).
* **Breadcrumb**: Path list helper.

### Phase 1.3 — Form Components
* **Checkbox**: Custom SVG mark, screen-reader friendly, accessible.
* **Radio**: Custom listbox choices.
* **Switch**: Frame Motion animated toggle slider.
* **Native Select**: Highly accessible standard native select wrapper (`<Select />`) to ensure total mobile reliability and layout compatibility.

### Phase 1.4 — Overlay Components
* **Modal**: Portal container, focus trapping, scrolling block, Esc/Click-outside dismiss, Framer Motion scale-up fade animations. Compound: `Modal`, `ModalHeader`, `ModalBody`, `ModalFooter`.
* **Drawer**: Sliders (Left/Right drawer panels), focus trapping, screen overlay.
* **Tooltip**: Focus-trigger and hover-trigger element positions.
* **Toast**: Hook `useToast()` to push alerts. Animated stack overlays.

### Phase 1.5 — Navigation, Display, & Polish
* **Tabs**: Accessible switching with Framer Motion selection indicator layout, keyboard arrow keys navigation.
* **Accordion**: Context-based expander utilizing Framer Motion layout animations for smooth heights.
* **Pagination**: Prev/Next, page numbers, ellipsis, active index styles.
* **Playground Subdivisions**: Category paths:
  - `/playground/components`
  - `/playground/forms`
  - `/playground/navigation`
  - `/playground/feedback`
  - `/playground/layout`
  - `/playground/motion`
* **Error & Loading Boundaries**: Create `components/system/error-boundary.tsx`, `components/system/loading-boundary.tsx` and `components/system/not-found.tsx`.
* **Component README**: Document folder specifications, standard structures, and instructions.

---

## Testing Standards & Requirements

### Testing Target:
- Cover critical user interactions.
- Cover accessibility behaviors.
- Cover edge cases.
- Cover component variants.
- Cover disabled/loading states.

### Coverage Targets:
- $\ge 90\%$ statements.
- $\ge 90\%$ branches for shared utilities.
- Focus on behavior over implementation details.

---

## Definition of Done (DoD)
- `[ ]` All planned components implemented.
- `[ ]` All components documented (JSDoc and Props specs).
- `[ ]` Unit tests passing successfully.
- `[ ]` ESLint static checks pass without errors.
- `[ ]` TypeScript build compiles successfully.
- `[ ]` No console warnings/errors produced during checks.
- `[ ]` Responsive compliance verified (Mobile, Tablet, Desktop).
- `[ ]` Dark mode accessibility verified.
- `[ ]` Keyboard accessibility and screen reader testing passed.
- `[ ]` Playground routes successfully demonstrate every component.
- `[ ]` Shared providers successfully integrated.
- `[ ]` Zero business logic included in Phase 1 modules.

---

## Verification & CI Commands

- **Build Check**: `npm.cmd run build`
- **Lint Check**: `npm.cmd run lint`
- **Test Check**: `npm.cmd run test`
- **TypeScript Strict Compilation**: `npx.cmd tsc --noEmit`

---

## AI Implementation Rules
- Implement only the current sub-phase.
- Do not create placeholder components for future phases.
- Do not introduce new dependencies unless requested.
- Do not refactor completed phases unless explicitly instructed.
- Keep commits focused on a single concern.
- Explain architectural decisions before major changes.
- Prefer composition over inheritance.
- Avoid premature optimization.
- Keep APIs stable across sub-phases.
