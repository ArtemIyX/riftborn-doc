# Data Serializer

> [Github Repository](https://github.com/ArtemIyX/DataSerializerUnreal)

## Overview

A set of utilities for convenient data serialization

## Data Serializer Lib

```C++
class DATASERIALIZER_API UDataSerializerLib : public UBlueprintFunctionLibrary
```

Features:

- WriteBytesToDisk, WriteBytesToDiskCompressed
- ReadBytesFromDisk, ReadCompressedBytesFromDisk
- SerializeObject, DeserializeObject
- SerializeObjects, DeSerializeObjects
- GetUtf8Bytes, Utf8BytesToString

## Write Bytes

You can write an array of bytes to disk using the following functions.

```C++
static bool WriteBytesToDisk(const TArray<uint8>& InBytes, FString InPath);
static bool WriteBytesToDiskCompressed(const TArray<uint8>& InBytes, FString InPath);
```

!!! note
    WriteBytesToDiskCompressed will write compressed bytes to disk, which will weigh less than the original byte array

## Read Bytes

You can load an array of bytes from disk using the following functions.

```C++
static bool ReadBytesFromDisk(TArray<uint8>& OutBytes, FString InPath);
static bool ReadCompressedBytesFromDisk(TArray<uint8>& OutBytes, FString InPath);
```

!!! note
    ReadCompressedBytesFromDisk should be used if the bytes were written using WriteBytesToDiskCompressed

## Serialize

!!! warning
    FObjectAndNameAsStringProxyArchive and FSerializationHeader are used for object serialization

You can serialize a single object into bytes using the following function.

```C++
static bool SerializeObject(TArray<uint8>& OutBytes, UObject* InObject);
```

!!! note
    **UObject::Serialize()** is used to serialize an object into bytes. Overwrite it for custom serialization or use UPROPERTY for default data types.

You can serialize an array of objects into bytes.

```C++
static bool SerializeObjects(TArray<uint8>& OutBytes, TArray<UObject*> InObjects);
```

!!! info 
    At the beginning of the record will be the size of the array for further correct loading from disk.



## Deserialize

You can deserialize an array of bytes into an object or array of objects.

!!! tip
    Use **DeSerializeObjects()** if you firstly used **SerializeObjects()**.

```C++
static bool DeserializeObject(const TArray<uint8>& InBytes, UObject* ObjectOuter, UObject*& OutObject);
static bool DeSerializeObjects(const TArray<uint8>& InBytes, UObject* InObjectOuter, TArray<UObject*>& OutObjects);
```

!!! note
    Will return **false** if the bytes were invalid.

## UTF8 Bytes

UTF8 encoding is often used to transmit string bytes over a network.

Use the following methods to serialize a FString into UTF8 bytes and back bytes into a FString

```C++
static void GetUtf8Bytes(const FString& InString, TArray<uint8>& OutBytes);
static FString Utf8BytesToString(const TArray<uint8>& InBytes);
```

## Serialize / Deserialize Object

You can use **USerializerObject** and **UDeSerializerObject** for more convenience workflow in Blueprints.

This is a wrapper over [FMemoryWriter](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Serialization/FMemoryWriter) and [FMemoryReader](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Serialization/FMemoryReader?application_version=5.5)

### Serialize Object

Wrapper of [FMemoryWriter](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Serialization/FMemoryWriter)

Features:

- Clear
- Prepare
- GetBytes
- PushBytes
- Serialize (int32, int64, float, double, bool, vector, vector2d, point, rotator, transform, string, object, objects[])

---
#### Clear

Clears all data and the pointer to MemoryWriter.

```C++
virtual void Clear();
```

#### Prepare

Clears the data and creates a new MemoryWriter.

```C++
virtual void Prepare();
```

#### GetBytes

Getter for bytes. Returns the current array with bytes. Useful for writing bytes to disk after all data has been serialized

```C++
virtual void GetBytes(TArray<uint8>& OutBytes);
```

#### PushBytes

Appends bytes to the current byte array.

Useful in conjunction with UDataSerializerLib.

```C++
virtual void PushBytes(const TArray<uint8>& InBytes);
```

#### Serialize

Write bytes to an array of bytes using the << operator of FMemoryWriter.

```C++
void USerializerObject::SerializeInt(int32 InInteger) { GetMemoryWriterRef() << InInteger; }
void USerializerObject::SerializeBigInt(int64 InBigInt) { GetMemoryWriterRef() << InBigInt; }
void USerializerObject::SerializeFloat(float InFloat) { GetMemoryWriterRef() << InFloat; }
void USerializerObject::SerializeDouble(double InDouble) { GetMemoryWriterRef() << InDouble; }
void USerializerObject::SerializeBool(bool InBool) { GetMemoryWriterRef() << InBool; }
void USerializerObject::SerializeByte(uint8 InByte) { GetMemoryWriterRef() << InByte; }
void USerializerObject::SerializeVector(FVector InVector) { GetMemoryWriterRef() << InVector; }
void USerializerObject::SerializeIntVector(FIntVector InVector) { GetMemoryWriterRef() << InVector; }
void USerializerObject::SerializeVector2D(FVector2D InVector) { GetMemoryWriterRef() << InVector; }
void USerializerObject::SerializePoint(FIntPoint InPoint) { GetMemoryWriterRef() << InPoint; }
void USerializerObject::SerializeRotator(FRotator InRotator) { GetMemoryWriterRef() << InRotator; }
void USerializerObject::SerializeTransform(FTransform InTransform) { GetMemoryWriterRef() << InTransform; }
void USerializerObject::SerializeString(FString InString) { GetMemoryWriterRef() << InString; }
```

### DeSerialize Object

Wrapper of [FMemoryReader](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Serialization/FMemoryReader?application_version=5.5)

Features:

- Clear
- Start
- Try Read

#### Clear

Resets FMemoryReader pointer.

```C++
virtual void Clear();
```

#### Start

Clears the data and creates a new MemoryReader.

```C++
virtual void Start(const TArray<uint8>& InBytes);
```

#### C++ Try Read

It tries to turn deserialize current bytes into a specific data type using the << operator of FMemoryReader

```C++
template <typename T>
bool TryReadT(T& OutValue);
```

!!! warning
    Returns **false** if FMemoryReader returned IsError() or IsCriticalError()

#### Blueprints Try Read

Uses **TryReadT(...)** to return a specific type in blueprints.

UDataSerializerLib is used for object deserialization

```C++
bool UDeSerializerObject::TryReadInt(int32& OutInt) { return TryReadT(OutInt); }
bool UDeSerializerObject::TryReadInt64(int64& OutInt64) { return TryReadT(OutInt64); }
bool UDeSerializerObject::TryReadFloat(float& OutFloat) { return TryReadT(OutFloat); }
bool UDeSerializerObject::TryReadDouble(double& OutDouble) { return TryReadT(OutDouble); }
bool UDeSerializerObject::TryReadBool(bool& OutBool) { return TryReadT(OutBool); }
bool UDeSerializerObject::TryReadUInt8(uint8& OutUInt8) { return TryReadT(OutUInt8); }
bool UDeSerializerObject::TryReadVector(FVector& OutVector) { return TryReadT(OutVector); }
bool UDeSerializerObject::TryReadIntVector(FIntVector& OutIntVector) { return TryReadT(OutIntVector); }
bool UDeSerializerObject::TryReadVector2D(FVector2D& OutVector2D) { return TryReadT(OutVector2D); }
bool UDeSerializerObject::TryReadIntPoint(FIntPoint& OutIntPoint) { return TryReadT(OutIntPoint); }
bool UDeSerializerObject::TryReadRotator(FRotator& OutRotator) { return TryReadT(OutRotator); }
bool UDeSerializerObject::TryReadTransform(FTransform& OutTransform) { return TryReadT(OutTransform); }
bool UDeSerializerObject::TryReadString(FString& OutString) { return TryReadT(OutString); }
bool UDeSerializerObject::TryReadObject(UObject* InObjectOuter, UObject*& OutObject)
{
	FMemoryReader& memoryReader = GetMemoryReaderRef();
	return UDataSerializerLib::DeSerializeObjectCpp(memoryReader, InObjectOuter, OutObject);
}

bool UDeSerializerObject::TryReadObjects(UObject* InObjectOuter, TArray<UObject*>& OutObjects)
{
	FMemoryReader& memoryReader = GetMemoryReaderRef();
	return UDataSerializerLib::DeSerializeObjectsCpp(memoryReader, InObjectOuter, OutObjects);
}

```


## Example

```C++
void UEasySettingsSubsystem::SaveContainer()
{
	if (!IsValid(SettingsSetter))
		return;

	// Prepare empty byte container
	TArray<uint8> bytes;
	FMemoryWriter writer(bytes);

	// Write bytes from settings
	SettingsSetter->Write(writer);

	// Save to file
	FString path = GetContainerSavePath();
	UDataSerializerLib::WriteBytesToDiskCompressed(bytes, path);
}

void UEasySettingsSetter::Write(FMemoryWriter& MemoryWriter)
{
	int n = EasySettings::VALUES_NUM;
	check((Values.Num() == n))
	// Write each float element
	for (const TTuple<EasySettings::MapKey, EasySettings::MapValue>& pair : Values)
	{
		bool bDefault = pair.Value.bDefaultValue;
		MemoryWriter << bDefault;
		float value = pair.Value.Value;
		MemoryWriter << value;
	}
}

```