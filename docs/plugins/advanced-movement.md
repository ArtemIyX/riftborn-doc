# Advanced Movement

## Overview

> [Github Repository](https://github.com/ArtemIyX/AdvancedMovementUnreal)

Author plugin written for Riftborn. It adds handy movement logic for Character Movement Component.

## Features

- Replication
- Sprinting
- Dashing

## Instructions

Inherit from **UAdvancedMovementComponent** to have all the features.

The owner of the component must inherit from **AAdvancedMovementCharacter**.

```C++
class HUM_API UHumMovementComponent : public UAdvancedMovementComponent
```

Override **GetMaxSpeed()** to adjust max speed of Character.

```C++
if (IsMovingBackward())
{
	return IsCrouching() ? MaxBackwardWalkSpeedCrouched : MaxBackwardWalkSpeed;
}
```

## Sprinting

Use **SprintPressed()** to start the sprinting and **SprintReleased()** to stop sprinting.

```C++
enhancedInputComponent->BindAction(sprintAction,
			                                   ETriggerEvent::Started,
			                                   this,
			                                   &AHumBaseCharacter::SprintPressed);
```

```C++
void AHumBaseCharacter::SprintReleased(const FInputActionValue& InValue)
{
	AdvancedMovementComponent->SprintReleased();
}

void AHumBaseCharacter::SprintPressed(const FInputActionValue& InValue)
{
	if (IsAbleToHandleInput())
	{
		AdvancedMovementComponent->SprintPressed();
	}
}
```

Sprint is allowed only if **IsSprintingAllowed()** returns true.

```c++
bool UAdvancedMovementComponent::IsSprintingAllowed() const
{
	return !IsCrouching() 
		&& !IsFalling() 
		&& !IsCustomMovementMode(CMOVE_Slide)
		&& IsMovingOnGround() 
		&& Velocity.SizeSquared() >= 100.0f 
		&& !Safe_bWantsToSprint
		&& AdvancedCharacter->CanSprint(); 
}
```

!!! note
	It also calls AdvancedCharacter CanSprint().
	This can be useful for adding additional conditions


!!! note 
    You can check if the character wants to run in GetMaxSpeed() to set it to the desired speed

```C++
IsMovementMode(MOVE_Walking) && Safe_bWantsToSprint
```

---

## Dashing

Use **DashPressed()** and **DashReleased()** to perform Dash.

!!! note 
    This must be executed on the client to replicate properly

**CanDash()** determines whether the character can currently Dash or not.

```c++
bool UAdvancedMovementComponent::CanDash() const
{
	return IsWalking() &&
		!IsSprinting() &&
		!IsSliding() &&
		!IsCrouching() &&
		!IsFalling() && AdvancedCharacter->CanDash();
}
```

!!! note AdvancedCharacter
	It also calls **CanDash()** from AdvancedCharacter.
	This is useful if you want to add conditions for performing a Dash