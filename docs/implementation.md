

# Frontend Implementation Roadmap

```
Phase 0
Project Foundation

↓

Phase 1
Design System

↓

Phase 2
App Shell

↓

Phase 3
Homepage

↓

Phase 4
Shop / Product Listing

↓

Phase 5
Product Details

↓

Phase 6
Authentication

↓

Phase 7
Cart

↓

Phase 8
Checkout

↓

Phase 9
Animations

↓

Phase 10
Testing

↓

Phase 11
Optimization & Polish
```

---

# Phase 0 — Project Foundation

এটা শুধু project setup.

### Goal

Create enterprise level scalable architecture.

---

### Deliverables

```
Next.js App Router

TypeScript Strict

Tailwind CSS v4

ESLint

Prettier

Absolute Imports

Path Alias

Environment Config

TanStack Query

TanStack Form

Framer Motion

Jest

RTL

Folder Structure

Mock API Layer

JSON Database

Reusable Utils

Theme Provider

Error Boundary

Loading Boundary

```

---

### Folder Structure

```
src/

app/

components/

features/

hooks/

lib/

providers/

services/

mock/

types/

constants/

utils/

styles/

```

---

### Mock Structure

```
mock/

products.json

categories.json

users.json

orders.json

```

---

### Output

Project compile without error.

---

# Phase 1 — Design System

এখান থেকেই পুরো UI শুরু হবে।

---

### Goal

Reusable UI Library

---

### Components

```
Button

Input

Textarea

Checkbox

Radio

Switch

Select

Badge

Avatar

Card

Product Card

Rating

Price

Breadcrumb

Tabs

Accordion

Modal

Drawer

Tooltip

Popover

Pagination

Skeleton

Toast

Loader

```

---

### Requirements

Every component

✔ Accessible

✔ Dark mode ready

✔ Fully typed

✔ Variant support

✔ Size support

✔ Loading state

✔ Disabled state

---

### Output

Component playground page

```
/playground
```

---

# Phase 2 — App Shell

---

### Goal

Global Layout

---

### Build

Navbar

Mega Menu

Search

Mobile Drawer

Sticky Header

Footer

Newsletter

Theme Toggle

Profile Menu

Cart Badge

Breadcrumb

Container Layout

---

### Output

Entire application shell complete.

---

# Phase 3 — Homepage

---

### Build

Hero

Carousel

Categories

Shop Room

New Arrival

Popular Products

Testimonials

Newsletter

Brand Logos

---

### Framer Motion

Fade

Slide

Scale

Stagger

Hover

---

### Output

Homepage Complete

---

# Phase 4 — Product Listing

Biggest Phase.

---

### Build

Product Grid

Sidebar Filter

Search

Price Slider

Color Filter

Brand Filter

Stock Filter

Category Filter

Sorting

Pagination

Loading

Empty State

Error State

---

### TanStack Query

Mock API

Caching

Search Params

Refetch

---

### URL

```
/shop

?brand=...

?color=...

?page=2

?sort=price

```

---

### Output

Production-ready Shop page.

---

# Phase 5 — Product Details

---

### Build

Gallery

Thumbnail

Zoom

Variant

Quantity

Wishlist

Share

Reviews

Description

Specifications

Related Products

Sticky Buy Box

---

### Output

Complete PDP

---

# Phase 6 — Authentication

---

### Pages

Login

Register

Forgot Password

Reset Password

Profile

---

### TanStack Form

Validation

Error

Loading

Success

Mock JWT

Protected Route

---

### Output

Authentication Flow

---

# Phase 7 — Cart

---

### Build

Cart Drawer

Cart Page

Quantity

Coupon

Shipping Progress

Order Summary

Remove

Clear Cart

Save Later

---

### Output

Shopping Cart

---

# Phase 8 — Checkout

---

### Build

Shipping

Billing

Payment

Review

Success

Order History

Mock Order

---

### Output

Checkout Flow

---

# Phase 9 — Animation

---

### Apply Motion

Page Transition

Card Hover

Navbar

Drawer

Modal

Tabs

Accordion

Hero

Image Gallery

Loading

Skeleton

Button Ripple

---

### Output

Premium UI Experience

---

# Phase 10 — Testing

---

### Jest

Utilities

Components

Cart

Forms

Product Card

Hooks

---

### RTL

Render

Click

Submit

Validation

Navigation

---

### Output

Good Test Coverage

---

# Phase 11 — Optimization

---

### Performance

Lazy Loading

Dynamic Import

Memo

Image Optimization

Bundle Split

SEO

Metadata

OpenGraph

Sitemap

Robots

Accessibility

Lighthouse

---

### Output

Production Ready

---

# প্রতিটি Phase-এর Prompt Structure

Antigravity-কে প্রতিবার একই ধরনের structured prompt দিলে consistency বজায় থাকবে।

```text
You are a senior Frontend Architect and Staff React Engineer.

Implement ONLY the current phase.

Do not modify features outside this phase.

Tech Stack

- Next.js App Router
- TypeScript Strict
- Tailwind CSS v4
- TanStack Query
- TanStack Form
- Framer Motion
- Jest
- React Testing Library

Requirements

- Follow Feature-Based Architecture
- Keep components reusable
- Follow SOLID principles
- Strict typing
- Clean code
- Responsive
- Accessible (ARIA)
- Production-ready
- Mock-data based
- No backend integration
- Use local JSON + service layer
- Avoid code duplication
- Keep folder structure scalable
- Export reusable hooks/types
- Add loading, empty, and error states where applicable

Deliverables

- Implement ONLY this phase.
- Explain created folders.
- Explain component hierarchy.
- Explain data flow.
- Explain reusable utilities.
- List completed checklist.
- Mention anything intentionally deferred to later phases.

Do not start the next phase automatically.
```

## আরও ভালো workflow

প্রতিটি phase-কে আরও ছোট **sub-phase**-এ ভাগ করলে AI-এর output আরও নির্ভুল হবে। যেমন, **Phase 4 (Product Listing)** একাই ৫–৬টি prompt হতে পারে:

1. Shop page layout + routing
2. Product grid + ProductCard
3. Filter sidebar + URL sync
4. Sorting + Pagination
5. TanStack Query + mock service integration
6. Loading, Empty, Error states + responsive polish

