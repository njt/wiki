---
url: https://www.patterns.dev/
title: Patterns.dev — Modern Design, Rendering, and Performance Patterns for Web Apps
author: Lydia Hallie, Addy Osmani
date_fetched: 2026-05-14
date_published: unknown
topics:
  - software-engineering-craft
---

# Patterns.dev

A free online resource on design, rendering, and performance patterns for building powerful web apps with vanilla JavaScript or modern frameworks like React, Next.js, and Vue.js.

Creators: Lydia Hallie (software engineering consultant and educator) and Addy Osmani (engineering manager at Google Chrome, leads Lighthouse/PageSpeed Insights/Chrome UX Report teams).

Contributors: Josh W. Comeau (whimsical UX), Anton Karlovskiy (software engineer), Leena Sohoni-Kasture (writer & editing), Nadia Snopek (illustrator), Hassan Djirdeh (software engineer & writer).

License: Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0).

Tagline: "We enable developers to build amazing things."

## Philosophy

Patterns.dev positions design patterns as "descriptive, not prescriptive" — they guide developers facing common problems but aren't meant to be forced into every scenario. The site aims to be "a catalog of patterns (for increasing awareness) rather than a checklist."

Features: practical CodeSandbox examples, animated visual explanations, and a downloadable eBook/PDF.

## Content Structure

### JavaScript Patterns (Vanilla JS & Node.js)

**Design Patterns:**
- Singleton Pattern — Share a single global instance
- Proxy Pattern — Intercept and control interactions to target objects
- Prototype Pattern — Share properties among many objects of the same type
- Observer Pattern — Use observables to notify subscribers
- Module Pattern — Split code into smaller, reusable pieces
- Mixin Pattern — Add functionality without inheritance
- Mediator/Middleware Pattern — Central mediator for component communication
- Flyweight Pattern — Reuse existing instances for identical objects
- Factory Pattern — Factory functions to create objects

**Performance & Rendering Patterns:**
- Animating View Transitions (View Transitions API)
- Optimize loading sequence
- Static Import / Dynamic Import
- Import On Visibility — Load when visible in viewport
- Import On Interaction — Load on user interaction
- Route Based Splitting — Dynamic loading by route
- Bundle Splitting — Small, reusable pieces
- PRPL Pattern — Precache, lazy load, minimize roundtrips
- Tree Shaking — Eliminate dead code
- Preload / Prefetch — Critical resource hints
- Optimize loading third-parties
- List Virtualization — Optimize list rendering performance
- Compressing JavaScript

### React Patterns (React & Next.js)

**Design Patterns:**
- Container/Presentational Pattern — Separate view from logic
- Higher-Order Component (HOC) Pattern — Pass reusable logic as props
- Render Props Pattern — Pass JSX elements through props
- Hooks Pattern — Reuse stateful logic via functions
- Compound Pattern — Multiple components working together

**Rendering Patterns:**
- Client-side Rendering (CSR)
- Server-side Rendering (SSR)
- Static Rendering (SSG)
- Incremental Static Generation (ISR)
- Progressive Hydration — Delay JS for less important parts
- Streaming Server-Side Rendering
- React Server Components — Render without adding to JS bundle

**Performance:**
- Optimize Next.js for Core Web Vitals
- React Stack Patterns (2025/2026) — Frameworks, build tools, routing, state management, AI integration

### Vue Patterns (Vue.js)

- Components, Async Components, Composables
- Container/Presentational Pattern, Data Provider Pattern
- Dynamic Components, Provide/Inject
- Render functions, Renderless components
- `<script setup>` — Compile-time syntactic sugar for Composition API
- State Management
