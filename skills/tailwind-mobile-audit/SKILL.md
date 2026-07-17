---
name: tailwind-mobile-audit
description: Audit a Tailwind CSS codebase for mobile-first integrity, touch usability, and iOS/Android hardware constraints. Use when asked to review mobile UX, responsive behavior, touch targets, or "why does this look/feel bad on my phone" in any Tailwind project.
---

# Tailwind Mobile-First Audit

Audit the codebase for mobile-first architectural integrity, touch usability, and hardware-specific constraints. Every finding must cite the file and the exact utility classes involved.

## 1. Configuration & Breakpoint Strategy

- Locate `tailwind.config.js`/`.ts` (or the CSS `@theme` block in v4).
- Verify a mobile-first (min-width) breakpoint strategy, not desktop-first (max-width).
- Flag custom screens that target specific devices (e.g. `812px`) instead of ranges.

## 2. Mobile-First Architecture Scan

Scan `app/`, `src/`, `components/` for layout utilities:

- **Desktop-default patterns:** fixed widths applied un-prefixed (`w-[500px]`) with responsive prefixes used to *shrink* (`md:w-full`). Rule: the un-prefixed class must be the mobile view (`w-full`/`w-auto`).
- **Grid complexity:** `grid-cols-3`+ without a responsive prefix. Mobile grids default to 1–2 columns.

## 3. Touch & Interaction (the 44px rule)

Scan interactive elements (`button`, `a`, `input`, `select`):

- **Hit area:** flag height/width below `h-11`/`w-11` (44px) unless compensated by significant padding (`p-3`+).
- **States:** flag `hover:` styles lacking corresponding `active:`/`focus:` styles — mobile users can't hover; they need tap feedback.

## 4. Input & Form Hygiene

- **iOS zoom:** flag any `input`/`select`/`textarea` at `text-sm` or smaller. iOS auto-zooms on inputs < 16px (`text-base`).
- **Keyboards:** flag generic `type="text"` for email/phone/numeric data — should be `type="email"`, `type="tel"`, or `inputmode="numeric"`.

## 5. Safe Area & Viewport

- **Dynamic viewport:** flag `h-screen` on root containers and full-page modals; recommend `h-dvh`/`h-svh` (mobile browser chrome expands/collapses).
- **Notch/home bar:** flag `fixed bottom-0` / `sticky top-0` elements lacking safe-area padding (`pb-safe`/`pt-safe` or `env(safe-area-inset-*)`).

## 6. Performance (mobile hardware)

- Flag `transition-all` — layout thrashing on low-end devices. Recommend `transition-transform` / `transition-opacity`.

## Output

- **Mobile Architecture Score:** qualitative assessment of mobile-first adherence.
- **"Fat Finger" Risk List:** components violating the 44px rule.
- **iOS Usability Report:** inputs risking auto-zoom or wrong keyboards.
- **Viewport Risks:** `h-screen` → `h-dvh` cases; unsafe fixed headers/footers.
- **Refactor Recommendations:** specific utility swaps per finding.
