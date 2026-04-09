---
title: "Deathmatch Party - Lag Compensation Component"
layout: single
permalink: /projects/deathmatch-party/lag-compensation-component/
nav_exclude: true
search_exclude: true
sidebar:
  nav: "projects"
---

# Lag Compensation Component — Overview

### Key data structures

- #### `FBoxInformation`: Location, rotation, extent for a single hitbox.
    ```cpp
    USTRUCT(BlueprintType)
    struct FBoxInformation
    {
        GENERATED_BODY()

        UPROPERTY()
        FVector Location;

        UPROPERTY()
        FRotator Rotation;

        UPROPERTY()
        FVector BoxExtent;
    };
    ```

-  #### `FFramePackage`: Timestamp, map of hitbox `FName` → `FBoxInformation`, and `APartyCharacter*`.
    ```cpp
    USTRUCT(BlueprintType)
    struct FFramePackage
    {
        GENERATED_BODY()

        UPROPERTY()
        float Time;

        UPROPERTY()
        TMap<FName, FBoxInformation> HitBoxInfo;

        UPROPERTY()
        APartyCharacter* PartyCharacter;
    };
    ```

- #### `FrameHistory`: DoubleLinkedList inside the `ULagCompensationComponent` clss storing recent frames; trimmed to `MaxRecordTime`.
    ```cpp
    TDoubleLinkedList<FFramePackage> FrameHistory;

    UPROPERTY(EditAnywhere)
    float MaxRecordTime = 0.5f;
    ```

---

### How it records frames
- Server-only recording: `TickComponent` calls `SaveFramePackage()` when the owning character has server authority.
- SaveFramePackage(FFramePackage&) captures current transform + extent of each collision box in the `HitCollisionBoxes` of the party character into a `FFramePackage` and pushes it into `FrameHistory`.
- Frames older than MaxRecordTime are removed.

### How it answers rewind queries
- RPC entry points (all UFUNCTION(Server, Reliable)):
- ServerScoreRequest — hit-scan (single hit).
- ProjectileServerScoreRequest — projectile hit validation.
- ShotgunServerScoreRequest — multi-pellet / shotgun validation.

---

### Request flow (common pattern):
1.	Client detects hit locally or the projectile hits locally and gathers relevant info (trace start, hit location(s), projectile init velocity, and HitTime computed as server time - round-trip estimate).
2.	Client calls the appropriate server RPC on its character’s ULagCompensationComponent.
3.	Server: GetFrameToCheck(HitCharacter, HitTime) locates two recorded frames around HitTime, interpolates between them with FrameInterpolation(...) to produce an accurate pose for that time.
4.	Server temporarily MoveBoxes(...) on the hit character to those historic transforms and disables the character mesh collisions (EnableCharacterMeshCollision(..., NoCollision)) to avoid double collisions.
5.	Depending on weapon type:
    - Hit-scan: ConfirmHit(...) performs LineTraceSingleByChannel against the moved box components to check for head/body hit.
    - Projectile: ProjectileConfirmHit(...) runs PredictProjectilePath using the supplied InitialVelocity and TraceStart, checks HitResult.
    - Shotgun: ShotgunConfirmHit(...) moves the target's boxes for each candidate character and performs multiple traces (one per pellet target), counting head/body hits.
6.	Server ResetBoxes(...) to the cached frame and re-enables mesh collision.
7.	If a hit is confirmed, the server applies damage via UGameplayStatics::ApplyDamage(...) using controller/weapon info (implemented inside the _Implementation methods).

---

### Files and call sites in this project

- Component creation / wiring:
    - APartyCharacter:
        - Creates component in constructor: LagCompensationComponent = CreateDefaultSubobject<ULagCompensationComponent>(...)
        - Sets PartyCharacter and PartyController on PostInitializeComponents.
        - Hit boxes are created and stored in HitCollisionBoxes via SetupCollisionBoxes().
- Weapon/attack flows that invoke rewind:
    - ProjectileBullet.cpp
        - OnHit — when projectile hits locally and bUseServerSideRewind is true and owner is locally controlled:
            - Calls OwnerCharacter->GetLagCompensationComponent()->ProjectileServerScoreRequest(...) with TraceStart, InitialVelocity, and OwnerController->GetServerTime() - OwnerController->SingleTripTime.

- Shotgun.cpp
    - AShotgun::FireShotgun — when client fires with bUseServerSideRewind true and not server-authority:
        - Gathers pellet HitTargets and calls ShotgunServerScoreRequest(...).

- Weapon.h
    - Weapons expose flag bUseServerSideRewind to opt into this behavior.

- Hit-scan usage:
    - ULagCompensationComponent::ServerSideRewind / ConfirmHit — supports hit-scan validation if you call ServerScoreRequest. (Search your project for where hit-scan weapons call this server RPC; shotgun/projectile examples above are explicit.)

---

### Important implementation notes / behavior details (quick)
- MaxRecordTime defaults to 0.5s; frames older than that are discarded from FrameHistory.
- Rewind works by moving UBoxComponent hitboxes to historical transforms, temporarily disabling `/ altering mesh collision to avoid mesh-level hits interfering with the test.
- Projectile paths validated using UGameplayStatics::PredictProjectilePath with the supplied InitialVelocity.
- Shotgun logic counts headshots vs body shots per target (maps returned in FShotgunServerSideRewindResult).
- RPCs are server-reliable — the authoritative server performs final validation and applies damage.

---

### How to enable / use in your weapons
- Set AWeapon::bUseServerSideRewind = true for weapons that should use server-side rewind.
- When firing:
    - For projectile weapons: ensure the projectile owner is locally controlled, then on local projectile hit call ProjectileServerScoreRequest(...) with:
        - TraceStart — starting point of projectile traces (usually muzzle socket location).
        - InitialVelocity — the projectile initial velocity (serialized).
        - HitTime — use controller server time minus estimated network one-way delay (GetServerTime() - SingleTripTime).
    - For shotgun / multi-hit weapons: build the pellet HitTargets array client-side and call ShotgunServerScoreRequest(...).
    - For hitscan: call ServerScoreRequest(...) with TraceStart, HitLocation, and computed HitTime.

---

### Where to look if behavior is wrong / debugging tips
- Check recorded frames in FrameHistory and timestamps inserted by SaveFramePackage.
- GetFrameToCheck finds older/younger frames and calls FrameInterpolation. Verify FrameInterpolation is producing expected transforms (interpolation fraction calculation matters).
- Ensure HitCollisionBoxes are properly initialized and registered (SetupCollisionBoxes in APartyCharacter).
- Confirm clients compute HitTime using the same clock base as server RPC arguments (the project uses APartyPlayerController::GetServerTime and SingleTripTime).
