# Theming Guidelines

This template supports Light and Dark color themes using standard CSS custom properties.

---

## 1. Theme Selection Chain

Themes resolve automatically according to this priority:
1. **User Selection**: Toggled manually by user theme controllers.
2. **Stored Cache**: Saved preferences queried using `storage.ts` wrapper.
3. **System Settings**: Fallback browser preference (`prefers-color-scheme`).
4. **Default Light**: Stone/Ivory light theme.

---

## 2. Preventing Transition Flashes

To prevent visual layout flashes or delays on server-side rendering (SSR) loads:
- A script is injected inside the `<head>` in `layout.tsx` to read the cached preference synchronous with the first HTML paint.
- Theme Provider adds `disable-transitions` class to the root `<html>` tag temporarily during the toggle event. This blocks transition animations on load and toggling, preventing delayed icon paint and flashing colors.

---

## 3. Transition Animators

Theme animations must execute smoothly. Enforce these constraints:
- Set body and elements transition rules:
  ```css
  transition-property: background-color, border-color, color, fill, stroke, box-shadow;
  transition-duration: 150ms;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  ```
- **CRITICAL**: Never transition dimensions (width, height, paddings, margins) during theme switches to prevent layout shifts.
