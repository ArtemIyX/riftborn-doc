# Advanced Sessions

## Overview

A Blueprint Library Plugin that exposes additional Networking/Session/OnlineSubsystem/Friends/Voice features to Blueprints that were missing.

- [Unreal Engine Forum](https://forums.unrealengine.com/t/advanced-sessions-plugin/30020)
- [Github Repo](https://github.com/mordentral/AdvancedSessionsPlugin)
- [Docs](https://www.vreue4.com/GeneratedDocs/AdvancedSessions/)

## Examples

Riftborn uses many helper methods from these libraries.

```C++ title="Calculate players hash"

TArray<TObjectPtr<APlayerState>> copy = InPlayers;
uint32 hash = 0;

for (int32 i = 0; i < copy.Num(); ++i)
{
	FBPUniqueNetId netId;
	UAdvancedSessionsLibrary::GetUniqueNetIDFromPlayerState(copy[i].Get(), netId);

	FString id;
	UAdvancedSessionsLibrary::UniqueNetIdToString(netId, id);

	hash = FCrc::StrCrc32(*id, hash);
	if (AHumPlayerState* item = Cast<AHumPlayerState>(copy[i].Get()))
	{
		FString team = UEnum::GetValueAsString(item->GetTeam());
		hash = FCrc::StrCrc32(*team, hash);

		FString selectedChar = item->GetSelectedCharacter().PrimaryAssetName.ToString();
		hash = FCrc::StrCrc32(*selectedChar, hash);
	}
}
return hash;

```

```C++ title="Player killed someone"
FBPUniqueNetId a;
UAdvancedSessionsLibrary::GetUniqueNetIDFromPlayerState(PlayerInstigator, a);
FBPUniqueNetId b;
UAdvancedSessionsLibrary::GetUniqueNetIDFromPlayerState(PlayerInstigator, b);

FString aStr;
UAdvancedSessionsLibrary::UniqueNetIdToString(a, aStr);
FString bStr;
UAdvancedSessionsLibrary::UniqueNetIdToString(b, bStr);

TRACE(LogHUM, "%s (%s) has killed %s (%s)",
	*PlayerInstigator->GetPlayerName(),
	*aStr,
	*PlayerInstigator->GetPlayerName(),
	*bStr);
```