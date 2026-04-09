---
title: "Deathmatch Party - Ability System Usage"
layout: single
permalink: /projects/deathmatch-party/gameplay-ability-system/
nav_exclude: true
search_exclude: true
sidebar:
  nav: "projects"
---

# Gameplay Ability System Usage  — Overview


## Damage Handling with the Gameplay Ability System (GAS)

This system integrates Unreal Engine’s **standard damage pipeline** with the **Gameplay Ability System (GAS)** to manage character attributes such as **Health** and **Shield**.

Damage is first received through Unreal's built-in damage system and then converted into **Gameplay Effects** that modify GAS attributes.

```cpp
ApplyDamage() -> OnTakeAnyDamage delegate -> ReceiveDamage() -> Create GameplayEffect -> AbilitySystemComponent applies effect -> AttributeSet updates attributes
```

### Damage Reception

The character registers for Unreal’s global damage delegate:

```cpp
OnTakeAnyDamage.AddDynamic(this, &ThisClass::ReceiveDamage);
```

Damage is processed on the server to ensure authoritative gameplay.

### Converting Damage into Gameplay Effects

Inside ReceiveDamage, the system converts incoming damage into Gameplay Effects.

Damage is split into:

* Shield damage
* Health damage

Shield absorbs damage first, and any remaining damage affects health.

#### Each portion of the damage is applied through a Gameplay Effect:

```cpp
AbilitySystemComponent->ApplyGameplayEffectSpecToSelf(*Spec);
```
The amount of damage is passed using SetByCallerMagnitude.

#### Example:

```cpp
Spec->SetSetByCallerMagnitude(DamageEffectTag, Damage * (-1));
```

The value is negative because the attribute is being reduced.

### Attribute Updates

Health and Shield live inside a GAS AttributeSet.
When Gameplay Effects modify these attributes, delegates notify the character.

```cpp
AbilitySystemComponent
->GetGameplayAttributeValueChangeDelegate(AttributeSet->GetHealthAttribute())
.AddUObject(this, &APartyCharacter::OnHealthChanged);
```

#### These callbacks are used to:

* Update the HUD
* Play hit reactions
* Check for elimination

---

# Applying Buffs with the Gameplay Ability System (GAS)

Buffs in the game are implemented using **Gameplay Effects applied through the Gameplay Ability System (GAS)**.  
They are triggered when a player overlaps with a **Buff PickUp actor** placed in the level.


The buff pipeline follows this flow:

```cpp
Player overlaps PickUp -> BuffPickUp::OnSphereOverlap() -> Create GameplayEffectSpec -> Set buff parameters -> AbilitySystemComponent applies effect -> AttributeSet modifies attributes
```

### PickUp Actor

Buffs are represented by actors derived from `ABuffPickUp`.  
Designers configure the buff behavior through Blueprint variables:

- **GameplayEffect** – the effect class applied to the player
- **EffectAmount** – magnitude of the buff
- **EffectDuration** – how long the buff lasts
- **EffectPeriod** – tick interval for periodic effects
- **EffectTag** – identifies which attribute or modifier is affected

This allows different buffs to reuse the same C++ logic.

### Overlap Detection

When a player touches the pickup, the overlap event triggers:

```cpp
void ABuffPickUp::OnSphereOverlap(...)
```

The actor checks if the overlapping actor is a character and retrieves its Ability System Component.

```cpp
UAbilitySystemComponent* AbilitySystemComponent =
    PartyCharacter->GetAbilitySystemComponent();
```

### Creating the Gameplay Effect

A GameplayEffectSpec is created dynamically using the character's Ability System Component.

```cpp
FGameplayEffectSpecHandle SpecHandle =
    AbilitySystemComponent->MakeOutgoingSpec(GameplayEffect.Get(), 1.0f, EffectContext);
```

The spec acts as a runtime instance of the Gameplay Effect.

### Configuring Buff Parameters

The buff parameters are set on the spec before applying it.

#### Buff Magnitude
This allows designers to control the buff value without creating multiple effect classes.
```cpp
Spec->SetSetByCallerMagnitude(EffectTag, EffectAmount);
```

#### Duration
Defines how long the buff remains active.
```cpp
Spec->SetDuration(EffectDuration, true);
```

#### Periodic Effects
If a period is defined, the Gameplay Effect executes repeatedly (for example healing over time).
```cpp
Spec->Period = EffectPeriod;
```

### Applying the Buff

Once configured, the effect is applied to the character:

```cpp
AbilitySystemComponent->ApplyGameplayEffectSpecToSelf(*Spec);
```

### GAS then handles:

* Attribute modification
* Replication
* Stacking rules
* Expiration of the effect
* PickUp Destruction

#### After applying the buff, the pickup destroys itself:

```cpp
Destroy();
```

A Niagara effect is optionally spawned for visual feedback.