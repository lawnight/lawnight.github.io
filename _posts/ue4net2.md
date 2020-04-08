

## 网络组件

### NetDriver，Connection，Channel

`NetDriver`是网络处理的核心，有三种类型的Driver：

- The Game NetDriver：负责主要的游戏网络交换
- The Demo NetDriver：记录数据，不会发送数据，用于回放重播系统。
- The Beacon NetDriver, responsible for network traffic that falls outside of "normal" gameplay traffic.

通常一个游戏，服务器只有一个Game NetDriver，NetDriver管理NetConnection列表，一个NetConnection就是一个玩家。Netconnection负责同步玩家所有channel的数据，包括Ac一个语音数据channel，一个控制channel，所以同步actor的channel。

FObjectReplicator:属性同步的执行器，每个Actorchannel对应一个FObjectReplicator，每一个FObjectReplicator对应一个对象实例。设置ActorChannel通道的时候会创建出来。

```puml
@startuml
package "NetDriver" {  
   [connections] ->NetConnection 
}
package Connection  {
  [control channel]
  [多个actor channel] ->channel
  [voice channel]
}
note left of connections:一个连接对应一个玩家

note right of [多个actor channel]:一个能同步的actor对应一个actor channel
package Channel{
}

package FInBunch{
    [PackageMap]
    [Buffer] ->TArrayint8
    [flags]
    [ Connection]
}

package FObjectReplicator  {
  [repLayout]
}

package UClass{
    
}

note left of [repLayout]:根据uclass创建出来，在netdrive中缓存
note left of [PackageMap]:Maps objects and names,来自UNetConnection


NetConnection -down->Connection
channel -down->Channel

@enduml
```


### FSocket和SocketSubSystem

FSocket是对网络底层的封装，对于上层需要的网络状态查询、发包、收包这些操作，全部都封装在了这里。屏蔽了与平台相关的底层。






### 传输

当connection收到packet的时候，会拆分的bunches列表，然后派分到对应的channel。

Example: Client RPC to Server.

- Client makes a call to Server_RPC.
- That request is forwarded (via NetDriver and NetConnection) to the Actor Channel that owns the Actor on which he RPC was called.
- The Actor Channel will serialize the RPC Identifier and parameters into a Bunch. The Bunch will also contain he ID of its Actor Channel.
- The Actor Channel will then request the NetConnection send the Bunch.
- Later, the NetConnection will assemble this (and other) data into a Packet which it will send to the server.
- On the Server, the Packet will be received by the NetDriver.
- The NetDriver will inspect the Address that sent the Packet, and hand the Packet over to the appropriate etConnection.
- The NetConnection will disassemble the Packet into its Bunches (one by one).
- The NetConnection will use the Channel ID on the bunch to Route the bunch to the corresponding Actor Channel.
-  ActorChannel will them disassemble the bunch, see it contains RPC data, and use the RPC ID and serialized arameters
    to call the appropriate function on the Actor.

每个包都有自增的序列号。


```puml
@startuml
:UNetConnection::ReceivedRawPacket( void* InData, int32 Count );
:UNetConnection::ReceivedPacket( FBitReader& Reader );
note right:解析bunch头部并派发bunch;ack;
:UChannel::ReceivedRawBunch( FInBunch & Bunch, bool & bOutSkipAck );
:UChannel::ReceivedSequencedBunch( FInBunch & Bunch);
:UActorChannel::ReceivedBunch( FInBunch & Bunch );
note right:合并完整的buch后派发到对应的bunch
:UActorChannel::ProcessBunch( FInBunch & Bunch );
note right:Read chunks of actor content
:UActorChannel::ReadContentBlockHeader( FInBunch & Bunch, bool& bObjectDeleted, bool& bOutHasRepLayout );
note right:islayout,isactor,len
:Connection->CreateReplicatorForNewActorChannel(Obj)
note right:创建actorReplication

:FObjectReplicator::ReceivedBunch;
:FRepLayout::ReceiveProperties;
while (读取所有属性和RPC)
    :OwningChannel->ReadFieldHeaderAndPayload Read each property/function blob into Reader;
    note right:reqindex,len -> reader;
    if (filed类型) then (UStructProperty)
        :FNetSerializeCB::ReceiveCustomDeltaProperty;
        :FRepLayout::ReceiveCustomDeltaProperty(ReceivingRepState, Params, ReplicatedProp);
        :UScriptStruct::ICppStructOps类型 CppStructOps->NetDeltaSerialize(Params, Params.Data);
    else (UFunction)
        :ReceivedRPC;
        :FRepLayout::ReceivePropertiesForRPC;
    
    endif
endwhile
stop
@enduml
```


### 消息收发流程

`World::Tick`来驱动整个游戏。

`net.UseRecvMulti`，`net.IpNetDriverUseReceiveThread`配置接收网络消息单独线程。不用game thread接收。因为udp协议，没有连接，也就不用多路复用。





`IpNetDriver`

 * End point data isn't directly handled by NetConnections. Instead NetConnections will route data to Channels.
 * Every NetConnection will have its own set of channels.
 *
 * Common types of channels:
 *
 *	- A Control Channel is used to send information regarding state of a connection (whether or not the connection should close, etc.)
 *	- A Voice Channel may be used to send voice data between client and server.
 *	- A Unique Actor Channel will exist for every Actor replicated from the server to the client.

`TUniquePtr`

## 反射
c++不支持原生的反射。UE4自己实现了一套反射系统。属性的定义在· ObjectBase.h·中有简单的注释，可以帮助理解它所代表的意思。

Unreal Build Tool (UBT) and Unreal Header Tool (UHT)来生产反射所需要的代码。存储在·per-module .generated.inl·和·per-header .generated.h·

可以通过类型，也可以通过对象，拿到对应的反射类。
```C++
UTypeName::StaticClass() 
Instance->GetClass() 
//遍历反射对象的属性
for (TFieldIterator<UProperty> PropIt(GetClass()); PropIt; ++PropIt)
{
	UProperty* Property = *PropIt;
	// Do something with the property
}
```

```C++
//////////////////////////////////////////////////////////////////////////
// Base class for mobile units (soldiers)

#include "StrategyTypes.h"
#include "StrategyChar.generated.h"

UCLASS(Abstract)

class AStrategyChar : public ACharacter, public IStrategyTeamInterface
{

	GENERATED_UCLASS_BODY()

	/** How many resources this pawn is worth when it dies. */
	UPROPERTY(EditAnywhere, Category=Pawn)
	int32 ResourcesToGather;

	/** set attachment for weapon slot */
	UFUNCTION(BlueprintCallable, Category=Attachment)
	void SetWeaponAttachment(class UStrategyAttachment* Weapon);

	UFUNCTION(BlueprintCallable, Category=Attachment)
	bool IsWeaponAttached();

	protected:

	/** melee anim */
	UPROPERTY(EditDefaultsOnly, Category=Pawn)
	UAnimMontage* MeleeAnim;

	/** Armor attachment slot */
	UPROPERTY()
	UStrategyAttachment* ArmorSlot;

	/** team number */
	uint8 MyTeamNum;

	[more code omitted]
};
```


### Net Serializtion
检测变量改变分为值类型数据和指针类型数据。
- 值类型，可以直接比较`Recent[]`然后用NetSerialize
- 指针类型，用NetDeltaSerialize比较状态和序列化。base state，delta state（发给客户端），full state（保存）。

有两种类型的`delta序列化`：generic Repication和fast array replication

`DataBunch.h`定义了bunch。
`FReceivedPacketView`:Represents a view of a received packet



## 结论

对网络注重性能优化，大部分优化从堡垒之夜而来。适合开发堡垒之夜类型的游戏。

## 参考
