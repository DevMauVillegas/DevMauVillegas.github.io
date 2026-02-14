---
title: "Deathmatch Party - Combat Component"
layout: single
permalink: /projects/deathmatch-party/combat-component/
author_profile: false
nav_exclude: true
search_exclude: true
---

## Architectural Responsibilities

### Ownership Model

The component is initialized with a reference to its owning APartyCharacter:

``` cpp
void UCombatComponent::InitializePartyCharacter(APartyCharacter* NewPartyCharacter)
```

**This method:**

- Caches references (character, camera)

- Stores default FOV

- Initializes carried ammo (server-only)

This establishes the character as the authority for combat state, while the component orchestrates combat behavior.

## Replication Model

Combat is multiplayer-driven and server-authoritative.

**Replicated Variables:**

``` cpp
DOREPLIFETIME(UCombatComponent, EquippedWeapon);
DOREPLIFETIME(UCombatComponent, BackupWeapon);
DOREPLIFETIME(UCombatComponent, bAiming);
DOREPLIFETIME(UCombatComponent, CombatState);
DOREPLIFETIME(UCombatComponent, bHoldingFlag);
DOREPLIFETIME_CONDITION(UCombatComponent, CarriedAmmo, COND_OwnerOnly);
```

**Key points:**

- `EquippedWeapon` / `BackupWeapon` → replicated to all clients

- `CombatState` → ensures synchronized reload/swap animations

- `CarriedAmmo` → replicated only to owning client (HUD optimization)

- Server validates fire rate and reload logic

**This ensures:**

- Visual correctness for all players

- Ammo privacy (only owner sees carried ammo)

- Server authority for gameplay integrity

- Firing System

### The firing system supports three fire types:

``` cpp
switch (EquippedWeapon->FireType)
{
    case EFireType::EFT_Projectile:
    case EFireType::EFT_HitScan:
    case EFireType::EFT_MultipleHitScan:
}
```

**Fire Flow:**

- Player presses fire

- `CanFire()` validates state

- Local prediction executes

- Server RPC validates and multicasts

Example – Hitscan Weapon:

``` cpp
void UCombatComponent::FireHitScanWeapon(const FVector_NetQuantize& TraceHitTarget)
{
    const FVector_NetQuantize HitTarget =
        EquippedWeapon->bUseScatter
        ? EquippedWeapon->TraceEndWithScatter(TraceHitTarget)
        : TraceHitTarget;

    LocalFire(HitTarget);
    ServerFire(HitTarget, EquippedWeapon->FireDelay);
}
```

## Network Flow
```
LocalFire(...)
→ ServerFire(...)
→ MulticastFire(...)
```

**This design provides:**

- Immediate local responsiveness

- Server validation

- Remote client synchronization

- Shotguns use an array of trace targets for pellet simulation.

- Fire Rate Control

- Fire rate is managed via timer-based gating:

``` cpp
void UCombatComponent::StartFireTimer()
{
    GetWorldTimerManager().SetTimer(
        FireTimer,
        this,
        &ThisClass::FireTimerFinished,
        EquippedWeapon->FireDelay);
}
```

**When the timer ends:**

``` cpp
if (bFireButtonPressed && EquippedWeapon->bAutomatic)
{
    Fire();
}
```

**This enables:**

- Automatic weapons

- Semi-auto enforcement

- Fire delay validation (also server-validated)

- Reload System

- Reloading is state-driven.

**Reload Trigger:**

``` cpp
void UCombatComponent::Reload()
{
    if (CarriedAmmo > 0 &&
        CombatState != ECombatState::ECS_Reloading &&
        !bLocallyReloading)
    {
        ServerReload();
        HandleReload();
        bLocallyReloading = true;
    }
}
```

## Server Authority

``` cpp
void UCombatComponent::ServerReload_Implementation()
{
    CombatState = ECombatState::ECS_Reloading;
}
```


Ammo values are only updated on the server:

``` cpp
void UCombatComponent::UpdateAmmoValues()
```

**This prevents:**

- Client-side ammo cheating

- Desynchronization

- Weapon Equipping & Swapping

- The system supports:

- Primary weapon

- Backup weapon

- Flag (special case)

## Primary Equip

``` cpp
void UCombatComponent::EquipPrimaryWeapon(AWeapon* inWeapon)
```

**Steps:**

1. Drop previous weapon

2. Set ownership

3. Attach to skeletal socket

4. Update ammo

5. Play sound

6. Update HUD

**Sockets used:**

- `HandSocket`

- `BackupSocket`

- `FlagSocket`

Weapon state changes drive visual logic (`EWS_Equipped`, `EWS_Backup`).

## Aiming & FOV Interpolation

**Aiming affects:**

- Walk speed

- FOV

- Crosshair spread

- Sniper scope UI

**FOV Interpolation:**

``` cpp
CurrentFOV = FMath::FInterpTo(
    CurrentFOV,
    EquippedWeapon->GetZoomedFOV(),
    DeltaTime,
    EquippedWeapon->GetZoomSpeed());
```

This creates a smooth zoom effect rather than instant snapping.

Walk speed is reduced while aiming for gameplay balance.

## Crosshair System

The crosshair dynamically reacts to gameplay factors:

- Player velocity

- In-air state

- Aiming

- Shooting recoil

**Spread is computed as:**

``` cpp
HUDPackage.CrosshairSpread =
    0.5f +
    CrosshairShootingFactor +
    CrosshairVelocityFactor +
    CrosshairInAirFactor +
    CrosshairAimFactor;
```

Each factor interpolates over time for smooth visual feedback.

**This provides:**

- Visual recoil feedback

- Movement penalty feedback

- Airborne accuracy penalty

- Aim stabilization

## Crosshair World Tracing

**Each tick (locally controlled only):**

``` cpp
TraceUnderCrosshairs(...)
```

**This:**

- Deprojects screen center

- Performs a visibility line trace

- Changes crosshair color if target implements interaction interface

**This supports:**

- Hit feedback

- Interactable highlighting

- Accurate hitscan targeting

## Combat State Machine

**Core states:**

- `ECS_Unoccupied`

- `ECS_Reloading`

- `ECS_SwappingWeapon`

**`CombatState` ensures:**

- No firing during reload (except shotgun case)

- No swap during reload

- Proper animation sync via `OnRep_CombatState`

## Multiplayer Design Pattern

**This component follows a classic:**

`Client-side prediction + server validation + multicast replication`

**Pattern used:**

- LocalFire → immediate feedback

- Server RPC → validation

- Multicast → remote sync

**Fire rate validated using:**

``` cpp
FMath::IsNearlyEqual(EquippedWeapon->FireDelay, FireDelay, 0.001f);
```

This prevents modified client fire rates.

## Design Strengths
1. **Separation of Concerns**

    Combat logic is isolated from character movement logic.

2. **Server Authority**

    All ammo mutations and combat states are server-driven.

3. **Prediction + Validation**

    Provides responsiveness without sacrificing integrity.

4. **Data-Driven Weapon Behavior**

    Weapon type and fire type determine behavior dynamically.

5. **HUD Integration**

    Crosshair and ammo updates are fully integrated and network-aware.

## Summary

**`UCombatComponent` is a fully networked combat system responsible for:**

- Weapon lifecycle management

- Firing orchestration

- Reload logic

- Aim mechanics

- Crosshair feedback

- Multiplayer synchronization

**It implements a scalable architecture that cleanly separates:**

- Visual feedback

- Gameplay authority

- Network replication

- Player input handling

This makes it robust for competitive multiplayer gameplay while remaining extensible for additional weapon types and combat states.