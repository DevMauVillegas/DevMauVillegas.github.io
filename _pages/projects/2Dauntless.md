---
title: "Dauntless"
image: "/assets/images/Dauntless/Dauntless-Game-screenshot-1.jpg"
permalink: /projects/dauntless/
layout: single
nav_exclude: true
search_exclude: true
sidebar:
  nav: "projects"
---


# Game Programmer — Phoenix Labs

## Project: Dauntless

![Dauntless Picture](/assets/images/Dauntless/Dauntless-Game-screenshot-1.jpg)

### Role Overview

As part of the UI Programming team on Dauntless, I was responsible for implementing and maintaining player-facing systems, including in-game HUD elements, menus, and interactive UI flows. The team owned all layers of player interaction; from menu architecture to reactive combat feedback systems.

Most of the UI architecture followed the Model–View–ViewModel (MVVM) pattern to ensure separation of concerns, scalability, and testability across screens and UI components.

---

### Responsibilities & Contributions
**UI Feature Development**

Implemented new UI/UX features that were shipped to production. This included:

- Menu construction and navigation flows.

- In-game HUD elements.

- Dynamic UI elements on screen. _(Like damage numbers or combat feedback)_

- Data-driven UI bindings using MVVM architecture.

**Work required close collaboration with:**

- Game Designers. _(Feature requirements and UX flows)_

- Artists. _(Basic visual implementation fidelity)_

- Gameplay Programmers. _(Data integration and event hooks)_

All features were production-grade and integrated into a live-service environment.



---

### Blueprint to C++ Migration

Refactored and migrated multiple Blueprint-based UI systems into C++.

**Objectives:**

- Improve performance.

- Increase maintainability.

- Reduce technical debt.

- Enforce architectural consistency.

**This involved:**

- Reorganizing existing Blueprint hierarchies.

- Designing C++ base classes for UI components.

- Preserving designer iteration workflows while strengthening system foundations.

The result was cleaner separation between logic and presentation, and improved long-term scalability of the UI layer.

---

### Bug Diagnosis & Cross-Team Debugging

Actively diagnosed, reported, and resolved issues both within and outside the UI team’s direct ownership.

**This included:**

- UI rendering issues.

- Data binding inconsistencies.

- Gameplay-to-UI event propagation bugs.

- Cross-system integration problems.

I regularly traced issues across gameplay systems and UI layers — requiring a strong understanding of Unreal’s lifecycle, memory management, and event systems.

>**Example of a Menu that I've worded on.**
>
>![Menu Example](/assets/images/Dauntless/dauntless-afterbattle-menu.jpg)

---

### Live Service Environment

All work was performed in a live production environment, meaning:

- Features had to be backward-compatible.

- Changes required careful regression testing.

- Stability and performance were critical.

- Fixes had to be deployed without disrupting ongoing player activity.

---

### Architecture: MVVM in Unreal

The UI systems followed a structured Model-View-Viewmodel approach:

- Model: Gameplay systems and data sources

- ViewModel: Data transformation layer, exposing bindable properties

- View: UMG Widgets responsible only for presentation

**This architecture enabled:**

- Clear separation between gameplay logic and UI.

- Easier debugging.

- Better testability.

- Safer iteration by designers.

---

### Recognition

- Credited as a developer in Dauntless.

- Contributed directly to features shipped in a commercially released live-service title.