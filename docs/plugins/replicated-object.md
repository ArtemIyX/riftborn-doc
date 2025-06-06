# Replicated Object

## About the plugin

> [Github Repository](https://github.com/ArtemIyX/ReplicatedObjectUnreal)

## Overview

As you may know Unreal Engine does not replicate UObjects. But in some solutions, such as inventory, it can come in handy very often.

This plugin provides a simple solution to replicate UObjects.

## Instruction 

To make replication work there are some steps that need to be followed.

1. Create a class that will inherit from UAdvancedReplicatedObject

    ```C++ title="MyReplicatedObject.h"
    class YOUR_API UMyReplicatedObject : public UAdvancedReplicatedObject 
    {
    GENERATED_BODY()
    public:
    UMyReplicatedObject(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer) {}
    virtual bool IsSupportedForNetworking() const override { return true; }
    }
    ```

2. Create a **UPROPERTY** variable with a pointer to your object.

    ```C++ title="MyActor.h"
    class YOUR_API AMyActor : public AActor {
    public:
    UPROPERTY(Replicated)
    class UMyReplicatedObject* MyObjectPtr;
    }
    ```

3. Override **GetLifetimeReplicatedProps(...)** and **ReplicateSubobjects(...)** in your actor.

    ```c++ title="MyActor.h"
    public:
    virtual void GetLifetimeReplicatedProps(TArray<class FLifetimeProperty>& OutLifetimeProps) const override;
    virtual bool ReplicateSubobjects(UActorChannel* Channel, FOutBunch* Bunch, FReplicationFlags* RepFlags) override;
    ```

    ```C++ title="MyActor.cpp"
    void AMyActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
        Super::GetLifetimeReplicatedProps(OutLifetimeProps);
        DOREPLIFETIME(AMyActor, MyObjectPtr);
    }

    bool AMyActor::ReplicateSubobjects(UActorChannel* Channel, FOutBunch* Bunch, FReplicationFlags* RepFlags)
    {
        bool sup = Super::ReplicateSubobjects(Channel, Bunch, RepFlags);
        if (IsValid(MyObjectPtr))
        {
            sup |= Channel->ReplicateSubobject(MyObjectPtr, *Bunch, *RepFlags);
            sup |= MyObjectPtr->ReplicateSubobjects(Channel, Bunch, RepFlags);
        }
        return sup;
    }
    ``` 

## Example

> The object must be created on the server side.

```C++ title="MyActor.cpp"
void AMyActor::BeginPlay()
{
  Super::BeginPlay();
  if (HasAuthority())
  {
    MyObjectPtr = NewObject<UAdvancedReplicatedObject>(this, UAdvancedReplicatedObject::StaticClass());
  }
}
```

!!! note "Outer of UAdvancedReplicatedObject"
    It is desirable to make the Outer of the object an Actor. Otherwise RPC will not work, and replication will work only if the Outer of your Outer is an Actor.

## Array of objects

If you need to make an array, then create a array of pointers and replicate the data through the FOR.

```C++ title="ReplicatedActorExample.h"
class REPLICATEDOBJECT_API AReplicatedActorExample : public AActor {
	GENERATED_BODY()
protected:
	UPROPERTY(Replicated)
	TArray<UAdvancedReplicatedObject*> MyArray;

public:
    virtual void GetLifetimeReplicatedProps(TArray<class FLifetimeProperty>& OutLifetimeProps) const override;
    virtual bool ReplicateSubobjects(UActorChannel* Channel, FOutBunch* Bunch, FReplicationFlags* RepFlags) override;
}
```

```C++ title="ReplicatedActorExample.cpp"
void AReplicatedActorExample::BeginPlay()
{
	Super::BeginPlay();
	if (HasAuthority())
	{
		MyArray.Add(MyObjectPtr = NewObject<UAdvancedReplicatedObject>(this, UAdvancedReplicatedObject::StaticClass()));
		MyArray.Add(MyObjectPtr = NewObject<UAdvancedReplicatedObject>(this, UAdvancedReplicatedObject::StaticClass()));
	}
}
void AReplicatedActorExample::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
	DOREPLIFETIME(AReplicatedActorExample, MyArray);
}
bool AReplicatedActorExample::ReplicateSubobjects(UActorChannel* Channel, FOutBunch* Bunch, FReplicationFlags* RepFlags)
{
	bool sup = Super::ReplicateSubobjects(Channel, Bunch, RepFlags);
	int32 n = MyArray.Num();
	if(n > 0)
	{
		for(int32 i = 0; i < n; ++i)
		{
			sup |= Channel->ReplicateSubobject(MyArray[i], *Bunch, *RepFlags);
			sup |= MyArray[i]->ReplicateSubobjects(Channel, Bunch, RepFlags);
		}
	}
	return sup;
}
```

## Move object

If you want to move an object between actors, then it's worth remembering how replication works in the Unreal Engine.

!!! warning 
    You can't just put a variable into another actor: its Outer will not change. To do this, you should use the **Rename** method.

```C++
AActor* anotherActor = ...;
MyObjectPtr->Rename(nullptr, anotherActor);
```
> It should be noted that the Rename function is quite slow and it will update on the client with a long delay.

There may be bugs when using the variable on the client. So I recommend DuplicateObject().

```C++
AReplicatedActorExample* anotherActor = ...;
UAdvancedReplicatedObject* copy = DuplicateObject(MyObjectPtr, anotherActor);
anotherActor->MyObjectPtr = copy;
MyObjectPtr->ConditionalBeginDestroy();
MyObjectPtr = nullptr;
```

## Tips and tricks

- Replication of large objects takes a long time. I recommend using direct TCP connection for large tasks like procedural generation or huge inventories in MMO RPG.
- Replication was tested only on the Dedicated Server. Listen server was not tested.
- Sometimes it is more advantageous to use RPC events to transfer bytes rather than replicating UObjects.
- Replicating arrays of UObjects takes a decent amount of time. Take into account that the inventory systems may take much longer to load than expected (more than 10ms).
- Remember that an object will live as long as its actor lives.

!!! note
    I suggest deleting (ConditionalBeginDestroy()) objects before the actor is deleted to avoid problems with garbage collection and memory leaks.