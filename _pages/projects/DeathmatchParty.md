---
title: "Deathmatch Party"
image: "/assets/images/DeathmatchParty.gif"
layout: single
permalink: /projects/deathmatch-party/
author_profile: false
nav_exclude: true
search_exclude: true
sidebar:
  nav: "projects"
---

## Overview

This is an unreal engine 5 online multiplayer FPS game.

It uses Steam services to connect players around the world as long as they have a steam account.

## Repository

<!--The nonsense after the link is so it opens in a new tab -->
### [Look at Project on Github](https://github.com/DevMauVillegas/DeathmatchParty){: target="_blank" rel="noopener noreferrer" }

## Index

1. [**Gameplay Mechanics**](#gameplay)

2. [**Relevant Authoritative Logic**](#authsystems)

3. [**Lag Controls**](#lagcontrols)

4. [**Gameplay Ability System**](#gassystems)

5. [**Enemy AI**](#enemyai)


Combat component


## Gameplay Mechanics {#gameplay}


### [-> Combat Component <-](/projects/deathmatch-party/combat-component/)

`UCombatComponent` is a replicated `UActorComponent` responsible for handling all combat-related logic for a character, including:

- Weapon equipping and swapping

- Firing logic (Projectile, Hitscan, Shotgun)

- Reloading

- Aiming and FOV interpolation

- Crosshair spread computation

- Ammo management

- Multiplayer synchronization (Server / Multicast RPCs)

- HUD updates

It centralizes combat logic outside the character class, keeping the character focused on movement and animation.

---

## Relevant Authoritative Logic {#authsystems}

* Connecting to server and etc

## Lag Controls {#lagcontrols}

* Recall thingy

## Gameplay Ability System {#gassystems}

* Buffs and stuff 

## Enemy AI {#enemyai}

* Behaviour tree


![First image](/assets/images/DeathmatchParty/BigMap_Portfolio.png)

![First image](/assets/images/DeathmatchParty/Lobby_Portfolio.png)

![First image](/assets/images/DeathmatchParty/MainMenu_Portfolio.png)

![First image](/assets/images/DeathmatchParty/PlayingMatch_Portfolio.gif)

