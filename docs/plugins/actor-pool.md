# Actor Pool

## Overview

> [Github Repository](https://github.com/ArtemIyX/ActorPoolUnreal)

The ActorPool plugin provides a flexible and efficient system for managing a pool of reusable actors. By utilizing an object pooling pattern, this plugin allows developers to optimize performance by reusing actors instead of repeatedly spawning and destroying them. The plugin is written in C++ and is fully customizable via Blueprints, making it suitable for both programmers and designers.

## Features

- Object Pooling: Efficiently manage a pool of actors to reduce instantiation overhead.
- Blueprint Support: Functions like GetActorFromPool, ReturnActor, and MakeActor are exposed to Blueprints for easy customization.
- Network Replication: The pool and active actor lists are replicated for multiplayer compatibility.
- Customizable Pool Initialization: Configure pool size, initialization behavior, and spawning options via Blueprint properties.
- Extensible: Override MakeActor and InitializePool in C++ or Blueprints to tailor actor creation and pool setup.

## IPoolable

!!! note
    The actor that is created by **AAbstractActorPool** must implement an **IPoolable** interface

When a pool is created, each actor will have a **DeActivePoolActor()** called.
**DeActivePoolActor()** also called when the object is removed back to the pool.

To retrieve an actor from the pool, you can use 
```C++
AActor* AAbstractActorPool::GetActorFromPool()
```

This will call **ActivatePoolActor()** on the actor.

If there are no free actors in the pool, **MakeActor()** will be called.

## MakeActor()

You must overwrite **MakeActor()** in your pool. It will be called when you want to initialise the pool or add a new actor to the pool.

!!! note "ActivatePoolActor()"
    Do not use **BeginPlay()** for initialization. The objects are created empty and only **ActivatePoolActor()** activates them.

## Replication

The plugin fully supports replication and uses the [Unreal Push Network Model](https://www.kierannewland.co.uk/push-model-networking-unreal-engine/) to optimize network.

## C++ Example

```C++
// Example of using the pool in C++
AAbstractActorPool* Pool = GetWorld()->SpawnActor<AAbstractActorPool>();
AActor* PooledActor = Pool->GetActorFromPool();
if (PooledActor)
{
    // Use the actor (e.g., set location, activate behavior)
    PooledActor->SetActorLocation(FVector(100, 100, 100));
}

// When done, return the actor to the pool
Pool->ReturnActor(PooledActor);
```

## Riftborn Example

```C++ title="HumWeaponCharacter.cpp"
if (ABloodEffectActor* bloodEffect = Cast<ABloodEffectActor>(bloodEffectPool->GetActorFromPool()))
{
	bloodEffect->SetActorLocation(InLoc);
	bloodEffect->PlayEffect(InType);
}
```

## Limitations

- Actors in the pool must implement the IPoolable interface to ensure proper activation and deactivation.
- The plugin assumes actors are lightweight and suitable for pooling. Heavy actors with complex initialization may require additional optimization.
- Network replication is supported, but ensure proper testing in multiplayer scenarios to avoid synchronization issues.