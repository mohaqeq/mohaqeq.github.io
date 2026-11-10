---
title: "A berth planning board as a native web component"
date: 2026-01-04
tags: [typescript, web-components, lit, ui, logistics]
summary: "A Gantt-like, drag-and-drop schedule of vessels at a port's berths, built with Lit and TypeScript as a standards-based custom element."
---

Port operations have a planning problem that looks simple and isn't. Vessels arrive, occupy a stretch of quay for a
while, and leave. Two ships cannot overlap in space and time, and the plan changes all day. Planners need a board where
they can *see* the quay over time and move things with the mouse.

I built such a board as a reusable web component:
[mohaqeq/berthing-schedule](https://github.com/mohaqeq/berthing-schedule), with a live demo at
[berthing.netlify.app](https://berthing.netlify.app/).

## What it does

- **A Gantt-like view of the quay:** berths (and bollards) along one axis, time along the other, and each vessel call as
  a box.
- **Drag and drop:** move a vessel to another berth or time, or resize it to change arrival and departure.
- **More than ships:** non-working periods such as maintenance or planning blocks are items on the board too.
- **Guard rails:** lock options, voyage timing management and automatic timing adjustments keep the plan consistent
  while you edit it.
- **Navigation:** keyboard control, zoom from 0.5× to 2×, and scroll tracking. An event log shows what happened.
- **Styling from outside:** berths, bollards, timeline periods and colours are configurable, and the look is themed with
  CSS custom properties.

## Why a web component

The board had to drop into any host application, whatever framework it uses, or none. A native custom element does
exactly that. The host writes `<berthing-schedule>`, sets properties and listens to events.

[Lit](https://lit.dev/) makes custom elements pleasant without turning them into a framework. The component is split
into small parts: `quay-panel`, `timeline-panel`, `grid-lines`, `corner-box` and `vessel-box`, composed by the main
`berthing-schedule` element. Types live in one `types.ts`. TypeScript targets ES2022, and Vite serves the demo and
builds the bundle.

## Things that were harder than expected

- **Time and pixels are two coordinate systems.** Every drag, resize and zoom converts between them. Keeping one
  well-tested conversion in one place avoids off-by-one-hour bugs.
- **Drag-and-drop must respect the domain**, not just the mouse: locked items, minimum durations, and what happens to
  the next voyage when you stretch this one.
- **Two scroll axes plus zoom** in a grid with fixed headers is a classic source of misaligned rows. Separate panels with
  synchronised scrolling worked better than one giant scrolling element.

## Try it

```bash
npm install
npm run dev     # the demo page
npm run build   # the distributable component
```
