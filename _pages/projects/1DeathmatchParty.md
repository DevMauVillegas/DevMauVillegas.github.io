---
title: "Deathmatch Party"
image: "/assets/images/DeathmatchParty/DeathmatchParty.gif"
permalink: /projects/deathmatch-party/
layout: single
nav_exclude: true
search_exclude: true
sidebar:
  nav: "projects"
---

> **PERSONAL PROJECT**
## Overview

This is an unreal engine 5 online multiplayer FPS game.

It uses Steam services to connect players around the world as long as they have a steam account.

## Repository

<!--The nonsense after the link is so it opens in a new tab -->
### [Look at Project on Github](https://github.com/DevMauVillegas/DeathmatchParty){: target="_blank" rel="noopener noreferrer" }


---


## [-> Combat Component <-](/projects/deathmatch-party/combat-component/)

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


## [-> Lag Compensation Component <-](/projects/deathmatch-party/lag-compensation-component/)

### Purpose:
Enable server-side rewind to fairly validate hits under client latency by recording historical hitbox transforms and rewinding hit collision checks to a player’s past pose.

It's owned by `APartyCharacter`; created in the character constructor and wired in `PostInitializeComponents`.

---

## [-> Gameplay Ability/Effect System Usage <-](/projects/deathmatch-party/gameplay-ability-system/)
#### The project uses the Gameplay Ability System to execute some of the gameplay events such as:

* Damage
* Buffs
* Shields

---

# Details

![First image](/assets/images/DeathmatchParty/BigMap_Portfolio.png)

![First image](/assets/images/DeathmatchParty/Lobby_Portfolio.png)

![First image](/assets/images/DeathmatchParty/MainMenu_Portfolio.png)

![First image](/assets/images/DeathmatchParty/PlayingMatch_Portfolio.gif)

