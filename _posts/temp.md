## 压缩

You can leverage some quantization functionalities exposed by the engine such has Vector quantization and Quaternion quantization。

### struct
当RPC的参数或者同步的属性是Struct的时候。可以自定义序列化方法。将结构体压缩序列化后再发送。

```c++
USTRUCT()
struct FMyCustomNetSerializableStruct
{
	UPROPERTY()
	float SomeProperty;
 
	bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess);
}
 
template<>
struct TStructOpsTypeTraits<FMyCustomNetSerializableStruct> : public TStructOpsTypeTraitsBase2<FMyCustomNetSerializableStruct>
{
	enum
	{
		WithNetSerializer = true
	};
};
```

```c++
// Runtime/Core/Private/Math/UnrealMath.cpp
void FRotator::SerializeCompressed( FArchive& Ar )
{
	uint8 BytePitch = FRotator::CompressAxisToByte(Pitch);
	uint8 ByteYaw = FRotator::CompressAxisToByte(Yaw);
	uint8 ByteRoll = FRotator::CompressAxisToByte(Roll);

	uint8 B = (BytePitch!=0);
	Ar.SerializeBits( &B, 1 );
	if( B )Ar << BytePitch; else    BytePitch = 0;
        ....
	
	if( Ar.IsLoading() )
	{
		Pitch = FRotator::DecompressAxisFromByte(BytePitch);
		Yaw = FRotator::DecompressAxisFromByte(ByteYaw);
		Roll = FRotator::DecompressAxisFromByte(ByteRoll);
	}
}
```
用1bit来表示这个属性是否为0，不为0的属性才会写入字节流。

###  Fast TArray Replication

UE4实现对基本数据类型（int，float..）通用序列化方法，也对TArray实现了通用的增量序列化（delta serialization）

Custom net delta serialization is mainly used in combination with fast TArray replication

Basically, if you want to replicate a TArray efficiently, or if you want events to be called on client for adds and removal, just wrap the array into a ustruct and use FTR.  Here is what the code documentation says about FTR:

Fast TArray Replication is a custom implementation of NetDeltaSerialize that is suitable for TArrays of UStructs

```c++
/** Step 2: You MUST wrap your TArray in another struct that inherits from FFastArraySerializer */
USTRUCT()
struct FExampleArray: public FFastArraySerializer
{
	GENERATED_USTRUCT_BODY()

	UPROPERTY()
	TArray<FExampleItemEntry>	Items;	/** Step 3: You MUST have a TArray named Items of the struct you made in step 1. */

	/** Step 4: Copy this, replace example with your names */
	bool NetDeltaSerialize(FNetDeltaSerializeInfo & DeltaParms)
	{
	   return FFastArraySerializer::FastArrayDeltaSerialize<FExampleItemEntry, FExampleArray>( Items, DeltaParms, *this );
	}
};
```


```c++
/** Various types of Properties supported for Replication. */
enum class ERepLayoutCmdType : uint8
{
	DynamicArray			= 0,	//! Dynamic array
	Return					= 1,	//! Return from array, or end of stream
	Property				= 2,	//! Generic property

	PropertyBool			= 3,
	PropertyFloat			= 4,
	PropertyInt				= 5,
	PropertyByte			= 6,
	PropertyName			= 7,
	PropertyObject			= 8,
	PropertyUInt32			= 9,
	PropertyVector			= 10,
	PropertyRotator			= 11,
	PropertyPlane			= 12,
	PropertyVector100		= 13,
	PropertyNetId			= 14,
	RepMovement				= 15,
	PropertyVectorNormal	= 16,
	PropertyVector10		= 17,
	PropertyVectorQ			= 18,
	PropertyString			= 19,
	PropertyUInt64			= 20,
	PropertyNativeBool		= 21,
	PropertySoftObject		= 22,
	PropertyWeakObject		= 23,
};
```

FArchive重载`<<`来同时实现序列化和反序列化，当FArchive是写模式，`<<`为序列化方法；当Farchive是读模式，`<<`为反序列化方法。一个操作符干两件事，如果你刚开始看UE4代码会感觉比较困惑。

调用RPC发送一个uint32的数组。UE4是不会对uint32的数据做压缩的，也就是uint32的值再小也占4个字节。

```c++
const TArray<uint32> data = {0,1,2,3,4,5,6,7,8,9,10,11,10,9,8,7,6,5,4,3,2,1,0};
HandleFire(30.0f,data);
```

![1](/assets/serialization1.png)