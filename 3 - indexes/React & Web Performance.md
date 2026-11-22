---
created: 2026-11-10 13:54
updated: 2026-11-10 13:54
type: moc
status: mature
tags:
  - react
  - performance
aliases:
  - Web Performance MOC
  - React MOC
---

# React Internals & Web Performance Hub

## 1. Browser Rendering & Main Thread Architecture
- [[Browser Functionality and Web Performance]] — DNS/TCP/TLS, DOM/CSSOM parsing, layout (reflow), paint, and compositing
- [[Browser's main thread]] — The 16.6ms frame budget, task splitting (`scheduler.yield`), batching, Web Workers, and FLIP
- [[CSS selector performance]] — Style recalculation and right-to-left selector matching

## 2. Core Web Vitals & SEO
- [[Core Web Vitals]] — 75th percentile thresholds, CLS, SSR hydration, and RUM optimization strategies
- [[Largest Contentful Paint (LCP)]] — Viewport element heuristics and perceived load speed
- [[Interaction to Next Paint (INP)]] — Input latency across clicks, taps, and key presses
- [[Metadata]] — HTML `<head>`, Open Graph tags, and search indexing
- [[Json-LD]] — Linked Data schemas (`@context`, `@type`) and Knowledge Graph entities

## 3. React Internals & Concurrent Rendering
- [[The heart of React]] — Virtual DOM reconciliation and scheduling priority (`rAF` vs. `requestIdleCallback`)
- [[React fiber]] — Fiber tree architecture, double buffering (`current` vs. `workInProgress`), Render vs. Commit phases
- [[React hooks]] — Internal hook array indexing, call-order rules, and synchronization lifecycles
- [[useEffect]] — Setup/cleanup mechanics, reactive dependencies, and client-side fetching pitfalls
- [[React batching]] — Automatic state update batching in React 18+
- [[React compiler]] — Automatic fine-grained memoization replacing manual `useMemo`/`useCallback`