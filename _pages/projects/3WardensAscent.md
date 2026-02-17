---
title: "Warden's Ascent"
image: "/assets/images/gamelogo.png"
permalink: /projects/wardens-ascent/
layout: single
nav_exclude: true
search_exclude: true
sidebar:
  nav: "projects"
---


# Gameplay Programmer

## Project: Warden’s Ascent

### Role Overview

Joined the programming team during the early stages of development, contributing to the technical design and implementation of core gameplay and progression systems.

My responsibilities centered around campaign progression architecture, traversal mechanics, combat extensibility using Gameplay Ability System (GAS), and backend synchronization systems. The role required cross-disciplinary collaboration with design, backend, and production teams to ensure gameplay systems were modular, scalable, and data-driven.

## Campaign Progression Systems

Owned and implemented several systems responsible for tracking player progression throughout the single-player campaign.

### Responsibilities

- Designed data structures to represent campaign states and progression checkpoints

- Implemented persistent progression tracking across sessions

- Built logic to handle branching campaign level ordering

- Integrated reward distribution logic based on campaign completion state

- Ensured synchronization with backend systems when campaign parameters changed

### Technical Focus

- Clean separation between progression logic and presentation

- Save-state integrity and validation

- Backend-aware architecture to avoid desync between server-configured campaign order and client state

- Extensible data-driven configuration to allow designers to modify campaign flows without requiring code changes

This system became foundational for how player state and reward logic were managed across the project.

## Traversal System — Climbing Mechanic

Designed and implemented a climbing system for playable characters.

### System Goals

- Enable vertical exploration

- Allow designers to define climbable surfaces flexibly

- Maintain consistency with character movement systems

- Integrate smoothly with animation and collision systems

### Technical Implementation

- Extended Unreal’s character movement framework

- Implemented custom movement states

- Created configurable parameters for designers (speed, stamina cost, traversal limits)

- Integrated animation state transitions with gameplay state

The system was designed to be modular, enabling level designers to construct traversal-heavy environments without requiring engineering intervention.

## Combat Extensibility — Gameplay Ability System (GAS)

Implemented a combat customization system leveraging Unreal Engine’s Gameplay Ability System.

### Objectives

- Allow designers to modify combat behavior for both Campaign and PvP modes

- Support scalable addition of abilities and effects

- Ensure deterministic execution across networked sessions

### Contributions

- Implemented custom abilities and gameplay effects

- Extended attribute handling logic

- Structured ability activation rules for multiple game modes

- Created configuration layers to differentiate Campaign vs PvP balancing

This system allowed combat tuning without deep code changes and provided a flexible foundation for content iteration.

## Backend Synchronization System

Extended and maintained systems responsible for syncing gameplay configuration with backend-defined settings.

### Scope

- Reward structures

- Campaign level ordering

- Event-based configuration

- Mode-specific parameters

### Technical Challenges

- Preventing desynchronization between client and backend

- Ensuring safe updates when game settings changed

- Handling edge cases during patch deployment

This required careful validation logic and defensive programming to avoid corrupted progression states.

## Collaboration & Engineering Practices

- Participated in early technical design discussions

- Worked closely with designers to translate mechanics into scalable systems

- Maintained high code readability and modularity standards

- Focused on long-term maintainability rather than one-off feature implementation