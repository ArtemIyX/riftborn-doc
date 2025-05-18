# Easy Settings

> [Github Repository](https://github.com/ArtemIyX/EasySettingsUnreal)

## Overview

A plugin that adds a subsystem that expands developers' options for game customization and makes it easier to develop certain settings.

## Developer Settings

Developer settings for the system,

```C++ title="EasySettingsSubsystemDeveloperSettings"
class EASYSETTINGS_API UEasySettingsSubsystemDeveloperSettings : public UDeveloperSettings
```

### Settings Setter Class

Used to create an instance of UEasySettingsSetter.

Change to your class to use its overwritten methods.

```C++ title="EasySettingsSubsystemDeveloperSettings"
TSubclassOf<UEasySettingsSetter> SettingsSetterClass;
```

### Container Save Name

Used in ``FString UEasySettingsSubsystem::GetContainerSavePath()``  to define the path to save the file.

Specify any file name and it will be saved to ``GetProjectDir()/ContainerSaveName``

```C++ title="EasySettingsSubsystemDeveloperSettings"
FString ContainerSaveName;
```

## Easy Settings Setter

```C++
class EASYSETTINGS_API UEasySettingsSetter : public UObject
```

A class that manages categorized float values.

UEasySettingsSetter is designed to store, retrieve, and serialize float values that are categorized by an `uint8` key.
It provides functionality to initialize, set, get, read, and write these categorized values.

```C++ title="EasySettingsSetter.h"
namespace EasySettings
{
	constexpr int32 VALUES_NUM = 254;
	typedef uint8 MapKey;
	typedef FEasySettingsContainerValue MapValue;
	typedef TMap<EasySettings::MapKey, EasySettings::MapValue> FContainer;
}

```

The map allows for fast retrieval and update of float values based on their category.

```C++ title="EasySettingsSetter.h"
protected:
EasySettings::FContainer Values;
```

Features: 

- SetValue
- ResetValue
- GetValue

### Set Value

Sets the float value for a specific category.
If the provided category exists in the Values map, this method updates the associated float value.

!!! tip
    Overwrite this method and you will be able to handle every parameter set in blueprints

```C++ title="EasySettingsSetter.h"
void SetValue(uint8 InCategory, float InValue);
```

!!! warning
    If the category does not exist, no action is taken.


### Get Value

Retrieves the float value associated with a specific category.

If the category exists, the method assigns the associated float value to the provided reference and returns true.

```C++ title="EasySettingsSetter.h"
virtual bool GetValue(uint8 InCategory, float& OutValue, bool& bOutDefault);
```

!!! warning
    If the category does not exist, the method returns false.

### Reset Value

Resets value to default

```C++ title="EasySettingsSetter.h"
void ResetValue(uint8 InCategory);
```
```C++ title="EasySettingsSetter.cpp"
Values[InCategory].bDefaultValue = true;
Values[InCategory].Value = 0.0f;
```

## Easy Settings Subsystem

```C++ title="EasySettingsSubsystem.h"
class EASYSETTINGS_API UEasySettingsSubsystem : public UGameInstanceSubsystem
```

A subsystem that extends game settings, includes changing console variables and more.

Features:

- SetSettingsQuality, GetSettingsQuality
- SetAntialiasingMethod, GetAntialiasingMethod
- SetOverallQualityLevel
- SetVsyncEnabled, GetVsyncEnabled
- SetFrameRateLimit, GetFrameRateLimit
- SetWindowedMode, GetCurrentWindowedMode, GetFullscreenMode
- SetResolution, GetCurrentResolution
- GetSupportedResolutions
- SetScreenPercentage, GetScreenPercentage
- SetContainerValue, GetContainerValue, ResetContainerValue
- ApplySettings
- ApplyContainer

### ESettingsType

An enumeration representing different types of graphical settings in the game.

```C++
enum class ESettingsType : uint8
{
	TYPE_NONE UMETA(Hidden),
	TYPE_AA UMETA(DisplayName="Anti Aliasing"),
	TYPE_Textures UMETA(DisplayName="Textures"),
	TYPE_Effects UMETA(DisplayName="Effects"),
	TYPE_Shadows UMETA(DisplayName="Shadows"),
	TYPE_Foliage UMETA(DisplayName="Foliage"),
	TYPE_Reflection UMETA(DisplayName="Reflection"),
	TYPE_GlobalIllumination UMETA(DisplayName="Global Illumination"),
	TYPE_ViewDistance UMETA(DisplayName="View Distance"),
	/** Represents the maximum value for this enum, used internally. */
	TYPE_MAX UMETA(Hidden)
};
```

### Set Settings Quality

Sets the quality values for the selected setting. 

Can be useful when using combo boxes together with **ESettingsType** enumeration.

```C++ title="EasySettingsSubsystem.h"
void SetSettingsQuality(ESettingsType InSettingsType = ESettingsType::TYPE_Effects, 
                        int32 InQuality = 3,
	                    bool bApply = true);
```

!!! note
    If bApply == true, it will call **ApplySettings()**

### Get Settings Quality

Retrieves the current quality level of a specific setting type.

```C++ title="EasySettingsSubsystem.h"
int32 GetSettingsQuality(ESettingsType InSettingsType = ESettingsType::TYPE_Effects) const;
```

### Set Antialiasing Method 

Sets the Anti-Aliasing method.

```C++ title="EasySettingsSubsystem.h"
void SetAntialiasingMethod(
		APlayerController* InController,
		int32 InValue,
		bool bApply = true);
```

### Get Antialiasing Method

Retrieves the current Anti-Aliasing method.

```C++ title="EasySettingsSubsystem.h"
int32 GetAntialiasingMethod() const;
```

### Set Overall Quality Level

Sets the Overall quality level.

```C++ title="EasySettingsSubsystem.h"
void SetOverallQualityLevel(int32 InValue, bool bApply = true);
```

### Set Vsync Enabled

Enables or disables VSync.

```C++ title="EasySettingsSubsystem.h"
void SetVsyncEnabled(bool bInValue, bool bApply = true);
```

### Get Vsync Enabled

Checks whether VSync is currently enabled.

```C++ title="EasySettingsSubsystem.h"
bool GetVsyncEnabled() const;
```

### Set FrameRate Limit

Sets the frame rate limit.

```C++ title="EasySettingsSubsystem.h"
void SetFrameRateLimit(int32 InValue, bool bApply = true);
```

### Get FrameRate Limit

Retrieves the current frame rate limit.

```C++ title="EasySettingsSubsystem.h"
int32 GetFrameRateLimit() const;
```

### Set Windowed Mode

Sets the window mode (fullscreen, windowed, or borderless window).

```C++ title="EasySettingsSubsystem.h"
void SetWindowedMode(
	TEnumAsByte<EWindowMode::Type> InWindowMode = EWindowMode::Type::Windowed,
	bool bApply = true);
```

### Get Current Windowed Mode

Retrieves the current window mode.

```C++ title="EasySettingsSubsystem.h"
TEnumAsByte<EWindowMode::Type> GetCurrentWindowedMode() const;
```

### Get FullscreenMode

This function returns the current fullscreen mode as configured in the game's settings. The possible modes
include fullscreen, windowed, and borderless windowed.

```C++ title="EasySettingsSubsystem.h"
TEnumAsByte<EWindowMode::Type> GetFullscreenMode() const { return GetGameUserSettings()->GetFullscreenMode(); }
```

### Set Resolution

Sets the screen resolution.

```C++ title="EasySettingsSubsystem.h"
void SetResolution(FIntPoint InResolution, bool bApply = true);
```

### Get Current Resolution

Retrieves the current screen resolution.

```C++ title="EasySettingsSubsystem.h"
FIntPoint GetCurrentResolution() const;
```

### Get Supported Resolutions

Retrieves the supported screen resolutions based on the selected window mode.
```C++ title="EasySettingsSubsystem.h"
void GetSupportedResolutions(TArray<FIntPoint>& OutResult,
	                            TEnumAsByte<EWindowMode::Type> InWindowMode = EWindowMode::Type::Windowed);
```

### Set Screen Percentage

Sets the screen percentage scaling factor for rendering.

```C++ title="EasySettingsSubsystem.h"
void SetScreenPercentage(int32 InValue = 100, bool bApply = true);
```

!!! note
    This function allows modifying the `r.ScreenPercentage` console variable, 
    which controls the resolution scale of the rendered scene.

### Get Screen Percentage

Retrieves the current screen percentage scaling factor.

```C++ title="EasySettingsSubsystem.h"
int32 GetScreenPercentage() const;
```
!!! note
    This function reads the value of the `r.ScreenPercentage` console variable, 
    which determines the resolution scale used for rendering.

### Set Container Value

Sets the container value for a specific category.

```C++ title="EasySettingsSubsystem.h"
void SetContainerValue(uint8 InCategory, float InValue, bool bApply = true);
```

!!! note
    his method sets the float value for a given category in the container. 
    If the `bApply` flag is true, the settings are applied immediately after setting the value.

### Get Container Value

Retrieves the container value for a specific category.

```C++ title="EasySettingsSubsystem.h"
void GetContainerValue(uint8 InCategory, float& OutValue,
		bool& bDefault);
```

### Reset Container Value

Resets the container value.

```C++ title="EasySettingsSubsystem.h"
void ResetContainerValue(uint8 InCategory, bool bApply = true);
```
```C++
void UEasySettingsSetter::ResetValue_Implementation(uint8 InCategory)
{
	if (Values.Contains(InCategory))
	{
		Values[InCategory].bDefaultValue = true;
		Values[InCategory].Value = 0.0f;
	}
}
```

### Apply Settings

Applies the current settings, saving them to the user's configuration file.

```C++ title="EasySettingsSubsystem.h"
void ApplySettings();
```

### Apply Container

Applies the current container values, saving them to the user's configuration file.

```C++ title="EasySettingsSubsystem.h"
void ApplyContainer();
```