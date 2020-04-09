---
title: UE4中RPC的序列化
categories: server
---  
  
UE4的网络同步主要分为属性同步和RPC调用两种方式。本文主要关注RPC调用中的序列化。
  
##  1. 网络概况  
  
`NetDriver`是网络处理的核心类，有三种类型的Driver：
  
- The Game NetDriver：负责主要的游戏网络交换
- The Demo NetDriver：记录数据，不会发送数据，用于回放重播系统。
- The Beacon NetDriver：负责除了游戏外的一些网络交互。
  
通常一个游戏，服务器只有一个Game NetDriver，NetDriver管理NetConnection列表，一个NetConnection就是一个玩家。Netconnection负责同步玩家所有channel的数据，包括一个语音数据channel，一个控制channel，所有同步actor的channel。
NetDriver通过Fsocket和socketSubSystem来完成网络状态查询、发包、收包。
  

![](/assets/1246a47dc1e4bd641fe4c56724e0622f0.png?0.1395610416772688)  
  
###  初始化，建立连接
  
  
当服务器加载地图（UEngine::LoadMap），就会调用`UWorld::Listen`监听端口。客服端就可以连接服务器了。`SocketSubsystem->CreateSocket()`根据所在平台，创建socket。`FSocket`来包装socket，提供跨平台。
  
在`UIpNetDriver::InitBase`中创建socket，并设置socket属性，发生接收buffer大小，开启广播，非阻塞等，最后绑定端口。
  
`UNetDriver::TickDispatch` 负责接收网络数据，根据数据包的地址来判断是否已经建立连接，还没建立连接的就会先进行握手。或者路由到对应connection，并调用`Connection->ReceivedRawPacket`。
  
UDP容易受到各种DOS攻击，特别是伪装IP地址，所以需要握手来判断IP地址的合法性。UE4用无状态的handshake，避免欺骗包消耗服务器的内存。
  
**玩家登陆流程，包含两次握手**：

![](/assets/1246a47dc1e4bd641fe4c56724e0622f1.png?0.8191023006415554)  
  
有时候，路由器会改变包的端口，收到还未握手的包，服务器会发送1个字节的回包，让其重新开启握手。
  
game level handshaking阶段，是用`NetControlMessage`完成。
  
##  2. 协议
  
  
每次channel的属性同步和RPC调用会封装成一个Bunch，最后所有bunch会合并成一个packet发送。一个完整的包就是一个packet，一个packet可能包含0个或多个bunch。
  
**packet组成大致如下**：

![](/assets/1246a47dc1e4bd641fe4c56724e0622f2.png?0.9483044965502683)  
  
packet header包含序列号和对应连接等信息，服务器根据信息路由到对应连接。bunch header有许多标志位。包括是否完整的Bunch，是否是可靠等。
  
packet是用结束标示来识别完整的packet。应该是因为UE4限制了包最大为MTU。所以基本不会有拆包的情况，收到包判断最后一个标志位大概率会直接成功。
  
packet header和bunch header随着版本变化，格式也可能发生变化。在`EEngineNetworkVersionHistory`定义了协议的历史版本号。
  
###  bunch
  
  
可以看到，主要的数据都是通过bunch传输的。属性同步和RPC调用的bunch格式是不一样的。
**RPC的bunch:**

![](/assets/1246a47dc1e4bd641fe4c56724e0622f3.png?0.9531045575118813)  
  
**属性同步的bunch：**

![](/assets/1246a47dc1e4bd641fe4c56724e0622f4.png?0.31515380331273524)  
  
##  3. RPC序列化
  
  
定义一个RPC的时候，需要加上`UFUNCTION`注解。这是UE4的反射系统。
```c++
UFUNCTION(Server, Reliable)
void HandleFire(float damage,const FMyData& datas, const TArray<uint32>& items);
```
  
除了`UFUNCTION`还有`UStruct`，`UProperty`等注解。在编译的时候unreal的编译工具会根据注解来生成反射所需要的代码。存储在·per-module.generated.inl·和·per-header.generated.h·中。
  
**反射的类型继承：**
```
UField
	UStruct
		UClass (C++ class)
		UScriptStruct (C++ struct)
		UFunction (C++ function)
  
	UEnum (C++ enumeration)
  
	UProperty (C++ member variable or function parameter)
			(Many subclasses for different types)
```
  
注解的参数在ObjectBase.h中有简单的注释，可以帮助理解它所代表的意思。
  
###  参数序列化
  
UE4反射工具会生成对应结构体来合并RPC函数的所有参数。参数由RPC对应的`FRepLayout`来序列化。`FPepLayout`描述了参数的类型Type，内存布局等。给定的结构体或RPC会对应一个FRepLayout实例。
```c++
//自动生成的结构体 shootCharacter.gen.cpp
void AshootCharacter::HandleFire(float damage, FMyData const& datas, TArray<uint32> const& items)
{
	shootCharacter_eventHandleFire_Parms Parms;
	Parms.damage=damage;
	Parms.datas=datas;
	Parms.items=items;
	ProcessEvent(FindFunctionChecked(NAME_AshootCharacter_HandleFire),&Parms);
}
FRepLayout::SerializeProperties_r(){
	.....
	// 序列化属性
	Cmd.Property->NetSerializeItem(TempWriter, TempWriter.PackageMap, const_cast<uint8*>(Data.Data));
	uint32 NumBits = TempWriter.GetNumBits();
	Writer.SerializeIntPacked(NumBits);
	Writer.SerializeBits(TempWriter.GetData(), NumBits);
}
  
```
  
参数或属性最终调用`UProperty->NetSerializeItem`方法将数据序列化到bunch中。UProperty是参数的反射类。函数签名如下：
```c++
virtual bool NetSerializeItem( FArchive& Ar, UPackageMap* Map, void* Data, TArray<uint8> * MetaData = NULL ) const;
```
  
序列化的时候会传入`FArchive`,以访问者模式来为UField提供各种序列化方法。由访问者`FArchive`来决定如何将数据转换为字节流，并且由`FArchive`持有这个字节流。Bunch实际上就是`FArchive`的子类。
```
FArchive
	FBitArchive
		FBitReader
			FNetBitReader (serializes FNames and UObject* through a network packagemap.)
				FInBunch
		FBitWriter
			FNetBitWriter
				FOutBunch
	...
```
  
  
  
FArchive通过重载`<<`来实现序列化方法。并且同时实现序列化和反序列化，当FArchive是写模式，`<<`为序列化方法；当Farchive是读模式，`<<`为反序列化方法。一个操作符干两件事，如果你刚开始看UE4代码会感觉比较困惑。
  
  
  
```c++
friend FArchive& operator<<(FArchive& Ar, uint16& Value)
{
	Ar.ByteOrderSerialize(&Value, sizeof(Value));
	return Ar;
}
friend FArchive& operator<<(FArchive& Ar, uint32& Value)
{
	Ar.ByteOrderSerialize(&Value, sizeof(Value));
	return Ar;
}
```
  
###  一个实例
  
默认实现的序列化，会按照原始内存布局序列化到字节流里，字节利用率并不高。如果调用RPC发送一个uint32的数组,不管里面uint32的值再小也占4个字节。比如下面的RPC调用，参数为23个uint32的数据，通过ue4提供独立的程序 `/Engine/Binaries/DotNET/NetworkProfiler.exe`对网络性能进行监控，可以看到bunch就占用了103个字节。
  
```c++
const TArray<uint32> data = {0,1,2,3,4,5,6,7,8,9,10,11,10,9,8,7,6,5,4,3,2,1,0};
HandleFire(data);
```

![1](/assets/serialization1.png )

所以在RPC参数类型选择上，应该尽量选择占用空间小的。能uint8表示的就不用uint32。
```c++
UFUNCTION(unreliable, server, WithValidation)
void ServerMove(float TimeStamp, FVector_NetQuantize10 InAccel, FVector_NetQuantize100 ClientLoc, uint8 CompressedMoveFlags, uint8 ClientRoll, uint32 View, UPrimitiveComponent* ClientMovementBase, FName ClientBaseBoneName, uint8 ClientMovementMode);
```
  
###  自定义序列化
  
  
当RPC的参数或者同步的属性是Struct的时候，可以自定义序列化方法。将结构体压缩序列化后再发送。加入自定义序列化方法如下：
```c++
USTRUCT()
struct FMyCustomNetSerializableStruct
{
	UPROPERTY()
	float SomeProperty;
	//定义序列化方法 step1
	bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess);
}
//通知引擎调用自定义序列化方法 step2
template<>
struct TStructOpsTypeTraits<FMyCustomNetSerializableStruct> : public TStructOpsTypeTraitsBase2<FMyCustomNetSerializableStruct>
{
	enum
	{
		WithNetSerializer = true
	};
};
```
UE4引擎的移动数据`FRepMovement`的同步，就用到了自定义序列化。我们可以看到如何用自定义序列化提高字节的利用率。
```c++
bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
{
	// pack bitfield with flags
	uint8 Flags = (bSimulatedPhysicSleep << 0) | (bRepPhysics << 1);
	Ar.SerializeBits(&Flags, 2);
	bSimulatedPhysicSleep = ( Flags & ( 1 << 0 ) ) ? 1 : 0;
	bRepPhysics = ( Flags & ( 1 << 1 ) ) ? 1 : 0;
	bOutSuccess = true;
	// update location, rotation, linear velocity
	bOutSuccess &= SerializeQuantizedVector( Ar, Location, LocationQuantizationLevel );	
	switch(RotationQuantizationLevel)
	{
		case ERotatorQuantization::ByteComponents:
		{
			Rotation.SerializeCompressed( Ar );
			break;
		}
		case ERotatorQuantization::ShortComponents:
		{
			Rotation.SerializeCompressedShort( Ar );
			break;
		}
	}	
	bOutSuccess &= SerializeQuantizedVector( Ar, LinearVelocity, VelocityQuantizationLevel );
	// update angular velocity if required
	if ( bRepPhysics )
	{
		bOutSuccess &= SerializeQuantizedVector( Ar, AngularVelocity, VelocityQuantizationLevel );
	}
	return true;
}
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
}
```
UE4提供了一些方法来更高效的序列化动量和向量。`FRepMovement`就用到了`SerializeQuantizedVector`来序列化坐标和速度等向量。
  
`FRepMovement`的旋转属性（Rotator）的序列化加入了1bit的标志位，用来表示这个属性是否为0。不为0的属性才会写入字节流。这样旋转向量中大量0的字段只会占用1bit。
  
###  增量序列化
  
  
除了`NetSerialization`还可以自定义`NetDeltaSerialization`。增量序列化（delta serialization）会比较之前的状态，只发送改变的数据。这适合需要持续同步某个结构体或数据，所以只会用在属性同步。
  
NetDeltaSerialization的主要应用就是`Fast TArray Replication`。如果你想高效的同步数组（Tarray），或者在数组增加或删除数据的时候客服端会收到事件，可以用到`Fast TArray Replication`。具体的使用方法在`NetSerialization.h`里有说明。
`
  
##  结论
  
  
1. 用UE4提供的向量序列化方法来更高效的序列化向量。
1. RPC的参数尽量选用占用空间更小的类型。能用int8表示参数就不要用int32。
1. 默认的属性序列化字节利用率不高，如果是大字节或者调用频繁的RPC，应该自定义序列化方法。
  
(The end)
  
##  参考
  
  
>1. http://www.aclockworkberry.com/custom-struct-serialization-for-networking-in-unreal-engine/
>1. https://blog.csdn.net/mohuak/article/details/83027211
> 2. replayout.h netdriver.h netserializtion.h(自定义序列化 replayout 注释
> 1. https://blog.ch-wind.com/ue4-network-overview/
>1. https://blog.uwa4d.com/archives/USparkle_Exploring.html
> 3. 反射：https://zhuanlan.zhihu.com/p/60622181
>4. 反射：https://www.unrealengine.com/zh-CN/blog/unreal-property-system-reflection
> 1. https://www.gameres.com/844472.html
  