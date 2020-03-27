


## 调试技巧
` 可以打开控制台
蓝图也可以断点

在控制台可以设置网络延时和丢包。

simulate，收到位置后，通过线性或者指数的插值方式，平滑移动到目标点。`client prediction data`。


`PerformMovement`函数负责角色的物理移动，会根据不同的移动方式调用对应的physics函数来计算角色的速度和加速度。

组件会在每帧调用`TickComponent`方法，计算移动。

```c++
        if (CharacterOwner->GetLocalRole() == ROLE_Authority)
        {
            PerformMovement(DeltaTime);
        }
        else if (bIsClient)
        {
            ReplicateMoveToServer(DeltaTime, Acceleration);
        }
```
每帧的移动数据存储在`SavedMoves`列表中，最后合并为一个move同步给服务器。

ReplicateMoveToServer
ServerMoveNoBase

### 跑帧

客户端会每帧更新组件。服务器不会。服务器统一在`UNetDriver::ServerReplicateActors`的更新中发送。

##### Calculating
两种模式。用客户端的时间戳，

默认配置，每秒60帧同步移动数据。`MAXPOSITIONERRORSQUARED`来配置服务器能容忍的最大位置误差

>If the character is performing root motion from an animation, MoveAutonomous also ticks the character's animation pose using the supplied delta time. Any animation events will trigger appropriately. Otherwise, animation ticks normally.


`root motion` 会有不一样的操作。


## 5. 其它

### online subsystem

OnlineSubsystem是一个多网络平台对接系统，在ue4中以插件的形式存在。可以通过他对接Steam，GooglePlay，Amazon，Xbox，Tencent等，进而使用对应平台的各项自定义功能，如语音聊天，在线统计，游戏大厅与游戏匹配等。

### 性能监控
ue4提供独立的程序 `/Engine/Binaries/DotNET/NetworkProfiler.exe`对网络性能进行了一定的监控。

### 地图切换


地图切换分为无缝和非无缝切换。应尽量使用无缝切换，不仅有更好的体验，还可以保持actor的状态。

在无缝切换时，需要准备一张小的过度地图，玩家从老地图进入过度地图再进入目标地图，避免老地图和目标地图同时存在而占用大量内存


