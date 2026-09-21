# Component Creation Guidelines

This document outlines standard practices for creating, structuring, and exporting reusable layout and UI components.

---

## 1. Directory Structure per Component

Always group related assets into a single subdirectory:
```
component-name/
├── index.ts               # Public barrel API
├── component-name.tsx     # Presentation/render code
├── useComponentState.ts   # Local state hook (optional)
└── component-name.test.tsx# Unit tests
```

---

## 2. Public API and Barrel Exports

- Keep exports explicit and clean. Avoid exporting default values:
  ```typescript
  // YES
  export function Button() { ... }
  
  // NO
  export default function Button() { ... }
  ```
- The folder's `index.ts` file acts as the public entry point:
  ```typescript
  // components/ui/button/index.ts
  export * from './button';
  ```
- External modules must import ONLY from the barrel file:
  ```typescript
  // YES
  import { Button } from '@/components/ui/button';
  
  // NO
  import { Button } from '@/components/ui/button/button';
  ```

---

## 3. Separation of Logic and Presentation

- UI components should remain presentation-only. Offload complex hook behaviors, scroll states, local caches, and API calls to separate files (e.g. `useHeaderState.ts`).
- This makes components easily testable using standard mocks and helps avoid hydration issues or unexpected layout shifts in React Server Components (RSC) and client transitions.
