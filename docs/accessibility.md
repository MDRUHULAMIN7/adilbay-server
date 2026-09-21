# Accessibility (a11y) Guidelines

All components in this template must align with the WCAG 2.2 AA accessibility specifications.

---

## 1. Keyboard Navigation

- **Focus Ring Indicators**: Every interactive control must display a visible, high-contrast focus ring when focused via keyboard:
  - Class: `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`
  - Outline should have adequate contrast against light/dark background surfaces.
- **Escape Closes Modals**: Pressing the `Escape` key must immediately close drawers, command palettes, and modals.
- **Keyboard Tab Trap**: Open dialogs and drawer menus must trap focus cycling inside the overlay using helper hooks (`useFocusTrap`).

---

## 2. ARIA Names and Descriptions

- **Accessible Names**: All icon-only buttons must provide an explicit `aria-label` attribute (e.g. `aria-label="Open search dialog"`).
- **Expandable Controls**: Buttons opening drop menus, drawers, or accordion sheets must declare `aria-expanded` and `aria-controls` linked to target ID structures.
- **Current Page Indicators**: Active links in layouts must display the `aria-current="page"` attribute.

---

## 3. Structural Semantics (Landmarks)

Always include layout semantic tags:
- `role="banner"` / `<header>`: Site top wrapper.
- `role="navigation"` / `<nav>`: Layout menu sections.
- `role="contentinfo"` / `<footer>`: Site footer.
- `role="main"` / `<main>`: Core page layout body.
- **Skip Navigation**: Ensure `SkipNav` is present at the top of the body to allow users to skip layout headers.
