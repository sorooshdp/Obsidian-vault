---
created: 2026-11-24 14:51
updated: 2026-11-24 14:51
type: note
status: seed
topic:
tags:
  - web/javascript
aliases: []
---

## Summary


## Notes


## References

## TypeScript & React
- [ ] **Use `readonly` for component prop types** — e.g., `Readonly<{ title: string }>` to prevent accidental mutation
- [ ] **Use ternary with `null` for conditional rendering** — avoids accidentally rendering `0` or `NaN` with `&&`
- [ ] **Use stable, unique IDs for React keys** — avoid array indices unless the list is completely static
- [ ] **Clean up side effects and event listeners** — return cleanup functions in `useEffect` to prevent memory leaks

## Styling & Markup
- [ ] **Use `cn()` instead of template literals for classes** — ensures clean conditional classes and proper Tailwind merging
- [ ] **Verify responsive design across breakpoints** — check mobile, tablet, and desktop layouts for overflow
- [ ] **Use semantic HTML elements** — prefer `<main>`, `<nav>`, `<article>`, `<section>`, and `<header>` over generic `<div>`s

## Accessibility & Navigation
- [ ] **Maintain strict heading hierarchy** — single `<h1>` per page, followed sequentially by `<h2>` and `<h3>`
- [ ] **Add meaningful `alt` text to images** — use `alt=""` for purely decorative images
- [ ] **Use `<button>` for actions and `<a>` / `<Link>` for navigation** — never use anchor tags without `href` to trigger JS functions
- [ ] **Use `<Link>` for standard navigation, `useRouter` for callbacks** — prefer `<Link>` for prefetching; reserve `router.push` for post-submit redirects

## Next.js & i18n
- [ ] **Call `setRequestLocale` on each page/layout (`next-intl`)** — required for static rendering compatibility in App Router
- [ ] **Verify Server vs. Client Component boundaries** — keep `'use client'` as low in the component tree as possible

## Verification & Hygiene
- [ ] **Run production build and linter locally** — run `npm run build` / `tsc --noEmit` to catch strict type or SSR errors
- [ ] **Remove dead code, `console.log`s, and check `.env`** — review `git diff` before pushing to ensure no secrets or debug logs leak
