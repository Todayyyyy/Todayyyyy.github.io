---
title: WwiseGameDocument
---

# Wwise与Unity集成

***TIPS:本文中出现的任何词汇均可以使用crtl+f在词汇表中进行查询。***

文档归档逻辑：

依据不同的工作内容进行归类，例如：在设计初期使用到的是designer layout，然后是mixer layout进行混音，然后再soundbank layout集成，后续需要测试bug以及优化性能则需要在profiler layout等。

**声音管线**

![1](image/Typora/1.png)

![2](image/Typora/2.png)

[TOC]



## 1.Designer Layout

在该流程下，我们需要完成SFX、InteractiveMusic在Audio界面的design设计，其中包括GameSyncs(Swtiches/States/Parameters/Triggers)、ShareSets(Effects/Attenuations/...)，并且在Event界面进行事件编辑。

下面的逻辑是：通过ProjectExplorer界面对各个板块进行详细介绍。

### 1.1 Audio

#### 1.Type

audio界面下，主要的几种文件夹类型如下。

##### 	1.Virtual Folder

虚拟的文件夹类型，只是助于分类文件，实际则没有任何作用。

##### 	2.Random Container

该文件夹内的SFX，会在每一次播放的时候，随机挑选。

##### 	3.Swtich Contianer

切换容器。其中的对象/容器可以映射到一系列 Switch（切换开关），它们对应游戏中特定条件的不同选项。

Contents Editor:

1. 在 Play 列中选择 **1st only** 选项，那么将仅当 Switch/State 有变化时才播放该对象。如果您不勾选 1st only 选项，那么游戏每次触发切换容器时，无论 Switch/State 是否有变化，都将播放该对象。
2. 在 Across Switches 列中选择 **Continue to play** 选项，那么如果音频对象同时指派至源和目标 Switches/States ，则在 Switch/State 变化期间对象将继续播放。如果您不勾选 Continue to play 选项，那么对象将停止并从开头重新播放。
3. 在 Switch/State 发生变化时让新对象淡入/现对象淡出，则请在 Fade-In/Fade-out 文本框中输入时间长度。

##### 	4.Blend Container

混合容器。其中的对象/容器将**同时**播放。

**Blend Track**

Blend Track 用于将混合容器内的对象及其属性值进行编组。

同一个 track内的对象属性可以通过 RTPC映射到游戏参数，即由参数值来控制对象属性。

Cross Fade:两个对象间也可以基于游戏参数值来进行交叉淡变。

fademode:右键crossfade，选择：

- **None（无）：**当参数值进入方块重叠的部分，RTPC 的属性立即从其最大值过渡到最小值（或由小到大）。

- **Automatic（自动）（默认）：**淡变范围等于相邻方块的重叠部分宽度。如果没有相邻方块，则没有淡变。

- **Manual（手动）：**您可以移动淡入结束或淡出起始的点。淡入起始或淡出结束的点将总是在重叠块的外部下角处开始。

  > TIPS:
  >
  > - 音频文件长度
  >
  >   - 音频文件长度必须大于或等于 0.2 秒。
  >   - 交叉淡变时间最短为 0.1 秒。
  >
  > - 交叉淡变时间与音频文件长度的关系
  >
  >   - 从声音 A 交叉淡变至声音 B 时，声音引擎所允许的淡变时间最长为音频文件 A 长度的一半。如果淡变时间大于允许的最大值，那么将被自动调整为淡出文件长度的一半。
  >
  >     | *备注*                                                       |
  >     | :----------------------------------------------------------- |
  >     | 如果交叉淡变对于容器内的若干个音频文件过长，则 Wwise 将不会进行限制或提示点。如果需要对交叉淡变时间进行调整，声音引擎会在运行时进行处理。 |
  >
  > - 暂停和交叉淡变
  >
  >   - 如果暂停播放使用交叉淡变过渡的声音，同时向暂停操作应用淡出，交叉淡变的时机可能会不准确。
  >
  > - 音高和交叉淡变
  >
  >   - 如果您使用 RTPC 设置了容器的音高值，或在播放容器时触发了 Set Pitch 事件动作，则对声音应用交叉淡变时可能会产生意外结果。
  >
  > - 源插件与交叉淡变
  >
  >   - 对源插件应用交叉淡变时，如果无法确定源的结束时间，那么淡变可能会被忽略。例如，当正弦波生成源的时长基于一个 RTPC 时，就会发生这种情况。在这些情况下，交叉淡变被忽略，并且过渡将会在没有交叉淡变的情况下完成。
  >
  > - 切换容器与交叉淡变
  >
  >   - 当 **Switch Container** 作为 **Sequence Container** 的子容器时，会根据指派给切换开关的对象数量，区别应用交叉淡变过渡。
  >
  > - 两个声部
  >
  >   - 交叉淡变期间，声音引擎会使用两个不同的声部。
  >
  > - 虚声部和交叉淡变
  >
  >   - 根据定义，在低于音量阈值或超过播放数量限制时，**Play from Beginning** 和 **Resume** 虚声部行为会影响声音持续时间，这不在交叉淡变时间机制的考虑范围之内。
  >   - 当声部的音量低于阈值时，声部会变为虚声部。对于任何声音，会使用其所有音频通道的实际有效音量与阈值相比较。这个有效音量包括 Actor-Mixer Hierarchy、淡变过渡、互动音乐过渡、RTPC、状态、定位和衰减等所有的音量影响。
  >   - 当计算声音的有效音量时，淡变过渡的音量影响也计算在内。因此，在 **Random**、**Sequence** 或 **Blend Container** 内的交叉淡变过渡期间，淡变声音在某些时候将有可能低于音量阈值。如果它们从虚声部恢复时的行为是**Play from Beginning**或**Resume**，则其真实时长将长于容器淡变的逻辑所预期的时长。这将导致不可预测的行为。更糟糕的是，当声音淡出至低于阈值时，它会停止发声，但仍进行“虚拟”播放，因此永远不会结束。因此，容器将可能停止播放后续声音。
  >
  >   总而言之，应该避免对使用交叉淡变过渡的容器设置这两种类型的虚声部。如果希望对这些容器使用虚声部，则应选择 **Play from Elapsed Time** 行为。

##### 	5.Physical Folders

实文件夹。一种高层元素，可以包含一组工程中其它的 Physical Folder 或 Work Unit。实文件夹不能作为容器或声音的子对象。

##### 	6.Music Segment

由若干 Music Track组成，每个 Music Track 都包含 Music Clip。这些片段可以在 Music Segment 内进行排列对齐。

music track:music segment中的每个轨道。

music clip:音乐片段，代表的是每一个音源文件。

- Bars/Beats（栏/节奏）
- Cues（提示点）
- Clips/Loops（片段/循环）

​	**1.子音轨**

​	将子音轨添加至音轨：

​		添加子音轨这一功能仅适用于被定义为 Random Step、Sequence Step 或 Switch 的音轨。

​		强制播放子音轨：使用 Force Usage 功能来取消音轨的随机或序列行为。

​	将子音轨关联至Switch/State：

​		在property editor中的General Settings中的Switch选项中选择即可。  

​	**2.使用片段**

- 循环片段

1. 点击片段裁剪点，并拖动以延展片段。
2. 延展片段，直至片段边缘标志显示在时间线上。

- 移动片段

​	使用 Snap 功能，可以准确沿时间线间隔对齐片段。

- 分割片段

​	在音轨中，右键点击要分割的片段，然后从快捷菜单中选择 **Split on Play Cursor**。

- 重叠片段

​	重叠片段时，更改片段的堆叠方式十分实用，特别是当要查看已重叠片段的同步点时。可以通过快捷菜单中的 Bring to Front 和 Send to Back 命令实现。

​	**3.Cue**

​		1.Entry/Exit Cue

​	音乐片段的入口和出口提示点。Entry前为pre-entry，Exit后为post-exit。

​	右键选择的音乐片段，选择Move Entry/Exit Cues to Selection：将入口和出口提示点与所选片段对齐。

​		2.Custom Cue

​	用于状态更改、过渡或插播乐句插入。右键，从快捷菜单中选择Add Custon Cue。

​	右键，Show in Multi Editor，可以调整精确位置和颜色等。

​		3.链接Event

​	在音乐片段的任意位置添加Action Event(Dialogue Event不行)，在播放 Music Segment 时，一旦 Play Cursor	（播放光标）到达相应 Event 所在位置，便会自动触发该 Event。

​	右键快捷菜单Add Music Event Cue/直接从Event菜单拖拽/从已有的复制。

##### 	7.Music Playlist Container

Music Playlist中的Group/Segment

- Sequence continuous:序列连续模式。每次播放组时依次按顺序播放该组中的所有音乐对象。
- Sequence step:序列步进模式。每次播放组时，仅播放组中的一个音乐对象。下次播放该组时，将播放组中的下一个音乐对象。
- Random continuous:随机连续模式。每次播放组时随机逐一播放该组中的所有音乐对象。
- Random step:随机步进模式。每次播放组时，仅随机播放所选组中的一个音乐对象。

Weight:权重，在random container中播放它的概率。

##### 	8.Music Switch Container

​	1.将GameSync与Music Switch关联

通过>>按钮来添加Switch Group或者State Group

​	2.将GameSync与Music Playlist/Segment关联

通过Add Path，将选中的State/Switch添加到Path，在相应的Path中，点击object旁的“...”来选择你要链接的对象。

​	3.将RTPC与其关联

将RTPC与Switch Group关联即可。

| 界面元素                                               | 描述                                                         |
| :----------------------------------------------------- | :----------------------------------------------------------- |
| ![add generic path](image/Typora/add generic path.png) | 创建完整的通用路径（*.*），来适用任意状态或切换开关。此选项仅在通用路径尚不存在时才可用。 |
| ![add path](image/Typora/add path.png)                 | 适用所选的状态/切换开关组或状态/切换开关值创建路径。         |
| ![update path](image/Typora/update path.png)           | 更新路径。针对所选音频对象更新路径。此选项会将所选对象的路径替换为当前选择的 State 或 Switch。此选项仅在所选路径尚未出现在列表中，并且在列表中选择了一个对象时才可用。 |
| ![remove path](image/Typora/remove path.png)           | 从路径列表中删除所选路径。                                   |
| ![unselectAll](image/Typora/unselectAll.png)           | 取消选中 States/Switches 窗格中的所有 State 和 Switch。      |
|                                                        |                                                              |
| （States/Switches 窗格）                               | 显示已分配给当前所选 Music Switch Container 的 State Group 和 Switch Group。您可以选择不同的状态或切换开关，来创建不同路径组合。 |
| ![choose](image/Typora/choose.png)                     | 打开可让用户从备选 Switch Group 或 State Group 对象列表中进行选择的菜单。用户还可以通过选择 "New.. " ，从该菜单创建新的状态组或开关组。 |
|                                                        |                                                              |
| Mode                                                   | 模式。在运行时触发的状态或切换开关有不只一条预定义路径可以匹配的情况下，指定声音引擎将选择哪条路径。**Best Match** – 最佳匹配。选择与运行时触发的状态或切换开关最匹配的路径。如果不能完全匹配，则将会选择具有最少通配符 (*) 的路径。**Weighted** – 加权。基于各路径的权重值，随机选择匹配路径之一。因为在路径中可以使用通配符（*），所以可能同时有几条预定义的路径都能够匹配在运行时触发的状态或切换开关。Default value: Best Match |
| Path Filter                                            | 路径筛选器。显示可用于筛选路径的列表。可以使用以下筛选器：**All** -- 所有。显示所有已创建的路径。**Current Selection** -- 当前选中项。仅显示包含所选切换开关或状态的路径。 |
| Object Filter                                          | 路径筛选器。显示可用于筛选路径的列表。可以使用以下筛选器：**All** -- 所有。显示所有已创建的路径。**Assigned object** -- 指派对象。仅显示与对象关联的路径。**Missing** -- 缺失。仅显示与已删除的对象相关联的路径。**None** -- 无。仅显示未与对象进行关联的路径。 |
|                                                        |                                                              |
| ![configureColumns](image/Typora/configureColumns.png) | 点击列标题区 **Configure Columns...**（配置列...）快捷方式（右键点击）选项。p此时将会打开 [“Configure Columns 对话框”一节](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=configure_columns)。指定要显示的列及其顺序。 |
| Path                                                   | 路径。包含 Music Switch Container 中可能存在的 Switch 或 State 的多种组合之一的特定路径。这些路径与音乐对象相关联。路径可能包含通配符，这将匹配任何相应的 Switch 或 State。通配符由星号表示。 |
| Weight                                                 | 权重。当存在其它匹配路径的情况下，该路径被选择的可能性。在一条匹配路径的权重为 100 时，其它权重低于 100 的匹配路径会被自动弃用。在一条配路径的权重为 0 时，除非其它所有匹配路径的权重也都为 0，否则该路径会被弃用。此选项仅在 Dialogue Event 处于“Weighted”模式时才适用。单位：% Default value: 50 Range: 0 to 100 |
| Object                                                 | 指派给路径的对象。可以将指派给路径的对象与其它平台断开链接，以便为相同路径根据不同平台来指派不同的对象 |
| ![..](image/Typora/....png)(浏览)                      | 浏览。打开Project Explorer - Browser（工程浏览器 —— 浏览器），可以在其中选择将指派给此路径的对象。您还可以直接从 Project Explorer 中拖拽对象。 |



##### 9.Sound Type

​	1.SFX

音效声。即音效对象。

​	2.Voice

语音。即语音对象。

#### 2.General Settings

在generalSetting界面下，有部分简单基础的功能，会根据当前所选中的SFX和Folder类型有些许改变。

##### 1.Voice

​	1.音量

​	2.音高

​	值1200=2倍速度，且相当于一个八度。

​	3.低通/高通滤波器

​	4.补偿增益

​	应用于所有其他音量调整之后，并且在Actor-Mixer Hierarchy中具有叠加特性。

​	可用于最终的响度归一化。

##### 2.Output Bus

​	1.Volume

​	输出到音频总线时，信号的衰减或增幅。

> ​	**TIPS:**如果使用了User-Defined Auxiliary Sends（辅助总线）的湿声（reverb）的话，Output Bus Volum将于干声电平链接，Auxiliary Bus Volume将与湿声电平链接。在游戏中可以用RTPC分别控制两者来控制游戏平衡。
>

​	2.高通/低通滤波器

​	输出到Bus时用滤波器。

##### 3.Game-Defined Auxilary Sends

​	Use Or Not:

​	决定是否使用该游戏对象的游戏定义的辅助发送，其中包括Auxiliary Bus以及Volume。若启用，则对象会受到以下函数值的影响：

1.ak.soundengine.setGameObjectAuxSendValues 设置 Auxiliary Bus 以输出给定游戏对象（湿声）。

> 其Arguments：
>
> | Name                                  | Type                                                         | 说明                                                         |
> | :------------------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
> | **gameObject** *                      | integer                                                      | 关联游戏对象 ID。 游戏对象 ID，64 位无符号整数。范围: [0,18446744073709551615] |
> | **auxSendValues** *                   | array                                                        | 此数组中包含一系列 [AkAuxSendValue](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=SDK&id=struct_ak_aux_send_value.html) 结构。 |
> | auxSendValues **[...]**               | object                                                       | [AkAuxSendValue](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=SDK&id=struct_ak_aux_send_value.html) 结构。 |
> | auxSendValues[...].**listener** *     | integer                                                      | 与此 Aux Send 关联的听者的游戏对象 ID。 游戏对象 ID，64 位无符号整数。范围: [0,18446744073709551615] |
> | auxSendValues[...].**auxBus** *       | any of:                                                      | Auxiliary Bus 的 ID ([GUID](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=SDK&id=struct_g_u_i_d.html))、名称或 Short ID。 |
> | string                                | 对象的名称。                                                 |                                                              |
> | string                                | 以下形式的对象 GUID：{aabbcc00-1122-3344-5566-77889900aabb}。 |                                                              |
> | integer                               | Wwise 对象的 Short ID。 32 位无符号整数。范围: [0,4294967295] |                                                              |
> | auxSendValues[...].**controlValue** * | number                                                       | 范围为 [0.0f:1.0f] 的值，Auxiliary Bus 的发送电平。          |

2.ak.soundengine.setGameObjectOutputBusVolume 设置给定游戏对象的输出总线音量（直达声）。

> 其Arguments：
>
> | Name               | Type    | 说明                                                         |
> | :----------------- | :------ | :----------------------------------------------------------- |
> | **emitter** *      | integer | 关联“发声体”游戏对象 ID。 游戏对象 ID，64 位无符号整数。     |
> | **listener** *     | integer | 关联“听者”游戏对象 ID。 游戏对象 ID，64 位无符号整数。       |
> | **controlValue** * | number  | 倍数：0 表示静音，1 表示不变。也就是说，在值介于 0 ~ 1 之间时会将声音减小，在大于 1 时会将声音放大。 |

**TIPS：**若游戏已spatial audio并且将Room和Portal数据发送到Wwise，则此属性决定是否要将对象发送到游戏对象当前所在的Room定义的AuxiliaryBus。

##### 4.Early Reflections

早期反射辅助发送。用于处理模拟声波在几何表面发生的反射。

因为最初的几次反射会携带更多的信息，所以需要在后期的混响中单独处理，使用ReflectVST生成更多细节。

**TIPS：在Unity中通过SpatialAudio来为游戏对象指定AuxBus(创建对象)。**

为了计算反射，WwiseSpatialAudio库必须初始化，并且将上层几何构造发送到WwiseSpatialAudio。

##### 5.InitialDelay

初始延迟。即播放事件发生后，需要等待几秒才会开始播放。

##### 6.Loop

是否循环。

##### 7.Stream流播放/文件包

启用后，可直接从游戏媒体中流播放音频。

SoundBank不包含流媒体文件。

> **流播放/流管理器StreamManager**
>
> 流管理模块的公共接口在 [IAkStreamMgr.h](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=SDK&id=IAkStreamMgr.h) 中定义，供声音引擎和游戏共同使用。前者使用此接口来加载 SoundBank 和流播放音频文件，而后者可用它来加载图形、纹理、游戏关、保存的游戏等。
>
> **文件包**
>
> 文件包（*.PCK）是通过将 Wwise 工程中的所有 SoundBank 和流音频文件组合形成单一文件。您还可以将 SoundBank 和流文件分配给不同的文件包。
>
> 在生成 SoundBank 后，将“FilePackager”可执行文件作为生成 SoundBank 的后续步骤，可以让该 Wwise 工具使用 SoundBank 和流文件自动生成文件包。

**Non-cachable**

不可缓存。在流播放管理器中启用缓冲区时，禁用此文件的缓存。可以避免长循环或者不经常播放的文件消耗流播放缓存区。

**Zero Latency**

零延迟。创建包含音频文件开头部分的小型音频缓冲区，其中包括预取文件的其余部分所需要的延迟时间，以达到无延迟的流播放整个声音。

Delay和Zero Latency设置按声轨进行。若各个声轨有多个音乐源，则各个源开头部分需要加载到内存中。

**Prefetch length**

加载到内存中声音开头部分，单位：ms。

##### 8.Scope

- **Global** 全局 —— 将游戏中使用的所有该容器作为一个对象进行处理，因此可以针对所有游戏对象避免重复声音或语音对象。
- **Game object** 游戏对象 —— 将该容器的各个实例作为单独实体进行处理。即不会在所有游戏对象间共用声音内容。

> TIPS:
>
> Scope 选项不适用于 Continuous 播放模式下的序列容器，因为每次事件触发该容器时都将播放全部播放列表。

##### 9.PlayType

1.Random

- standard。播放完后不会从列表中拿出。

- shuffle洗牌模式。播放对象后将他们从播放列表中移除。

- avoid repeating。standard模式下，最后播放的x个对象将从列表中弃用；shuffle模式下，在列表中排除最近播放的X个对象。

2.Sequence

- Restart。从头开始播放。
- In reverse。逆向开始播放。

##### 10.PlayMode

1.Step

每次播放，仅播放容器内的一个对象。

2.Continuous

每次播放会完整播放容器内所有对象。

​	1.Always reset playlist

​	确定在播放期间停止播放之后，时继续播放还是从头开始。勾选则是从头播放。

​	2.Loop

​	不再赘述

​	3.Transitions

​	过渡。启用后，在对象与对象之间创建和定义过渡。

​		①Type过渡类型。

- **Xfade (amp)** 淡变（恒定振幅），在两个对象之间保持恒定振幅进行交叉淡变。
- **Xfade (power)** 淡变（恒定功率），在两个对象之间保持恒定功率进行交叉淡变。
- **Delay** 延迟，在两个对象之间添加无声段落。
- **Sample Accurate** 精确到采样点，对象之间进行零延迟的无缝过渡。请注意，Opus（在从磁盘进行流播放时）和 XMA 音频格式不支持精确到采样点的过渡。
- **Trigger rate** 触发速率，使用特定速率来触发容器内的对象。该选项对于模拟快速枪声十分实用。除此之外，也可使用 MIDI 来以更高的精度发送每个子弹声音。有关详细信息，请参阅 SDK 文档中的[模拟快速射击](https://www.audiokinetic.com/library/edge/?source=SDK&id=goingfurther_rapidgunfire.html)章节。

​		②Duration持续时间。

​		输入所需的交叉淡变、延迟或触发速率的时长。

##### 11.Time Settings

1. 在 Tempo（速度）文本框中，键入原始音乐的速度（单位为每分钟节拍数，即 bpm）。
2. 在 Time Signature（拍号）文本框中，键入原始音乐每小节的拍数和长度。
3. 在 Frequency（网格频率）下拉菜单中，选择用多长的网格来划分 Music Segment，例如 4 bars（每 4 小节）、beat（每拍）、whole note（每个全音符）等。
4. Offset，若要为频率值设定偏置，请从列表中选择预定义选项，或者通过选择 Custom（自定义）并在对应 Offset（偏置）文本框中键入数值（单位为毫秒）来创建自定义偏置。

##### 12.Play Option

Continue to play on Switch change

如果希望 Switch/State 改变时，源和目标共用的音乐对象继续播放，请选择 **Continue to play on Switch change**（在 Switch 更改时继续播放）选项。

如果希望 Switch/State 改变时，源和目标共用的音乐对象在下一个同步点停止并从头播放，则不应勾选 **Continue to play on Switch change** 选项。

#### 3.Conversion

##### 1.Conversion Settings Editor

###### 1.转码设置 

| Notes                          | 备注。有关转码的详细信息。                                   |
| ------------------------------ | ------------------------------------------------------------ |
| Platform                       | 工程中的平台列表。转码设置是为工程中的每个平台设置的。       |
| Channels                       | 声道数。您希望包含在转码文件中的音频声道数。以下选项可用：**As Input** – 同输入。将音频声道数保持与原始媒体文件相同。**Mono** – 单声道。所有声道将混合为一个声道，但 LFE 除外，该声道将被弃用。**Mono drop** – 单声道弃用。除了第一个声道外，其它所有声道都将被弃用。**Stereo** – 立体声。所有声道都将混合到左前和右前声道，但 LFE 除外，该声道将被弃用。**Stereo drop** – 立体声弃用。除了左声道和右声道外，其它所有声道都被弃用。有关 Wwise 如何处理音频声道的下混转换的详细信息，请参阅 [创建音频转码设置 ShareSet](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=creating_audio_conversion_settings_sharesets) 。Default value: As Input |
| L-R Mix                        | 左右混音。左扬声器和右扬声器的信号功率电平，其中 <C> 代表两个扬声器上的功率均等指派。L-R Mix 控制仅在将单声道声音转换为立体声（反之亦然）时应用。对于所有其它转换，混音都由 Wwise 完成。有关 Wwise 如何处理 L-R Mix 的详细信息，请参阅[“关于音频声道”一节](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=creating_audio_conversion_settings_sharesets#about_audio_channels)。 |
| Sample Rate                    | 数字音频信号每秒采样的次数，单位为赫兹 (Hz)。您可以选择以下选项中的任何一个选项：**As Input** - 使用与原始音频文件相同的采样率对文件做转码。如果特定平台或音频格式不支持该采样率，则会改用最接近的可用采样率。**Auto (Low/Medium/High)** - 在对文件执行 FFT 分析后，使用 Wwise 选定的采样率对文件做转码。低品质、中品质和高品质设置之间的区别是截止频率阈值，该阈值将用于确定转码结果文件的最佳采样率。您可以通过在 Project Setting（工程设置）对话框中定义频率阈值，来微调各个品质设置的品质高低。**300 to 48,000** – 300 至 48,000。使用特定采样率对文件做转码。各种平台和格式对应的采样率范围可能有所不同。Default value: 0 |
| Min Sample Rate                | 最小采样率。仅在 **Sample Rate** 设置为 **As Input** 或 **Auto** 时才会采用。定义转码文件的最小采样率。 |
| Max Sample Rate                | 最小采样率。仅在 **Sample Rate** 设置为 **As Input** 或 **Auto** 时才会采用。定义转码文件的最大采样率。 |
| Format                         | 格式。要针对各个平台将音频源转码成的音频格式。其中包括：***[AAC](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_aac)/*** ***[ADPCM](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_adpcm)/*** ***[WEM Opus](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_opus)/***** ***[PCM](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_pcm)****[Vorbis](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_vorbis)/***[并非所有音频格式都支持多声道文件](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=creating_audio_conversion_settings_sharesets#about_audio_channels)。例如，如果原始文件是多声道文件，并且要转换为 AAC，则应从 Channel 列表中选择以下选项之一：Mono、Mono drop、Stereo 或 Stereo drop。如果选择 **As Input** 选项，Wwise 将强制下混为立体声或单声道。 |
| Sample rate conversion quality | 品质。压缩文件的品质设置。某些音频格式（例如 Vorbis）允许进行品质设置。更高的品质设置意味着：更多的磁盘空间需求。更好的音频品质。Default value: Normal |
| Adv.                           | 高级。针对某些音频格式的特殊设置。点击 **Edit**（编辑）以对这些格式进行设置。 |

###### 2.Options

| 采样率转码品质         | 采样率转换品质。确定用于转换采样率的方法。您可以选择以下方法之一；**Normal**: 正常。转码品质较好，比“High”（高）选项快 3 到 6 倍。**High** —: 最佳转码品质。如果原始音频内容包含高频，并且您的转码采样率低于 24 kHz，则建议您使用“High”选项。 |
| ---------------------- | ------------------------------------------------------------ |
| Insert filename marker | 插入文件名标记。确定是否在文件开头添加带有文件名的标记。对于将特定动作绑定到声音引擎中的特定文件，例如对口型这种情况，可能会非常有用。标记仅包含文件名，而没有文件的路径和扩展名。Default value: false |
| Remove DC offset       | 移除直流偏置。移除已转码音频文件的 *[DC Offset](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_dc_offset)* 。直流偏置可能会导致 clicking 噪声、音量损失和失真。下列情况下，最好移除直流偏置：源文件录制质量较差，波形相对于 0 点不对称；效果器导致声音存在累积偏置（这种情况很难预测，但有可能是造成问题的原因。）不过对于以下情况，最好不要移除直流偏置：位于精确到采样点的容器中的声音。归一化为 0 dB 的声音。精确到采样点的循环播放声音。事实上，我们建议不要为循环播放的声音消除直流偏置。消除机制采用高通滤波器，因此不保证将以相同方法修改循环的第一个采样和最后一个采样，因为并不知道这两个样本将连续播放。这可能产生信号中断，导致听到爆音。Default value: false |
| Apply dither           | 应用抖动。对音频源进行比特率转换时应用抖动。抖动是在量化之前添加到信号中的噪声，目的是减少量化过程导致的失真和噪声调制。抖动仅在采样位深更改（例如从 16 位变为 8 位）时才会应用。如果不勾选此选项，则将不会应用抖动。默认情况下，此选项已启用。Default value: false |
| Allow channel upmix    | 允许声道上混。如果选中此选项并且 Channel 列表设置为 Stereo 或 Stereo drop，则单声道源文件将在转码期间上混为该立体声配置。如果未选中此选项，则无论 Channel 如何设置，单声道源文件转码时都不会进行上混。Default value: true |

###### 3.Audio Sources

| Show              | 显示。确定哪些信息显示在 Audio Source 列中。您可以选择以下选项之一：**Name**: 名称。仅显示音频源名称。**Path**: 路径。显示完整路径，包括音频源名称。 |
| ----------------- | ------------------------------------------------------------ |
| Convert...        | 为使用此转码设置的音频源打开 Audio File Conversion 对话框。还可以使用音频源列表中的快捷菜单，来对音频源做转码。 |
| Copy to clipboard | 将所选平台的转码结果复制到 Windows 剪贴板，以便将它们粘贴到其它应用程序。 |
| Audio Source      | 音频源。音频源的名称。在 Show 选项设置为 Path（路径）时，完整路径将会与音频源的名称一起显示。 |
| Language          | 音频源的语言。                                               |
| Orig. Chan.       | 源声道。原始音频源的声道配置。                               |
| Conv. Chan.       | 转码后声道。转码后音频源的声道配置。                         |
| Original SR       | 原始音频源的采样率。                                         |
| Converted SR      | 转码后音频源的实际采样率。                                   |
| Original Size     | 原始音频源大小。                                             |
| Converted Size    | 转码后的源大小。                                             |
| Size Ratio        | 大小比率。原始音频源大小和转码源大小的比率。                 |
| Duration          | 时长。转码后音频源的长度（单位为秒）。                       |
| Bandwidth         | 带宽。在给定时间内可能传输的最大数据量。单位：KB/s（每秒千字节数）如果您将鼠标指针放在带宽列中的某个值上，则将会出现以千位/秒（kbps）显示的带宽值提示点。 |

##### 2.Loudness Normalization（没懂）

Enable   Loudness Normalization

启用响度归一化。

#### 4.Postioning

##### 1.Center%（中置百分比）

通过中置扬声器的音量百分比。

​	对于Direct Assignment模式，仅当单声道对象输出到带中置声道的<u>总线</u>时才适用。

​	对于Balance-Fade模式，仅适用于带中置声道的输出（例如单声道、3.0、5.1和7.1）。

​	对于3D化模式，仅适用于带中置声道的输出；并且只有开启此值，信号才会发送中置声道。

##### 2.Speaker Panning

​	Direct Assignment。FL对应FL扬声器，FR对应FR扬声器。

​	Balance-Fade。允许调节2.0-7.1Audio Bus中各个声道的音量。离远点较近的声道音量更大，反之更小。

​	Steering。允许在输出总线声道中重新分配不同声道的声音，配比按照panner中计算。

##### 3.Listener Relative Routing

听者相对通路。启用后，会针对该对象计算发声体和听者的关系。

确保信号链中至少有一个对象启用了Listener Relative Routing，否则可能导致混音图完全重复。

​	1.3D spatialization

​		Position模式。将使用环绕声配置中特定扬声器进行播放，来表现声源移动。

​		Position+Orientation模式。除此之外，还会随着发声体和听着的相对朝向进行旋转。仅当音频为多声道时才起作用。

​		None。依据speaker panning摆位。

​	2.Attenuation

​		启用衰减。可以应用衰减曲线ShareSet，并添加RTPC控制。

​		①Height Spread。高度散布。

​		启用高度散布。若在2D 配置下实施声像摆位，则 Height Spread 根据高度角自动计算最小散布值: 若在 7.1.4之类的 3D 配(半球空间)下实施声像摆位，则直接根据负高度角计算最小散布值。藉此，可帮助实现 3D 声音从听者上方或下方掠过时声像摆位的平滑过渡。

​		在默认情况下，始终启用 Height Spread，即便声音没有衰减。只能通过取消选中Height Spread 来禁用该属性。若想在声音没有意减的情况下禁用 Height Spread,须专门为此创建 Attenuation ShareSet，并设置恒定不变的 Output Bus Volume。

​		②Cone Use。声锥衰减。创建不同角度上的锥形来控制声音衰减。其中，Cone filter可以通过RTPC来控制。

##### 4.3D Postion（3D定位）

​	①Emitter（发声体）：由实时游戏位置数据来定义“发声体”游戏对象的空间定位。在选择 Emitter 时，Wwise 中所定义的传播属性将直接关联至游戏中听者和发声体的位置或朝向。

​			**Hold Emitter Position and Orientation**

​			储存声音开始播放时游戏对象的瞬时位置和朝向，以及声音播放期间相对该基准位的偏移。

​			**Diffraction and Transmission**

> 启用衍射。针对声音在 Spatial Audio 中启用衍射和透射处理。
>
> 衍射用于模拟声波在障碍物周围发生弯曲的声学现象，透射则用于模拟声波在虚拟环境中穿透障碍物的传播情形。障碍物由通过 API 从游戏传到 SpatialAudio 的 Room、Portal 或几何构造定义。声波在障碍物周围的弯曲取决于衍射设置，声波穿透障碍物的传播情形则取决于透射损失。两者都会影响声音的最终音量和滤波效果。
>
> 为了对衍射和透射进行模拟，游戏必须定义上层几何构造或 Room 和 Portal.
> 并将其发送到 Wwise Spatial Audio。
>
> 在针对声音启用衍射和透射时，Wwise Spatial Audio 中会发生以下情况:
>
> - 计算发声体和听者之间的声音路径。该路径由一条直达/透射路径和零条或多条衍射路径构成。
> - 针对直达/透射路径计算透射损失系数(0%-100%)。透射损失由声音所穿透的 Room 和几何构造决定。
> - 按照 Wwise 工程设置中定义的 ocdlusion 曲线将透射损失系数 (%) 换算为音量/低通/高通滤波器值。声音会相应地被衰减和滤波。
> - 根据衍射路径中的角度之和计算衍射系数(0%-100%)。直线路径视为0%衍射，弯曲角度达到或超过 180 度的路径视为 100% 衍射。
> - 按照 Wwise 工程设置中定义的 obstruction 曲线将行射系数 () 换算为音量/低通/高通滤波器值。声音会相应地被衰减和滤波。
> - 对于衍射路径，通过计算声音相对于听者的视位置创建虚声源位置让听者感觉声音就像围绕角落或通过 Portal 传播一样。
>
> 备注:3D Game Object Viewer 允许用户查看 Wwise Spatial Audio 内发生了什么包括上层几何构造、Portal 以及声音的衍射路径和生成的虚声源位置。
>
> 备注:为了使衍射和透射生效，Wwise Spatial Audio 库必须初始化，而且游戏必须将上层几何构造或 Room 和 Portal 发送到 Wwise Spatial Audio。

​	②Emitter+Automation（发声体自动化）：通过在 Wwise 中创建相对于关联“发声体”游戏对象位置的路径来定义对象的空间定位。

这种情况下，需要使用游戏对象来定义能听到该声音的区域。

​	③Listener+Automation（听者自动化）：通过在 Wwise 中创建相对于关联“听者”游戏对象位置的路径来定义对象的空间定位。

​			**Hold Listener Orientation**

> 跟踪听者方向。确定动画路径的位置是否锁定到听者的朝向.
>
> 若未选中此选项，则路径将随听者移动。这意味着无论听者的朝向是什么总是能够通过相同的扬声器	听到声音。若选中，则听者将独立于路径移动。这意味着当听者转身时将通过不同的扬声器听到声音。
>
> 比如可以在听者周围使用自动化路径，在游戏中创建非固定位置的鸟叫声有一个单点路径位于右前象限中。当听者在游戏中转身时，将发生以下情况:
>
> Hold Listener Orientation (OFF): 始终通过右前扬声器播放鸟叫声.
>
> Hold Listener Orientation (ON): 通过不同的扬声器播放鸟叫声.
>
> 此选项可用于创建非固定位置的环境声此选项只能在游戏中测试，因为 Wwise 设计工具中还没有听者的概念.

#### 5.Transition

##### Source/Desitination

- **Music object：**音乐对象，可以是 Music Segment（音乐段落）、Music Playlist Container，也可以是 Music Switch Container。
- **Virtual Folder：**虚拟文件夹，Music Switch Container 内的音乐对象可组织到虚拟文件夹中。将 Virtual Folder 选为音乐过渡的 Source 或 Destination 时，过渡规则将对该文件夹内的所有音乐对象生效。
- **Any：**任意对象，表示容器中的任何音乐对象都可用作 Source 或 Destination。
- **Nothing：**无对象，表示 Source 或 Destination 为空，即不播放任何音乐对象。

##### Source

1.若source为Switch Container,可以调整Exit source at:

- ​	**Immediate（立刻）：**立即停止播放 Source。
- ​	**Next Grid（下一网格线）：**在下一个网格线停止播放 Source。通过 Grid（网格线），可以按任意间隔长度来划分音乐对象。
- ​	**Next Bar（下一小节）：**在下一小节停止播放 Source。
- ​	**Next Beat（下一拍）：**在下一拍停止播放 Source。
- ​	**Next Cue（下一提示点）**：在下一个提示点停止播放 Source，可以是 Custom Cue，也可以是 Exit Cue（出口提示点）。
- ​	**Next Custom Cue（下一自定义提示点）**：在下一个 Custom Cue 停止播放 Source。如果当前 Music Segment（音乐段落）不包含 Custom Cue，则 Wwise将继续播放下一段落，直到遇到 Custom Cue 为止。
- ​	**Exit Cue（出口提示点）：**在出口提示点处停止播放 Source。

> tips:如果您选择 Next Cue 或 Next Custom Cue，就可以在 **Match** 文本框中输入提示点名称，精确筛选对该过渡规则有效的提示点。

2.Play post-exit：播放Exit cue后的内容。

3.Fade-out可对衰弱曲线进行调整。

##### Destination

1.

若destination为Music Playlist，可以在jump to中选择开始播放的时刻：

- Start of Playlist :播放列表的开头。跳到第一个播放列表项
- Specific Playlist ltem :特定播放列表项。跳到指定的播放列表项。若要指定所要跳到的播放列表项，请单击”浏览”按钮。此项仅在目标对象为 MusicPlaylist Container 时可用
- Last Played Segment:上次播放的段落。跳到上次播放的段落
- Next Segment:下一段落。跳到上次播放的段落之后的下一播放列表项

若destination为Music Switch Container,则在sync to中选择：

- **Entry Cue（入口提示点）：**将从入口提示点处开始播放 Destination。
- **Same Time as Playing Segment（与现有音乐时间相同）：**Destination 开始播放的位置将与 Source 段落切出的时间点相同。例如，如果 Source 段落已经播了 10 秒，则 Destination 也将从 10 秒处开始播放。
- **Random Cue（随机提示点）**：从随机选择的提示点处开始播放 Destination。勾选该选项即选择了所有提示点，包括 Entry Cue（入口提示点）和所有 Custom Cue。
- **Random Custom Cue（随机自定义提示点）**：将从随机选择的 Custom Cue 处开始播放 Destination。如果段落中没有自定义提示点，则使用入口提示点。
- **Last Exit Position（上次退出位置）**：根据上次 Destination 的段落播了多久施加相应的时间偏置，然后再开始播放 Destination。例如，若上次 Destination 的第三个段落播了 15 秒，则将从 15 秒处开始播放 Destination。

> tips:若使用了Random Cue/Random Custom Cue，则可通过 Custom Cue Filter（自定义提示点筛选器）进一步精确筛选有效的开始位置。
>
> - **Match（匹配名称）：**只有用了该名称的提示点才会被选为开始位置。
> - **Match source cue name（匹配源提示点名称）：**仅当提示点与 Source 段落所用提示点名称相同时，才会作为开始位置。

2.过渡时如果要播放 Destination 的 Pre-entry（前导段），请选择 **Play pre-entry（播放前导段）**。

##### Transition Segment

选择Trasition Segment片段之后 ，

- 如果要播放过渡段的 Pre-entry，请勾选 **Play pre-entry**。
- 如果播放过渡段时需要淡入，请勾选 **Fade-in**。
- 如果要播放过渡段的 Post-exit，请勾选 **Play transition post-exit**。
- 如果停止过渡段时需要淡出，请选择 **Fade-out**。

#### 6.Stinger

插播乐句。Stinger 是一种简短乐句，可以与当前播放的音乐<u>叠加并混合</u>播放。游戏会调用与 Stinger Music Segment相关联的 Trigger，从而播放 Stinger。

##### 1.添加stinger

将音乐对象加载，在其Stinger界面进行关联Trigger和Music segment

play at(播放位置)：

- **Immediate** —— 立即切换状态。如果为 Music Track（音乐轨）设置了 Look-ahead Time（预读时间），则须等到这段时间过后才能播放该 Stinger。
- **Next Grid** -- 下一网格线。在下一网格线处播放 Stinger。通过 Grid，可以按想要的频率将音乐对象进行虚拟划分。
- **Next Bar** -- 下一小节。在下一小节处播放 Stinger。
- **Next Beat** -- 下一拍。在下一拍播放 Stinger。
- **Next Cue** -- 下一提示点。在下一提示点处播放 Stinger。下一提示点可以是 Entry Cue（入口提示点）、Exit Cue（出口提示点）或者 Custom Cue（自定义提示点）。
- **Next Custom Cue** -- 下一自定义提示点。在下一个自定义提示点处播放 Stinger。
- **Entry Cue** -- 入口提示点。在入口提示点处播放 Stinger。
- **Exit Cue** -- 出口提示点。在出口提示点处播放插播乐句。

##### 2.stringer option

 **Don't play this Stinger again for** <u>x</u> **seconds**（在 x 秒内不再播放该插播乐句）字段中键入须经过几秒才能再次播放该 Stinger。

若当前段落剩余时间不够而需在下一段落中播放 Stinger，请选中 **Allow playing the Stinger in next segment**（允许在下一段落中播放插播乐句）选项。

##### 3.预演stinger

在 transport control（就是播放界面）

#### 7.Advanced Settings（动态混音）

##### 1.Playback Limit

Limit sound instances to: X  /   XX；

限制发声数。X为数量，XX为：

- Per game object:每个游戏对象。分别为此层级中的每个游戏对象 应用发声限制。
- Globally:全局。为此层级中所有游戏对象的发声总数 应用限制。

​	When limit is reached:

- kill vioce for lowest priority:停止播放具有最低优先级的实例。会执行几毫秒的淡出。
- use virtual voice settings for lowest priority:为优先级最低的声部应用其virtual voice behavior。

​	When priority is equal:当达到上限且优先级相等

- Discard oldest instance——丢弃最早播放的优先级最低的实例。
- Discard newest instance——丢弃最新播放的优先级最低的实例。

##### 2.Virtual voice

Virtual voicce behavior:

当对象音量低于音量阈值或者播放数量超出限制时，对象的行为 ：

- Continnue to play.即使无法听见也继续播放。
- Kill voice.停止播放对象，并且没有淡出处理。
- Send to virtual voice.将对象发送到虚声部列表。当某个对象被发送到虚声部列表时，该对象的某些参数将由声音引擎监视，但不会对音频或震动进行任何处理。
- Kill if finite else virtual.如果对象不是无限循环，则停止播放；如果对象无限循环，则将发送到虚声部列表。对于此选项，On return to physical voice行为将自动设置为Play from elpsed time,而不考虑其父级设置行为。

##### 3.Playback Priority

Priority:

优先级。优先级为1的对象具有最低的优先级，而优先级为100的对象具有最高的优先级。

Offset priority by X at max distance:

在最大距离时，对优先级做量为x的偏置。指定一个值，当对象达到Attenuation Editor中指定的最大距离时，可以用此值来升高或降低该对象的优先级。

##### 动态混音技术

在 Advanced Setting 选项卡中可应用下面的动态混音技术。

- **限制总线或主角混音结构上的播放次数，为重要声音留出空间**

  例如，当有大量动作、爆炸或玩家应关注的任何元素上时，应减少环境声音和拟音的数量。查找环境声和拟音与枪声和爆炸声并存的总线，对该总线设置限制，并降低前者的优先级。

- **对优先级使用距离偏置**

  例如，在环境声音中，结合基于距离的优先级使用播放数限制，以将焦点集中在更近的声音上。**Offset priority by** 选项指定最大距离处的优先级偏置值，此值插值于 0 和 **at max distance** 之间。

- **闪避不重要的声音的音量**

  有时候不可使用播放数限制。例如，如果您在游戏关卡一开始就启动无限循环的环境声音，则应该避免使用播放数限制功能来停止这些声音，因为通过这种方式停止后，它们就不会再重新播放了。在这种情况或者您认为合适的任何情况下，当游戏音频的其它更重要区域中有活动时，应使用需要的技术来闪避它们的音量。例如，在配音或格斗期间关闭环境声音。当炸弹在您附近爆炸时，您无需听到电灯的嗞嗞声。此过程可视为元数据*[侧链](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=Help&id=glossary#glossary_side_chaining)*。使用控制总线上的总线闪避功能或特定事件中的 **Set Voice Volume** 动作可用来触发音量变化。

  在特定场合中，一旦通过降低不重要声音的音量来清理混音后，微调它们的“under **Volume Threshold**”行为，以在它们无法被听见时使用尽可能少的 CPU 和内存资源。

- **实现代码侧动态混音系统**

  注意，播放数限制、优先级和优先级偏置可通过 RTPC 将控制接口交给游戏。您可以毫无障碍地实现一个系统，以根据游戏中的情况来调整这些设置。

### 1.2 Events

#### 1.Events

Event Actions：

详情可见：[Event Action 列表 (audiokinetic.com)](https://www.audiokinetic.com/zh/library/edge/?source=Help&id=event_actions_list#bp1305022)

##### *通用属性*

| Target            | 目标。Action 的应用对象。                                    |
| ----------------- | ------------------------------------------------------------ |
| Scope             | 通过以下两个选项指定 Action 应用于哪些游戏对象：**Game object**，将事件动作作用于触发事件的游戏对象。**Global**，将事件动作作用于所有游戏对象。Default value: Game Object |
| Delay             | 延时，执行 Action 之前的时间量。单位：s Default value: 0 Range: 0 to 600 |
| Fade Time         | Action 逐渐生效所需的时间。渐变将在指定的延迟（如果存在）后开始。单位：s Default value: 0 Range: 0 to 60 |
| Fade-in Curve     | 定义 Action 的淡入曲线形状。Default value: Linear            |
| Value             | 值。Default value: 0                                         |
| Absolute/Relative | 绝对/相对。设置如何应用 Action：**Absolute**:直接使用指定值。**Relative**:目标值将按指定量增减。Default value: Absolute |

##### 1.Play

播放该关联的对象。

##### 2.Stop

- stop

  停止播放这个对象。

  可以将多个对象放入一个stop event，然后用一个event就实现了多个对象的控制。

- stop all

  停止播放所有对象，可添加<u>例外</u>（add exceptions...）

##### 3.Pasue

- Pasue/Pasue all

  pasue:暂停播放。

  pasue all:停止所有对象的播放，但可以添加例外。

- Resume/Resume all

  resume:继续播放该对象。

  resume all:全部继续。继续播放所有暂停的 Wwise 对象，但可添加。

##### 4.Break

中断。停止播放循环声音、振动对象或连续容器，但允许当前对象播放完毕。

##### 5.Seek

- seek

  更改关联 Wwise 对象的播放位置。

- seek all

  更改所有 Wwise 对象的播放位置，但可以添加[例外](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=event_editor#add_exceptions)。

  此操作不会影响当前未播放的对象。



| Seek Type              | 跳转位置使用什么类型的值。共有以下值可用：**Percent**（百分比，默认类型）：以声音总时长的百分比来表示跳转位置。**Time**:* **Time**（时间）：以文件开始后的时间的绝对值来表示跳转位置，单位为秒。Default value: Percent |
| ---------------------- | ------------------------------------------------------------ |
| Seek Percent           | 以声音总时长的百分比来表示跳转位置。此时长会将循环考虑在内。例如，定位到自己循环两次的声音的 50% 会将光标置于第二次循环的开头。无限循环的声音是一种例外情况，因为时长是无限的。在此类情况下，有效跳转位置的计算会以声音未循环来进行计算，然后通过考虑循环来应用。因此，不可能在无限循环声音的循环区域之后（“尾声部分”）进行跳转。在音乐段落内进行定位时，定位是相对于 Entry Cue（开始提示）进行的并且对象持续时间由 Entry Cue 和 Exit Cue（退出提示）进行定义。因此，不可能在 Entry Cue 之前（Pre-Entry（预开始））或在 Exit Cue 之后（Post-Exit（退出后））进行定位。单位：% Default value: 0 Range: 0 to 100 |
| Seek Time              | 以时间绝对值来表示跳转位置。此时长会将循环考虑在内。例如，跳转到自己循环两次的 10 秒声音的 10 秒处会将光标置于第二次循环的开头。同样，不可能在无限循环声音的循环区域后（在结尾部分）进行定位。在音乐段落内进行跳转时，跳转是相对于 Entry Cue 进行的。因此，不可能在 Entry Cue 之前（前导段）进行定跳转但是，可以使用此选项在 Exit Cue 之后（后尾段）进行跳转。单位：s Default value: 0 Range: 0 to 3600 |
| Seek To Nearest Marker | 将最近的标记点作为跳转位置。在标记导入到 Wwise 时将标记嵌入到波形文件中并予以保留。在音乐段落内进行定位时，此选项旨在将有效的位置和 Segment Editor 中编写的最近的 Segment Custom Cue（段落自定义提示点）或 Segment Entry Cue（段落开始提示点）对齐。Exit Cue 将始终会被忽略。Default value: false |

> **Seek/SeekAll 备注和限制**
>
> - 创建 Event 时，若要从第一个 Wwise 声音对象的特定位置开始播放，请在 Seek 动作之后添加 Play 动作。
> - 在跳转时不执行任何音量淡入淡出，因此在声音播放期间进行跳转则可能导致爆音。
> - Music Playlist Container（音乐播放列表容器）和振动对象不支持跳转。
> - 当声音是插件源时，如果插件中实现了跳转，则跳转将由插件执行。当前，只有 [“Silence”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=wwise_silence_plug_in) 支持跳转。
> - 跳转仅适用于当前播放的声音。您不能使用此选项以在属于连续序列部分的对象之间跳转。
> - 如果跳转位置大于声音长度，则声音将停止播放。
> - 如果播放声音的其中一个上层对象是具有以下特殊过渡的连续（随机或序列）容器，则跳转将失败：
>   - Trigger rate（触发率）
>   - 交叉淡变（幅度）
>   - 交叉淡变（功率）
>   - 精确到采样点（只有在当前播放的声音是连续序列的第一个声音时，跳转才会对连续序列生效并做精确到采样点的过渡。）
> - 跳转时间或百分比会将循环（和循环区域）考虑在内（请参阅“Seek percent”和“Seek time”参数的说明）。
> - 在跳转之后，播放可能会因流播放延迟而延迟。在 Music Segment 这种情形中，此延迟等于段落的预读时间（请参阅音乐轨的属性）。段落的预读时间是所有采用流播放的子音轨中最大的流播放预读值。

##### 6.Post

发送事件。从事件内触发别的事件。

##### 7.Bus Volume

- Set

  发送事件。从事件内触发别的事件。

- Reset

  重置音量。将关联 Wwise 总线的音量复位至其原始电平。

- Reset Bus Volume All

  重置所有音量。将所有 Wwise 总线的音量复位至其原始值, 可以添加例外。

##### 8.Voice Volume

- Set

  设置音量。更改关联 Wwise 对象的音量电平。

- Reset

  重置音量。将关联 Wwise 对象的音量复位至其原始电平。

- Reset Voice Volume All

  重置所有音量。将所有 Wwise 对象的音量复位至其原始值, 可以添加例外。

##### 9.Voice Pitch

- Set

  设置音高。更改关联 Wwise 对象的音高。

- Reset

  重置音高，将关联对象的音高复位至其原始值。

- Reset Voice Pitch All

  重置所有声部音高。将所有 Wwise 对象的音高恢复至其原始值，但可添加[例外](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=event_editor#add_exceptions)。

##### 10.Voice High-pass Filter

- Set LPF

  更改作用于关联 Wwise 对象的高通滤波器效果量。

- Reset LPF

  将作用于关联 Wwise 对象的高通滤波器效果量复位至其原始值。

- Reset LPF All

  将作用于所有 Wwise 对象的 Low-Pass Filter 效果量复位至其原始值，但可以添加[例外](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=event_editor#add_exceptions)。

##### 11.Voice Low-pass Filter

- Set HPF

  更改作用于关联 Wwise 对象的高通滤波器效果量。

- Reset HPF

  将作用于关联 Wwise 对象的高通滤波器效果量复位至其原始值。

- Reset HPF All

  将作用于所有 Wwise 对象的 Low-Pass Filter 效果量复位至其原始值，但可以添加[例外](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=event_editor#add_exceptions)。

##### 12.Mute

- Mute

  将关联 Wwise 对象静音。

- Unmute

  解除静音。将关联 Wwise 对象恢复为其“静音前”的原始音量电平。

- Unmate All

  将所有 Wwise 对象恢复为其原始“静音前”音量电平, 但可以添加[例外](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=event_editor#add_exceptions)。

##### 13.Game Parameter

- Set Game Parameter

  更改游戏参数值。

- Reset Game Parameter

  重置游戏参数。将 Game Parameter 值改回默认值。

| Bypass Game Parameter Interpolation | 旁通游戏参数插值。该选项用于旁通 Game Parameter 的内部渐变。Default value: false |
| ----------------------------------- | ------------------------------------------------------------ |

##### 14.State

- Enable State

  在应用 Disable State Action 后，为相关 Wwise 对象重新启用 State。

- Disable State

  为关联 Wwise 对象禁用状态。

  > TIPS:
  >
  > | 技巧                                                         |
  > | :----------------------------------------------------------- |
  > | 此功能基本上可以用 State 的 **None**（无）选项来替代。在很多情况下，将全局 State 目标更改为 **None** 更为简便；不过，有时仍需为特定 Wwise 对象调用 **Disable State** 事件动作。 |

- Set State

  激活特定状态。

  Action 属性分组框中所选 State Group（状态组）和 State（状态）显示在 **Objects**（对象）列中。

##### 15.Set Switch

​	激活特定状态。

​	Action 属性分组框中所选 State Group（状态组）和 State（状态）显示在 **Objects**（对象）列中。

##### 16.Trigger

调用 Trigger，以启动 Stinger。

所选 Trigger 显示在 Objects 列中。

##### 17.Bypass Effect

- Enable Bypass

  旁通作用于关联 Wwise 对象的效果器。

  > | 备注                                                         |
  > | :----------------------------------------------------------- |
  > | 只有使用同一范围选项（All 或者 Specific）时，Enable Bypass 和 Disable Bypass 动作才会相互影响。若针对效果器应用了 Enable/Disable Bypass 动作，必须使用相同范围的选项才能撤消该动作。 |

  例如：使用选项 Specific 的 Enable Bypass 仅可使用带选项 Specific 的 Disable Bypass 进行撤消。

- Disable Bypass

  禁用旁通。取消 Effect 旁通设置，并将 Effect 重新应用于相关的 Wwise 对象。

  > | 备注                                                         |
  > | :----------------------------------------------------------- |
  > | 只有使用同一范围选项（All 或者 Specific）时，Enable Bypass 和 Disable Bypass 动作才会相互影响。若针对效果器应用了 Enable/Disable Bypass 动作，必须使用相同范围的选项才能撤消该动作。 |

  例如：使用选项 Specific 的 Enable Bypass 仅可使用带选项 Specific 的 Disable Bypass 进行撤消。

- Reset Bypass Effect

  重置旁通效果器。将关联对象的旁通效果器选项复位至其原始设置。

- Reset Bypass Effect All

  重置旁通效果器。将关联对象的旁通效果器选项复位至其原始设置。

##### 18.Release Envelope

释音包络。使[包络调制器](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=modulator_envelope)进入释音段，从而结束延音段。

##### 19.Reset Playlist

重置播放列表。返回 [Sequence Container](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=random_sequence_container_property_editor)播放列表的开头，而不播放下一对象。

#### 2.Dynamic Dialogue

**Dialogue Event Editor**

操作逻辑等同于Music Switch Editor。

### 1.3 Soundbanks

详情请见**<u>*3.Soundbank  Layout*</u>**

### 1.4 Game Syncs

#### 1.RTPC

RealTimeParametersController

将特定的对象属性与游戏中的某些参数值绑定。

备注:如果RTPC绑定到初始延迟或优先级（Initial Delay&Priority），使用内置参数来控制RIPC会导致出问题。使用给定游戏对象播放声音时，会计算这些内置参数。因此，它们对大部分声音属性来说都很适用。内置参数控制的 RIPC 不适用于对象的播放逻辑属性 (如 Initial Delay和 Priority) ，因为触发 Play Action 时它们的值是未知的。

##### 1.将游戏参数绑定到内置参数（build-in parameter）

- **Distance**

  距离。游戏对象和听者之间的距离。当多个听者或位置指定到同一个游戏对象时，将使用所有听者与声音位置组合之间的最短距离值。

- **Azimuth**

  方位角。投射在水平面上的听者与游戏对象之间的角，单位为度。0 度值表示声音来自听者正前方，-90 度表示声音来自左侧，90 度表示声音来自右侧，+/- 180 度表示声音位于听者正后方。

  当为游戏对象指定了多个听者或声音位置时，则使用听者和声音之间位置最靠近的角度值。

- **Elevation**

  仰角。听者与游戏对象之间相对于水平线的顶角，单位为度。0 度值表示声音位于听者的同一水平面上；90 度值表示声音位于正上方；-90 度表示声音位于正下方。

  当为游戏对象指定了多个听者或声音位置时，则使用听者和声音之间位置最靠近的角度值。

- **Emitter Cone**

  Emitter Cone（发声体锥）表示发声体朝向与发声体和听者间连线所形成的向量之间的 3D 角度。0 度表示发声体直接朝向听者，180 度表示发声体完全背朝听者。

  当为游戏对象指定了多个听者或声音位置时，则使用听者和声音之间位置最靠近的角度值。

- **Obstruction**

  声障。通过 Obstruction，可访问通过 [SetObjectObstructionAndOcclusion ](https://www.audiokinetic.com/library/edge/?source=SDK&id=namespace_a_k_1_1_sound_engine_a1b3e18a25b405e55ba82de9b70cd11ba.html)API 为游戏对象设置的声障值。

  如果为游戏对象指定了多个听者，则声障取值为给距离当前声音位置最近的听者指定的值。

- **Occlusion**

  声笼。通过 Occlusion，可访问通过 SetObjectObstructionAndOcclusion API 为游戏对象设置的声笼值。

  当为游戏对象指定了多个听者时，则声笼取值为给距离当前声音位置最近的听者指定的值。

- **Listener Cone**

  Listener Cone（听者锥）表示听者朝向与发声体和听者间连线形成的向量之间的 3D 夹角。0 度表示听者直接朝向发声体，180 度表示听者完全背朝发声体。

  当为游戏对象指定了多个听者或声音位置时，则使用听者和声音之间位置最靠近的角度值。

- **Diffraction**

  在使用带有 Room 和 Portal 或几何构造的声音传播路径时，Diffraction 可提供由 Wwise Spatial Audio 计算得出的衍射角度。

  要想接收此内置参数，必须在 Property Editor 的 Positioning 选项卡中选中 Diffraction and Transmission 复选框，并且发声体和听者必须处在由一个或多个 Portal 连通的单独 Room 中或被传给 Wwise Spatial Audio 的几何构造阻挡。

  通过 Room 和 Portal，“发声体”游戏对象可接收与其干声衍射相关的值；由发散角（度）表示传播路径与发声体和听者之间直线路径的偏差角度。由 Spatial Audio 内部注册的房间游戏对象也会接收到一个衍射值，但与其“湿声”衍射相关——即房间内声音扩散后，声场的衍射保留区。湿声衍射角度是传播路径与从门户开口法线之间的发散角，以度为单位。

  在声音可通过多个 Portal 或多条路径到达听者所在位置时，将选用各条路径当中的最小衍射角度。衍射值的范围为 0 - 100，其代表衍射百分比而非衍射角度。

- **Transmission Loss**

  在使用带有 Room 和 Portal 或几何构造的声音传播路径时，Transmission Loss 可提供由 Wwise Spatial Audio 计算得出的透射损失。

  要想接收此内置参数，必须在 Property Editor 的 Positioning 选项卡中选中 Diffraction and Transmission 复选框，并且发声体和听者必须处在由一个或多个 Portal（无论是否启用）连通的单独 Room 中或被传给 Wwise Spatial Audio 的几何构造阻挡。

  计算得出的 Transmission Loss 可应用于直接连接发声体和听者的射线。几何构造的 Transmission Loss 可在几何构造上定义 (AkAcousticSurface::transmissionLoss)，Room 的 Transmission Loss 则在 Room 上定义 (AkRoomParams::TransmissionLoss)。该参数的值域为 0 ~ 100。

  将最大值用于发声体和听者所在 Room 以及与之交叉的所有几何构造表面。

##### 2.创建智能音高曲线

智能音高曲线基于以下两个变量：

- **Native value:** 原始音高对应的 Game Parameter 值。
- **Subdivision level** ：曲线的精度，从 1 到 10 分为十个等级。

**创建智能音高曲线的方法是：**

1. 在 RTPC 坐标图视图中创建音高曲线。

2. 右键点击音高曲线，然后从菜单中选择 **Build Smart Pitch Curve**。

   此时将打开 Build Smart Pitch Curve 对话框。

3. 输入智能音高曲线的原声值。原声值是指录制声音所对应的属性值。

4. 输入智能音高曲线的细分等级。曲线细分等级越高，曲线节段越多，运行时评估时间越长。

5. 点击 **OK**。

| 备注                                                         |
| :----------------------------------------------------------- |
| 只可为 4800 和 -4800 音分之间的音高值创建智能音高曲线。在此范围之外，曲线将在一个极端值处“锁定”。 |

##### 3.查看GameObject

在RTPC界面找到图标

![image-20230417220546976](image/Typora/image-20230417220546976.png)

##### 4.侧链压缩

侧链压缩对游戏来说是非常强大的工具，它方便控制玩家关注焦点，并减少喧闹环境下的嘈杂声。另外对于同类对象，它还有助于安排优先级并获得清晰的混音。

通常，会先在同类对象之间设置优先级系统，再在不同类别之间设置优先级系统。例如，按照这一规则，游戏可判定玩家角色 (PC) 的武器声比非玩家角色 (NPC) 的武器声更重要。然后，旁链压缩就可设置为 PC 武器声播放时降低 NPC 武器声的音量。PC 和 NPC 之间的武器声非常相似；但是，在这种情况下，系统需要确保始终将 PC 声音作为玩家的主要关注焦点。

> **TIPS：闪避信号**
>
> 可以通过修改以下属性和行为来控制信号闪避方式：
>
> - Ducking volume（闪避音量）
> - Fade out（淡出）
> - Fade in（淡入）
> - Curve shape（曲线形状）
> - Recovery time（恢复时间）
> - Maximum ducking volume（最大闪避音量）
>
> ![duck](image/Typora/duck.png)
>
> **闪避某信号的方法如下：**
>
> 1. 将一条**总线**加载到 Property Editor。
>
> 2. 在 **Auto-ducking**（自动闪避）组框中，点击 **Insert**（插入）。
>
>    此时将会打开 Project Explorer - Browser 以及可被该总线要求闪避的总线列表。
>
>    | 备注                                 |
>    | :----------------------------------- |
>    | 总线无法要求自己或其直接父总线闪避。 |
>
> 3. 从列表中，选择在当前总线收到信号时您要求闪避的总线。
>
> 4. 点击 **OK**。
>
>    所选总线已经添加到了闪避列表中。
>
> 5. 通过修改以下设置，定义闪避总线的属性和行为：
>
>    - **Volume**：音量。在当前总线收到信号时，所选总线要降低的音量。
>    - **Fade Out**：淡出。从原始音量淡出至闪避音量的时间。
>    - **Fade In**：淡入。逐渐返回原始音量的时间。
>    - **Curve**：曲线。用于定义淡出和淡入的曲线形状。
>
> 6. 在 **Recovery time**文本框中，指定您要当前总线信号结束到做出闪避的信号淡入开始的时间量。
>
> 7. 在 **Maximum ducking volume**文本框中，指定当前总线在被若干条总线要求闪避后可以衰减的最大量。

可利用实时参数控制 (RTPC) 和 Meter 效果器轻松设置侧链压缩。以下示例分三步介绍了如何让 PC Weapon 音频总线的信号自动压缩 NPC Weapon 音频总线的信号音量。

1. **创建 Game Parameter**：首先，创建一个游戏参数（即 PC_Weapon_Volume），并将取值区间设为 -48 ~ 0，近似代表游戏的动态范围。该游戏参数将被用作总线之间的沟通渠道。

   ![3](image/Typora/3.png)

2. **在总线上插入 Meter 效果器**：然后，对于需要确保信号清晰可闻的总线（本例中为 PC_Weapon 总线），为其插入 Meter 效果器，并选择 Output Game Parameter（输出游戏参数）。

   ![4](image/Typora/4.png)

   Meter 效果器会监控输入的音频信号。在将效果器用于旁链压缩时，将使用 Mode（模式）、Attack（起音）、Release（释音）和 Hold（保持）参数来降低输出信号的响应速度，并将平滑处理后的值发送给游戏参数（本例中为 PC_Weapon_Volume）。

3. **创建 RTPC 曲线**：最后，为需要衰减的总线创建 RTPC 曲线。在本例中，先将总线音量关联至 PC_Weapon_Volume 游戏参数，然后创建衰减曲线。X 轴代表 Meter 效果器计算得出的 RMS 信号，Y 轴代表 NPC Weapon 音频总线音量的衰减量。

   ![5](image/Typora/5.png)

若要使用旁链压缩构建总线层级结构，只需重复上述三个简单步骤即可。

因为 Meter 效果器输出的是通用游戏参数，所以任何可关联至 RTPC 的属性都可由旁链压缩驱动。例如，可以用旁链压缩控制 EQ 频段的增益，从而对特定频段进行陷波处理。再如，可以驱动 Compressor 效果器的阈值、修改 Flanger 效果器的 LFO 频率或者增大 FutzBox Lo-Fi 效果器的失真强度。

#### 2.State

利用最少的内存、CPU、素材和磁盘空间创建最动听音频，这是设计师日常面临的挑战。一种创新而高效的应对方案就是 State。

一个state group内有多个不同的state，然后再在property editor界面选择创建的state group以及其中的state。

##### 1.为同一个state group内的state之间设置切换过渡

①设置统一的过渡。

1. 选中一个state group到property editor。
2. 在 Default transition time 中，为该 State Group 中所有 State 切换设置默认过渡时长。

②自定义state之间的过渡。

1. 在 States 层级中，双击 State Group 将其加载到 Property Editor 中。

2. 点击 **Insert**。

   Transition Time 列表中将新增一行。

3. 在 From 列中，选择源 State。

4. 在 Time 列中，设置两个 State 之间的过渡时间。

5. 在 To 列中，选定目标 State。

6. 要为两个切换方向设置相同的过渡时间，请勾选双向复选框（标记为：<->）。

##### 2.将state指派给对象和总线

###### **1.为音乐对象设置state切换点**

在音乐对象属性级别和总线级别都可以设置切换时间点。

**为音乐对象和总线定义 State 切换点的方法如下：**

1. 将音乐对象或总线加载到 Property Editor。

2. 切换到 States 选项卡。

3. 对于采用的 State Group，在 **Change occurs at** 列中选择下列之一：

   - **Immediate** —— 立即切换状态。如果音轨设置了 Look-ahead time（预读时间），那么经过预读时间之后 State 才会切换。
   - **Next Grid** —— 在下一网格线处 **切换状态**。网格线允许您按照任意间隔长度来分割音乐对象。
   - **Next Bar** —— 在下一小节**切换状态**。
   - **Next Beat** —— 在下一拍**切换状态**。
   - **Next Cue** —— 在下一提示点处**切换状态**。下一提示点可以是 Entry cue（入口提示点）、Exit cue（出口提示点），也可以是 Custom Cue（自定义提示点）。
   - **Next Custom Cue** —— 在下一自定义提示点处**切换状态**。
   - **Entry Cue** —— 在 Entry cue 处**切换状态**。
   - **Exit Cue** —— 在 Exit cue 处**切换状态**。

   当前 State Group 中的所有 State 切换都将发生在指定时间点。

###### **2.自定义对象的State属性**

将 State Group 指派给对象后，您可以自定义该对象在各 State 下的属性。

**为对象自定义 State 属性值的方法如下：**

1. 将对象加载到 Property Editor 中。
2. 切换到 States 选项卡。
3. 通过单击 **Properties**按钮来打开 **State Properties**（状态属性）对话框。
4. 选中受 State Group影响的属性，然后单击 **OK**（确定）按钮。
5. 为各个 State设置属性值。

在工程中使用 State 之前，有必要先了解 State 值会如何与现有属性值和其他适用修改因子（如 RTPC）进行交互。

在将修改因子应用于现有属性值时，会按照以下某一方式决定最终属性值：

- **Absolute**：使用由 RTPC 决定的值，忽略对象的现有属性值。仅可应用一个 RTPC，不支持应用 State（状态）。
- **Sum All Values**：将由 RTPC 或 State 决定的值添加到对象的现有属性值。
- **Use Highest Value**：使用由 RTPC 和对象的现有属性值决定的最大值。
- **Boolean**：使用逻辑 OR 结合由 RTPC 或 State 决定的值。也就是说，若仅设置了一个 RTPC 或 State 值，则设置最终属性值。在这种情况下，会忽略对象的现有属性值。

应用修改因子时所用的方式决定是否使用对象的原始属性值。若属性为绝对值或布尔值，则禁用原始属性控制。否则，原始属性控制保持可用状态。

###### 3.在State之间复制属性值

①在不同State Group之间将State值复制到State

可在不同 State Group 和不同对象类型之间复制粘贴 State 值，包括在 Master-Mixer Hierarchy、Actor-Mixer Hierarchy 和 Interactive Music Hierarchy 之间进行复制粘贴。甚至，还可选择多个目标对象并将值一键粘贴到所有目标对象。

**将 State 值复制到任意 State：**

选择 **Copy**或者按下 **Ctrl+C**，选择 **Paste**或者按下 **Ctrl+V**。

②在同一State Group内复制State值

利用 Copy State Values 对话框，可将 State 值复制到同一 State Group 内的现有或新建 State。另外，还可指定哪些与 State Group 关联的对象会受源对象的 State 值影响。

1. 在以下任一视图中单击 **Copy State Values** 按钮：

   - Property Editor 中的 States 选项卡。
   - Mixing Desk。

   这时将打开 Copy States Values对话框。

   | *技巧*                                                       |
   | :----------------------------------------------------------- |
   | 另外，也可在 Property Editor 的 States 选项卡中或在 Mixing Desk 内右键单击 State 并选择 **Copy State Values**。 |

2. 点击 **State Group** 选择器（**>>**），并选择要复制自定义属性值的 State 所在的 State Group。

3. 点击 **From** 选择器（**>>**），并选择要复制自定义属性值的 State。

   采用该 State Group 的所有对象都将显示在 Affected Object 列表中。

4. 点击 **To** 选择器（**>>**），并执行下列操作之一：

   - 要将自定义属性值复制到新 State，请选择 **New**，为新 State 命名并点击 **OK**。
   - 要将自定义属性值复制到现有 State，则只需从列表中选择该 State 即可。

   Wwise 将分别处理 **Affected Object** 列表中的对象。

5. 在 **Use**列中，针对所有要使用新的 State设置的对象选中复选框。

6. 单击 **OK**来将 State设置应用于所选对象。

#### 3.Switch

在 Wwise 中，需要先将 Switch置于 Switch Group中才能使用。通过用 Switch Group 对 Switch 进行编组，您可以高效管理游戏中不同条件下的声音、音乐和振动对象。

##### 1.将Game Parameter值映射到Switch

复选Use Paremeter选项，并在右下角>>勾选RTPC。

##### 2.StateGroup(State)用于动态对话(DynamicDialogue)

假定您正在开发一款带有现场解说的高尔夫游戏，需要先为其中各个不同类别分别创建 State Group，每个 State Group 包含该类别相应的所有 State。高尔夫游戏需要多种 State Group，包括选手（Player）、俱乐部（Club）、球场（Course）、击球（Shot）、位置（Location）、反应（Reaction）等。

下图说明了高尔夫游戏中，如何将部分类别与 State Group 和其中的 State 进行对应。

![6](image/Typora/6.png)

#### 4.Trigger

Trigger（触发器）将调用 Stinger（插播乐句），让一段简短的 Music Segment与当前音乐叠加播放。

### 1.5 ShareSets

#### 1.Attenuations

详见1.1.4 Positioning部分。

#### 2.Conversion

详见1.1.3 Conversion部分。

#### 3.Effects

给该audio添加效果器。

##### 1.Soundbank 管理

效果器必须打包到 SoundBank 中并由游戏加载。如何将效果器打包到 SoundBank 中取决于其本身的用法。

- **Actor-Mixer：**对于 Actor-Mixer 对象所在的 SoundBank，会同时打包该对象所用的全部效果器，除非在 SoundBank Editor（音频包编辑器）中特地弃用。

- **Master-Mixer：**Master-Mixer 对象（Audio Bus、Auxiliary Bus）所用的效果器会打包到 Init SoundBank 中。

  > TIPS:
  >
  > 在生成 SoundBank 的过程中，Wwise 会自动生成 Init SoundBank。
  >
  > Master-Mixer 和 Audio Device 效果器会自动添加到 Init SoundBank，但有些效果器或效果器的组成部分并不会如此。以下条目不会自动添加到 Init SoundBank：
  >
  > - **效果器媒体：**Init SoundBank 不会打包任何媒体文件。效果器媒体必须打包到 User-defined SoundBank 并在使用效果器之前由游戏加载。
  > - **运行时加载的效果器：**游戏可利用 SDK 函数在运行时更改 Audio Bus（音频总线）或 Audio Device 效果器。若所要加载的效果器在生成 SoundBank 的过程中被工程使用（即被 Audio Bus、Auxiliary Bus 或 Audio Device 引用），则会将其打包到 Init SoundBank 中。否则，必须将效果器打包到 User-defined SoundBank 中并在使用之前由游戏加载。

- **Audio Device：**Audio Device（音频设备）对象所用的效果器会打包到 Init SoundBank 中。

##### 2.离线渲染

到需要离线渲染的对象，打开property editor中的effect，勾选对应效果器的render即可。

若针对某个效果器同时选中了 Bypass 和 Render 选项，则将优先应用 Bypass 选项而不会对效果器进行渲染。

无法将 RTPC 应用于经过渲染的处理器，因为在渲染后无法更改这些效果器的属性。

在效果器链中渲染一个效果时，也将自动渲染它之前所有的效果器。（会渲染序号比自己小的所有效果器）。

##### 3.CPU占用

一般来说，在 Master-Mixer 层级应用效果器时，占用的 CPU 资源会比其他层级少些。

#### 1.5.4 Modulators

##### 1.LFO

LFO（低频振荡器）用于为属性值带来随时间变化的调制。其属性为：

| 界面元素             | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| Depth                | 深度。振荡器的幅度变化。最大幅度为 1.0。单位：% Default value: 100 Range: 0 to 100 |
| Frequency            | 频率。每秒钟的周期数。单位：Hz Default value: 1 Range: 0 to 20000 Units: Frequency |
| Waveform             | 调制器的波形包含以下选项：Sine/Triangle/Square/Saw up/Saw down/Random:Random：选择 Random 将在调制器每次运行时随机应用电平。Default value: Sine |
| Smoothing            | 平滑。对波形进行低通滤波，从而平滑尖锐的边缘。单位：% Default value: 0 Range: 0 to 100 |
| PWM                  | 脉冲宽度调制。脉冲波的宽度；仅用于Square（方波）波形。单位：% Default value: 50 Range: 0 to 100 |
| Attack               | 起音。振荡器达到满幅度所用的时间。单位：s Default value: 0 Range: 0 to 100000 |
| Initial Phase Offset | 初相。振荡器波形的初始相位。单位：° Default value: 0 Range: -180 to 180 |
| Scope                | 作用域。定义如何创建 LFO 实例：**Voice**:声部。为每个声音/对象播放创建的 LFO 实例。**Note/Event**:音符/事件。为每个播放实例或在 MIDI 环境中使用时的音符创建一个 LFO 实例。**Game Object**:游戏对象。为每个游戏对象创建一个 LFO 实例。**Global**:全局。为整个工程创建单个 LFO。Default value: Note or Event |

**使用 LFO 调制 Voice Volume 的方法是：**

1. 在 Project Explorer 中，选择要添加 LFO 的对象。
2. 在 Property Editor 中，转至 RTPC 选项卡。
3. 在 RTPC 列表中，点击 [>>] 按钮。
4. 从选择器菜单中，选择 **Voice Volume**。
5. 点击 X 轴选择器按钮。
6. 从选择器菜单中选择 LFO > Default (Custom)。
7. 点击 [...] 按钮以编辑 LFO 属性。
8. 编辑曲线以设置调制范围。

LFO 对象可以被创建成 Custom 或 ShareSet。Custom 对象是原地保存的，即直接保存在拥有该对象的对象内部。ShareSet 被保存在一个单独的工作单元中，并且可以在多个对象之间进行重用。

##### 2.包络envelope

| Attack Time                 | 起音时间。定义了当琴键第一次被按下时，电平从零值上升至峰值所用的时间。单位：s Default value: 0.2 Range: 0 to 10000 |
| --------------------------- | ------------------------------------------------------------ |
| Attack Curve                | 起音曲线。把起音曲线的斜率从线性的默认斜率（50%）调整到其它形式：指数风格的包络（0%），其变化速率在最开始较慢，随后逐渐变快对数包络（100%），其变化速率在最开始较快，随后逐渐变慢单位：% Default value: 50 Range: 0 to 100 |
| Decay Time                  | 衰减时间。指随后从起音电平下降到指定延音电平所用的时间。单位：s Default value: 0.2 Range: 0 to 10000 |
| Sustain Level               | 延音电平。指在释放按键前，声音持续期间主序列的电平。单位：% Default value: 100 Range: 0 to 100 |
| Release Time                | 释音时间。指释放按键后电平从延音电平衰减到零值所用的时间。单位：s Default value: 0.5 Range: 0 to 10000 |
| Scope                       | 作用域。定义如何创建包络实例：**Voice**:**Voice**（声部）：为每个声音/对象播放创建的包络实例。**Note/Event**:**Note/Event**（音符/事件）：为每个播放实例或在 MIDI 环境中使用时的音符创建一个包络实例。Default value: Note or Event |
| Trigger On                  | 可能触发包络（即进入起音段）的 Action/MIDI Event：**Play**:Play Action 或 MIDI Note 事件**Note-Off**:仅 MIDI Note-Off 事件Default value: Play |
| Auto Release                | 决定是否需要 Action/MIDI Event 才能让包络退出延音段，并进入释音段。如果进行了设置，则该包络将在经过了 **Sustain Time**（延音时间）后退出延音段。如果没有设置，则该包络将在特定情形下退出延音段：游戏可通过 [Release Envelope](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=Help&id=event_actions_list#release_envelope) 事件让包络进入释音段。如果包络是由 MIDI Note-On Event 触发，还可以通过 MIDI Note-Off Event 让其进入释音段。Default value: false |
| Maximum Sustain Time        | 延音时间。定义了该包络在进入释音段之前，在延音段中维持的时间。如果已设置 **Auto Release**，则此值有效。单位：s Default value: 0 Range: 0 to 10000 |
| Stop playback after release | 在释音后停止播放。如果进行了设置，则被关联声音的播放将在释音段完成以后终止。Default value: true |

①用在 MIDI 环境中时，包络配置成在 Note-On 或 Note-Off（音符停止）的方式下播放声音。

如果以 Note-On 的方式播放声音：

- 包络配置成在 Note-On 时触发（**Trigger On** 参数）。
- 包络一直持续到第一次出现以下情况：
  - 收到 **Release Envelope** 事件，
  - 收到 MIDI Note-Off 事件，
  - 出现延音段的最大时长（设置了 **Auto Release**）。

如果以 Note-Off 的方式播放声音：

- 包络配置成在 Note-Off 时触发（**Trigger On** 参数）。
- 包络一直持续到第一次出现以下情况：
  - 收到 **Release Envelope** 事件，
  - 出现延音段的最大时长（设置了 **Auto Release**）。

②当用于播放声音的一般环境中时：

- 包络配置成遇到播放动作时触发（**Trigger On** 参数）。
- 包络一直持续到第一次出现以下情况：
  - 收到 **Release Envelope** 事件，
  - 出现延音段的最大时长（设置了 **Auto Release**）。

##### 3.时间的使用

使用 Time Modulator（时间调制器），其中 RTPC 的 X 轴为时间元素。

| 界面元素             | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| Initial Delay        | 初始延迟。等待指定时长后，再开始基于时间的调制。备注：在此延迟期间，属性值在 0 时间点处保持为 RTPC 坐标图所决定的值。单位：s Default value: 0 Range: 0 to 4 |
| Duration             | 时长。在正常播放速率下，基于时间的调制进行一次迭代的时长。备注：实际总时间受 Initial Delay、Playback Rate 和 Loop Count 影响。单位：s Default value: 1 Range: 0.1 to 100 |
| Loop Count           | 循环次数。调制的重复次数。播放容器的次数。Default value: 1 Range: 0 to 100 |
| Playback Rate        | 播放速率。调整关联声音的播放速率：若值等于 1，则正常播放若值小于 1，则减速播放若值大于 1，则加速播放Default value: 1 Range: 0.25 to 4 |
| Scope                | 范围。定义如何创建时变调制器实例：**Voice**:为每个声音/对象实例播放创建一个时间调制器实例。**Note/Event**:为每个播放实例或在 MIDI 环境下使用时的音符创建一个时间调制器实例。Default value: Note or Event |
| Trigger On           | 触发时机。可能触发时变调制器的 Action/MIDI Event：**Play**:Play Action 或 MIDI Note 事件**Note-Off**:仅 MIDI Note-Off 事件Default value: Play |
| Stop playback at end | 若勾选，达到总时长时（初始延迟、循环次数和播放速率都会计入）将停止播放关联声音。 若未勾选，则达到总时长后，经过调制的属性值将保持为 RTPC 曲线图所决定的最终值。Default value: true |

从某些方面来说，Time Modulator 与 [LFO Modulator](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=Help&id=working_with_lfos) 和 [Envelope Modulator](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=Help&id=working_with_envelopes) 存在很大差异：

- 它的曲线图可以包含多个控制点，而其他调制器最多只有两个控制点。这方便设计师在特定时间点处精确调整属性值。
- 它可以使用所有可用的[坐标图曲线](https://www.audiokinetic.com/zh/library/2022.1.3_8179/?source=Help&id=specifying_shape_of_curve_between_control_points#curve_shapes)，而其他调制器只能使用线性和分贝标度的曲线。
- 它只包含两项可启用 RTPC 的属性：Initial Delay（初始延迟）和 Playback Rate（播放速率）。

Time Modulator 可创建为 Custom（自定义）或 ShareSet（共享集）。Custom 对象是原地保存的，即直接保存在拥有该对象的对象内部。ShareSet 存储在单独的 Work Unit（工作单元）中，并可被多个对象复用。

#### 1.5.5 Virtual Acoustics



#### 1.5.6 Metadata



### 1.6 Sessions

#### 1.Sound Caster

详情请见**<u>*4.Mixer Layout*</u>**中<u>***3.Soundcaster***</u>

#### 2.Mixing Sessions

详情请见**<u>*4.Mixer Layout*</u>**

#### 3.Control Surface Binding（难以用到，用到在学）

用来定义控制设备在 Wwise 中的行为。

控制设备可以用来控制 Wwise 的功能或工程属性。

- Group/Binding

  - Global

    Global 组中是与特定对象有关的绑定。

    目标对象可以直接在绑定中指定。

    **为键盘快捷键创建绑定。**

    Global Binding（全局绑定）可用于触发全局命令，这些命令和 Keyboard Shortcuts（键盘快捷键）管理器中的命令相同。您可以创建一个由 Keyboard Shortcu 管理器和 Control Surface Session 的绑定触发的全局命令。

    在 Control Surface Binding 视图中创建全局绑定的方法如下：

    1. 选择 **Global**（全局）组。

    2. 点击 **Add & Learn Binding**（添加和学习绑定）按钮。

       绑定条目将被添加，并且 **Learn** 按钮也将激活。

       Properties/Command 可在 UI 变绿时进行选择。

    3. 点击 [>>] 按钮选择命令。

    4. 从菜单中选择 **Global commands...**（全局命令...）

    5. 浏览找到要启动的全局命令。

    6. 点击 OK。

    7. 使用所需的硬件控件来作绑定。

       **Learn**（学习）按钮处于已停用状态。

       绑定现在创建好并且就绪了。

       （待续...）

       [将控制设备连接到 Wwise (audiokinetic.com)](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=connecting_control_surface_device_to_wwise)

### 1.7 Queries

在此选项卡中，可创建并管理查询 Work Unit。

#### Query Editor

可以创建和运行查询。查询大体上是请求显示工程中的一部分信息。当查找特定对象时，您可创建非常具体的查询；也可创建一般性查询，例如查找特定平台中的所有声音对象。另外，Query Editor 还支持 WAQL 查询。有关 WAQL 查询的详细信息，请参阅[了解 Wwise Authoring Query Language (WAQL)](https://www.audiokinetic.com/library/edge/?source=SDK&id=waql_introduction.html)。

- Query From

  从此处查询。所要搜索的对象类型或其他工程元素，如声音、Audio Bus（音频总线）、SoundBank（音频包）等。

  除此之外，也可从列表中选择 **WAQL Query** 来启用 WAQL Query 字段。

- Start From

  起始位置。工程层级结构中开始查询的位置。

- Criteria

  - Browser

    浏览器。查询所依据的条件列表，如 Name 或 Output Bus。

  - Criteria

    为查询所选择的条件列表。

    - Operator

       当查询中包含多个条件时使用的逻辑运算符。

      可使用以下运算符：

      **And**，匹配所有条件的项。

      **Or**，匹配至少一个条件的项。

- Result

  只选了一部分陈述。

  - Size Preview

    右击选择configure column...选择勾选。

    - Total Size

      该元素 Media Size 和 Structure Size 大小的总和，近似等于它们将在 SoundBank 中占用的大小。

    - Media Size

      Wwise 对象的媒体文件（包括转换的源文件，事件和效果器）将在 SoundBank 中占用的近似大小。

      > 对于需要授权的 Effect，除非输入相应授权码，否则其大小值将不计入。

      如果父对象同时设置了 [media files are set to stream](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=streaming_media) 与 **Zero latency**，将只显示较小的音频缓冲区大小。如果父对象选择了流播放但未启用 **Zero Latency** 设置，其大小将不会显示。

      > 在很多情况下，对象的媒体大小已改变，但视图未更新。要确保所有的预计大小都是最新的，请使用快捷菜单中的 **Refresh All Size** 命令。要更新一个或多个特定项，请选择它们并使用 **Refresh Size** 快捷菜单选项。

    - Object Size

      Wwise 对象将在 SoundBank 中占用的大致大小，不包括磁盘上声音文件的大小。

      Action 不具有大小。只位于 Init Bank 中的对象，如 Game Sync，大小将不计入。

    - When  limit  is reachedStructure

      对象及其子结构将在 SoundBank 中占用的近似大小，其中对象大小即为 Object Size。

## 2.Profiler Layout（暂不用，待完善）

三个布局涵盖了不同范围的性能分析。

| Layout（布局）       | 描述                                                         | 所含视图                                                     |
| :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Profiler             | Profiler 布局允许您在捕获时监控每个游戏元素的性能。您还可以在将 Soundcaster 触发的模拟内容集成到游戏之前监视这些模拟的性能。 | “Advanced Profiler”一节<br/>“Capture Log”一节<br/>“Performance Monitor”一节 |
| Voice Profiler       | Voice Profiler（性能分析器）布局允许监控游戏中捕获的声部，并检查驱动每个声部的结构。 | “Advanced Profiler”一节<br/>“Capture Log”一节<br/>“Performance Monitor”一节 |
| Game Object Profiler | Game Object Profiler 布局允许您监控游戏中注册的游戏对象，包括发声体、听者或既是发声体又是听者的对象。 | “AdvancedProfiler”一节<br/>“Capture Log”一节<br“Performance Monitor”一节 |

### 2.1 Capture Log

用于捕获并记录来自声音引擎的所有信息。用于捕获并记录来自声音引擎的所有信息。

双击 Capture Log 中的条目，即可在 Property Editor 或 Event Editor 中加载相关的 Wwise 对象或事件。

此外，还可在条目上按下 Ctrl+/ 以便移动 Performance Monitor（性能监控器）中的时间光标，并将所有其他视图同步至同一时间。

可以将列表保存为 PROF 文件，以便之后在 Wwise 中重新加载它。

#### 1.筛选器工具栏

此视图中的筛选器工具栏能够帮助您减少视图中显示的信息数量，以便快速锁定想要查看的元素。有关更多详细信息，请参阅 [“在性能分析视图中筛选数据”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=filtering_in_profiling_views) 章节。

![ProfilerFilteringToolbar](image/Typora/ProfilerFilteringToolbar.png)

1.**UnlinkFilter**:禁止在多个筛选器视图之间进行同步。

2.**Text Filter**：通过指定文本来筛选内容。系统会将您所指定的字词与内容中所含名称或字符串的开头进行匹配。键入的字词越多，显示的结果越细化。匹配项不区分大小写。有关高级用法的信息，请参阅“[“使用性能分析器筛选器表达式”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=using_profiler_filter_expressions)”。

3.**Object Filter**：通过指定 Wwise 对象来筛选内容。系统会将您所指定的 Wwise 工程对象与视图中的内容进行匹配。同时，还会依据对象关系（如父子对象关系和输出总线关系）对内容进行匹配。

4.**Browse Object Filter**：显示 Project Explorer 浏览器，以便选择所要筛选的对象。

5.**Mute/Solo Filtering**：若启用，则从结果中排除激活了 Mute 的对象，而只显示激活了 Solo 的对象。

6.**Options**：显示其他操作。

#### 2.界面元素

| 界面元素                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| ![eyes](image/Typora/eyes.png)                               | 打开 Capture Log 的 Filter 对话框，您可以在其中定义 Capture Log 输出的筛选条件。 |
| ![save](image/Typora/save.png)                               | 将 Capture Log 内容保存为 PROF 文件或制表符分隔的文本文件。文本文件将包含任何已作用于列表的筛选或分类。 |
| ![search_2](image/Typora/search_2.png)        ![search_2](image/Typora/search.png) | 打开搜索框，在其中输入标准字母和数字会筛选掉视图中不相匹配的元素。阅读 [“使用表格”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=using_tables) 了解详细信息。点击搜索图标左侧的 Close（关闭）图标，以关闭搜索字段并删除筛选器。*info_outline*备注搜索不包括[“List View（列表视图）”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=list_view), [“Query Editor”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=query_editor), [“MIDI Keymap Editor 视图”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=midi_keymap_editor_midi_keymap_editor), and [“Reference View 视图”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=reference_view)中折叠起来的节点。 |
| ![blue](image/Typora/blue.png)![orange](image/Typora/orange.png)           （标志）       ![white](image/Typora/gray.png)![white](image/Typora/white.png) | 圆点的颜色随底色变化：在 Classic（经典）主题下显示为青绿色，在 Dark（深色）主题下显示为橙色。这样方便指示在距离 Performance Monitor 时间光标位置 100 ms 内捕获了 Capture Log 中的哪些条目。颜色越亮，条目离时间光标位置越近。您可以将光标强制移动至特定日志条目所在的时间戳：从快捷菜单 (Ctrl+/) 选择 **Move Cursor to Timestamp**（将光标移至时间戳）。或者直接双击时间戳。 |
| （上行内容）                                                 | 关联圆点的颜色随底色变化：在 Classic（经典）主题下显示为深灰色，在 Dark（深色）主题下显示为白色。这样方便指示 Capture Log 中哪些条目是相互关联的。它们只有在日志中选择单个条目时才会显示。 |
| Timestamp                                                    | 时间戳。捕获该条目时的游戏时间。时间戳有助于识别在 Wwise 中模拟中 或游戏中发生的情形的精确时间。*info_outline*技巧按下 Ctrl+/ 或双击时间戳可将 Game Profiler 光标移动至条目所对应的时间。 |
| Type                                                         | 记录在 Capture Log 中的条目类型。关于 [不同捕获信息类型](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=capture_log_filter#capture_log_types) 的简短列表和说明，请参阅 Capture Log Filter 页面。 |
| Description                                                  | 有关日志项的描述性信息。还可能包括属性值更改、状态更改等相关信息。 |
| Wwise Object Name                                            | Wwise 对象名称。与日志条目关联的 Wwise 对象的名称（如存在）。如果多个 Wwise 对象与日志项相关联，则会显示“Multiple Object”。如果没有与日志项相关联的 Wwise 对象，则会显示“No Element”。 |
| Wwise Object ID                                              | Wwise 对象 ID。与日志条目关联的 Wwise 对象的 ID（如存在）。如果多个 Wwise 对象与日志项相关联，则会显示“Multiple Object”。如果没有与日志项相关联的 Wwise 对象，则会显示“No Element”。*info_outline*备注在使用 Capture Log 中的搜索字段时，ID 必须完整且准确才会匹配。 |
| Game Object Name（游戏对象名称）                             | 与日志项关联的游戏对象的名称。“Transport/Soundcaster” 可跟没有关联游戏对象的 Wwise 对象所触发的条目相关。 |
| Game Object ID（游戏对象 ID）                                | 游戏对象 ID。与日志条目关联的游戏对象的 ID。“Transport/Soundcaster” 可跟没有关联游戏对象的 Wwise 对象所触发的条目相关。*info_outline*备注在使用 Capture Log 中的搜索字段时，ID 必须完整且准确才会匹配。 |
| Scope                                                        | 作用域。日志项作用于游戏内的对象的范围。作用域可以是两种类型之一：**Global**，意味着日志条目适用于所有游戏对象。**Game object**，意味着日志条目仅适用于在游戏对象列中的游戏对象。 |

#### 3.Capture Log筛选器

Capture Log Filter（捕获日志筛选器）对话框用来根据条目类型、Wwise 对象名称和作用域针等一系列筛选器选项在 Capture Log 中加入或删除条目。

图标：![eyes](image/Typora/eyes.png)

##### 1.Types

| Types（类型）                                |                                                              |
| -------------------------------------------- | ------------------------------------------------------------ |
| Notifications                                | 通知。显示来自声音引擎的通知。通知包含声音引擎处理动作的状态。例如：事件播放。开始延迟。 |
| Markers（标记）                              | 显示音频文件标记和音乐片段自定义提示。                       |
| Event                                        | 显示由 Wwise 或游戏引擎触发的动作事件和对话事件。            |
| Actions                                      | 显示事件中的动作。                                           |
| Properties                                   | 属性。显示对属性值所做的更改。                               |
| States                                       | 状态。显示状态更改。                                         |
| Switches                                     | 切换开关。显示切换开关更改。                                 |
| SoundBanks                                   | 显示与 SoundBank 有关的信息，包括 SoundBank 的名称。         |
| Events Preparation（事件准备）               | 准备事件。显示使用[ `PrepareEvent()`](https://www.audiokinetic.com/library/edge/?source=SDK&id=namespace_a_k_1_1_sound_engine_a98b617d05618e6d18680d198845ff3d7.html)和[ `PrepareGameSyncs()`](https://www.audiokinetic.com/library/edge/?source=SDK&id=namespace_a_k_1_1_sound_engine_ad5fc8ab51eecee6511d5c61b03659fa4.html)函数准备的事件和游戏同步器。 |
| Error                                        | 错误。显示声音引擎出现的任何错误。                           |
| Messages                                     | 消息。显示声音引擎发送的任何消息。消息通常会传达问题或特殊信息。这些消息可在游戏引擎中进行编程。例如：未知事件名。主人公的脚步声。 |
| MIDI 事件                                    | MIDI 事件。显示声音引擎播放的所有 MIDI 事件。根据 MIDI 选项卡的 [MIDI Events 信息](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=midi_tab_actor_mixer_objects#midi_events) 及 [Filters](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=midi_tab_actor_mixer_objects#midi_filters) 设置，显示从 MIDI 文件得到的信息。其中将显示通道数（1 - 16），MIDI 事件名（Note-on 音符开或 Note-off 音符关），键位以及力度。 |
| API Calls                                    | 显示对声音引擎的 API 调用。这里显示了 API 的大致类别，可根据需要选择或排除，包括：Event、RTPC、Game Sync、Game Objects、Listener Spatialization （空间定位）、Positions（定位）、Auxiliary Bus、Obstruction/OcclusionMotion（振动）、MIDIDynamic Sequence （动态序列）、Spatial Audio、其他，列出的每个调用都将显示相应函数以及传递的参数（如果存在）。 |
| ![select all](image/Typora/select all.png)   | 选择列表中的所有类型。                                       |
| ![select none](image/Typora/select none.png) | 清除列表中所有类型的选择。                                   |

##### 2.Include关系

| Include parents and busses（使用 Object Filter 时） | 包含父级和总线。将包含针对所选对象的所有父对象层级和子对象可能连通到的所有总线的通知。 |
| --------------------------------------------------- | ------------------------------------------------------------ |
| Include game object relations                       | 包括游戏对象关系。将包含发生在与所选对象具有相同的游戏对象的那些对象上发生的事情。例如，假定您已经选择了 Sound A 作为筛选后的对象，并且它是在已经播放了 Sound B 的某个游戏对象播放。它将在 Capture Log 视图中的该游戏对象上添加所有关于 Sound B 的播放关系。 |
| Scope                                               |                                                              |
| Global                                              | 全局。显示适用于所有游戏对象的条目。                         |
| Game object                                         | 游戏对象。仅显示适用于特定游戏对象的条目。                   |
|                                                     |                                                              |
| Include playback relation                           | 显示播放关系。显示目标对象所在事件的所有通知，即使它们与目标对象没有直接联系。它会显示所有元素，用蓝点符号链接在一起。 |
| ![Reset](image/Typora/Reset.png)                    | 将该对话框重置为默认设置。                                   |
| ![Close](image/Typora/Close.png)                    | 关闭。关闭窗口。                                             |

##### 3.Capture Log中 报告的错误

不在此文档中陈列，附上文档链接。

[Capture Log 中报告的错误 (audiokinetic.com)](https://www.audiokinetic.com/zh/library/edge/?source=Help&id=common_errors_reported_in_capture_log)

### 2.2 Performance Monitor

性能监视器显示有关性能的信息，包括声音引擎执行的各个活动的 CPU、内存和流播放等内容。

曲线图的右侧列出了一系列关键性能计数器，用于显示图形中各时间点的实际性能。

可以通过在坐标图中拖动 Performance Monitor 的时间光标来查看特定时间点上各个计数器的性能。

除此之外，还可按下 **Alt+逗点** (,) 来向左移动一个音频帧，或按下 **Alt+句点** (.) 来向右移动一个音频帧。

| 界面元素                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| ![performance setting](image/Typora/performance setting.png) | 单击视图右上角的 View Settings 图标以弹出 [“Performance Monitor Setting”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=performance_monitor_settings) ，可在其中指定曲线图和性能数据列表中显示哪些内容。 |
| 性能数据列表                                                 | 性能[计数器](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=performance_monitor_settings#profiler_counters)的列表，在捕捉过程中会实时更新相应数值，并且根据曲线图时间轴中的选定时刻显示数值。可以在 [“Performance Monitor Setting”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=performance_monitor_settings) 的 **Show in List** 列中选定要列出的的性能计数器。 |

#### Performance Monitor Setting

##### 1.Counter

计数器。可在 Performance Monitor 中显示的计数器的名称。

有些计数器并不适用于所有平台。

- **API calls**: 在特定时刻，针对声音引擎的 API 调用数量。开放调用的数量包括对同一个函数的多次调用。所以比如在某一时刻有 5 个不同的 API 函数被调用，但由于对同一个函数进行了多次调用，所以总调用次数可能要比 5 高得多。

- **Command Queue Size**：当前为 Command Queue 分配的内存量，用于储存游戏应用程序发送给声音引擎的命令。命令队列的默认大小为 256 KB。最大值可以在 Wwise SDK 中进行设置。

  例如：播放事件（Play Event）。RTPC = 50。

  只有在 [“Profiler Settings”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=profiler_settings) 中启用了 **API Calls** 数据类型，才会在 Capture Log 中显示相应的命令。

- **Command Queue Used**：当前时刻 Command Queue 的占用百分比。

- **CPU - Plug-in Total**：在指定时刻，插件占用的 CPU 比例。

- **CPU - Total**: 音频线程的 CPU 占用量。Audio Thread CPU 所基于的计数器会从唤醒处理音频的音频线程时开始计数，并在音频线程处理完成时结束计数。根据所在平台和运行在同一 CPU 核心上的其它线程的优先级，此数字可能会比声音引擎的实际 CPU 占用大很多。但是，您可以将其视为“音频运算资源百分比”，也就是说，如果它接近 100%，则音频将很有可能无 CPU 可用。

- **Hardware Audio Decoder Usage**：任意时刻所用 Hardware Audio Decode 资源的百分比。

  只有在选择特定平台（PS4 和 PS5）时才会显示此计数器。

- **Loaded Banks (Memory)**：当前内存中加载的 SoundBank 大小。

- **Loudness Momentary**(Instance A to D)[“Loudness Meter”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=loudness_meter)：瞬时（0.4 秒矩形时间窗口）响度级。

- **Loudness Short-term**(Instance A to D)[“Loudness Meter”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=loudness_meter)：短期（3 秒矩形时间窗口）响度级。

- **Number of Active Events**：由游戏发送（尚未达到相应 Active Listener 上限）的事件数量。

- **Number of Active Listeners**：活跃的听者数量。

- **Number of Active Sends**：活跃的 Game-defined 或 User-defined 辅助发送数量。

- **Number of Audio Objects Used (System)**：在任意时刻播放的 System Audio Object 数量。

- **Number of Prepared Events**： 使用函数 [`PrepareEvent()`](https://www.audiokinetic.com/library/edge/?source=SDK&id=namespace_a_k_1_1_sound_engine_a98b617d05618e6d18680d198845ff3d7.html)进行预备的事件数量。

  当预备 Event 时，只有该 Event 关联的特定媒体会被加载到内存中。

- **Number of Registered Objects**：当前时刻已注册的游戏对象数量。

- **Number of State Transitions**：当前时刻 State 之间的过渡数量。

- **Number of Streams**：当前时刻打开的流播放的总数。

- **Number of Streams (Active)**：激活的流数量。在上一个性能分析帧中，如果流播放需要或正在等待至少一条 I/O 传输时，则该流播放处于活跃状态。

- **Number of Transitions/Interpolations**：当前时刻活动的淡变/过渡/插值的数量。

- **Number of Voices (Physical)**: 当前时刻播放的实声部数量。

- **Number of Voices (Total)**: 当前时刻播放的音频声部或独立实例总数（实声部和虚声部）。

- **Number of Voices (Virtual)**：在指定时刻，虚声部列表中的虚声部数量。

- **Output Peak**：在指定时刻，声音引擎的峰值输出，单位为 dB。

- **Prepared Events (Memory)**： 已预备的事件所占用的内存大小。该大小也包括预备事件之后，因此而加载到内存中的所有媒体文件的大小。

- **Profiler Bandwidth**：由游戏发送到 Wwise 设计工具的性能分析数据量。单位：KB/s（千字节/秒）。

- **Spatial Audio - Computed Edges**：此帧当中已处理的衍射边缘数。Spatial Audio 针对边缘可见性构建有相应的内部数据结构。对此，每帧都会逐步加以更新。

  已知问题：在 [“Audio Preferences”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=audio_preferences) 中选择 Multi-Core Rendering 时，Spatial Audio 计数器（Spatial Audio - CPU 除外）可能无法正确显示数值。

- **Spatial Audio - CPU**：Spatial Audio 的总计 CPU 用量，表示为音频帧的百分比。该值约等于 Spatial Audio - Raytracing CPU、Spatial Audio - Path Validation CPU、Spatial Audio - Portal Raytracing CPU 和 Spatial Audio - Portal Path Validation CPU 之和。注意，这跟 Advanced Profiler CPU 选项卡中显示的 CPU 百分比有所不同：Spatial Audio CPU 表示每次调用 Spatial Audio 时的 CPU 占用率，而 CPU 选项卡中所示 Spatial Audio CPU 百分比则表示一帧之内的累计 CPU 占用率。除此之外，CPU 选项卡中所示实例数表示每帧调用 Spatial Audio 的次数。在调用 Spatial Audio 时并不一定会触发路径的计算。Spatial Audio - CPU 值取决于发声体的数量、几何构造的复杂程度（即三角形和衍射边缘的数量）和所启用的功能（即反射阶数、衍射和对衍射的反射）。

- **Spatial Audio - Diffraction Edges**：当前 3D 环境中的衍射边缘总数。

- **Spatial Audio - Emitters Processed**：由 Spatial Audio 处理的发声体数量：对于每个发声体，Spatial Audio 都会对反射和衍射路径进行验证。

- **Spatial Audio - Path Validation CPU**：Spatial Audio 在路径验证阶段的 CPU 用量，表示为音频帧的百分比。在路径验证阶段，Spatial Audio 会对射线追踪阶段找到的以及之前帧中验证过的潜在反射和衍射路径进行验证。Spatial Audio - Path Validation CPU 成本主要受发声体数量影响：发声体越多，成本就越高。

- **Spatial Audio - Portal Path Validation CPU**：Spatial Audio 在验证从 Portal 到听者和发声体的路径时的 CPU 用量，表示为音频帧的百分比。Spatial Audio - Portal Path Validation CPU 成本主要受发声体数量和 Room 与 Portal 通路的复杂程度影响。

- **Spatial Audio - Portal Raytracing CPU**：Spatial Audio 在对环境进行采样以查找从 Portal 到听者和发声体的潜在路径时的 CPU 用量，表示为音频帧的百分比。Spatial Audio - Portal Raytracing CPU 成本主要取决于几何构造的复杂程度（即三角形和衍射边缘的数量）和所启用的功能（即反射阶数、衍射和对衍射的反射）。该值不受发声体的数量影响。

- **Spatial Audio - Primary Rays**：由 Spatial Audio 标定的初级射线数量。

- **Spatial Audio - Raytracing CPU**：Spatial Audio 在射线追踪阶段的 CPU 用量，表示为音频帧的百分比。在射线追踪阶段，Spatial Audio 会对环境进行采样以查找听者和发声体之间的潜在反射和衍射路径。Spatial Audio - Raytracing CPU 成本主要取决于几何构造的复杂程度（即三角形和衍射边缘的数量）和所启用的功能（即反射阶数、衍射和对衍射的反射）。该值不受发声体的数量影响。

- **Spatial Audio - Triangles**：当前 3D 环境中的三角形总数。

- **Total Media (Memory)**：已加载到内存中的媒体文件的总大小。

- **Total Reserved Memory**：预留或映射到物理内存的内存总量（运行时内存分配不一定会使用）。

- **Total Streaming Bandwidth**：在指定时刻，流播放占用的总带宽。此值包含 Low-level I/O 传输以及通过 Stream Manager 缓存进行的传输（不需要访问磁盘），因此此值大于或等于总 I/O 带宽。

- **Total Streaming Bandwidth (Low-Level)**：在指定时刻，流播放占用的总带宽。总底层播放流带宽。该值仅会考虑 Low-level I/O 传输，并且不考虑 Stream Manager 中通过缓存数据所做的虚拟传输。

- **Total Used Memory**：运行时内存分配所用的内存总量。

##### 2.Show in Graph

在坐标图中显示。确定计数器是否显示在 Performance Monitor 坐标图视图中。

##### 3.Show in List

在列表中显示。确定计数器是否显示在 Performance Data 列表中。

##### 4.Graph Min

坐标图最小值。为各个计数器定义坐标图带的最小值域。选择高亮的字段进行修改。

在曲线图底部显示的粗线表示当前值小于 Graph Min（图表下限）。

##### 5.Graph Max

坐标图最大值。为各个计数器定义坐标图带的最大纵轴值域。选择高亮的字段进行修改。

在曲线图上方显示的粗线表示当前值大于 Graph Max （图表上限）。

##### 6.Moving Avg

移动平均。定义在多长的采样期间移动平均曲线；值越大，曲线越平滑。

在设为零时，将不显示任何平均曲线。

### 2.3 Advanced Profiler

您可以通过单击 Show Live Data 按钮![live](image/Typora/live.png)来在捕获的同时查看相应信息，或者通过在坐标图中拖动 Performance Monitor 的时间光标来查看特定时间点的信息。

可按下 **Alt+逗点** (,) 来向左移动一个音频帧，或按下 **Alt+句点** (.) 来向右移动一个音频帧。

#### 1.Voices Graph

结构中每个元素之间的关系用线条来表示，并且包含元素信息。例如：

1. 对象或总线下方带有*<u>名称的蓝色方框</u>*表示其应用的效果器。

2. 事件或对象名称旁边带有*<u>乘号和数字的灰色方框</u>*表示实例的数量。

3. 对象或总线名称下方带<u>*有数字和图标的浅灰色方框*</u>表示相对音量电平（dB），低通滤波器值（0-100）和高通滤波器值（0-100）。

4. 鼠标指向一个对象时，将高亮其整个路径：包括父对象、子对象以及连接线。音频对象和 Auxiliary Bus（辅助总线）之间由虚线连接。

   > ​	**如果虚线是红色的...**
   >
   > ​	红色虚线表示延迟。在音量输出旁边，会以毫秒为单位显示延迟量。
   >
   > ![img](https://www.audiokinetic.com/images/2022.1.4_8200/?source=Help&id=Images/latency_line.png)
   >
   > ​	如果 Auxiliary Bus 在当前音频帧中正在进行混音，那么后续到达的辅助发送信号就会出现延迟。Wwise 必须等待下一个音频帧，才能将新的辅助发送信号进行混音。为了避免这种情况，请勿向更低层级的Auxiliary Bus 进行辅助发送。

5. 连接线上的浅灰色方框显示该通路的相对音量、低通滤波器和高通滤波器参数。

6. 图中一组元素后面的深灰色背景方框表示游戏对象。左上角的浅灰色方框显示了游戏对象的名称。

7. 如果名称显示为灰色，且旁边显示标有 **Virtual** 的浅灰色矩形，表示它是虚声部。

![img](https://www.audiokinetic.com/images/2022.1.4_8200/?source=Help&id=Images/AP_Effect_Graph_Example1.png)

Voice Graph 选项卡中的数据仅在各个捕获周期收集一次。但是，您可以使用 Performance/Voice Monitor 中的时间光标来浏览不同时间段的数据。

凡是列有游戏对象的地方（如 [“Capture Log”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=game_profiler_capture_log) 中或 Advanced Profiler 的 [“Voices（语音）”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=voices) 和 [“Voices Graph 选项卡”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=voices_graph) 选项卡中），都提供对象专有快捷菜单，包含以下选项：

- **Mute Game Object**：将该游戏对象关联的所有音频对象静音。
- **Unmute Game Object**：为该游戏对象关联的所有音频对象取消静音。
- **Solo Game Object**：间接地将其他游戏对象关联的所有音频对象静音。
- **Unsolo Game Object**：为其他游戏对象关联的所有音频对象取消静音。
- **Search Game Object in Voice Graph**：打开 Advanced Profiler 的 [“Voices Graph 选项卡”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=voices_graph) 选项卡，将其 Filter 设置为指定的游戏对象。

#### 2.Voices

显示有关声音引擎在任一时刻管理的各个声部或播放实例的信息。

为了使振动对象显示在 Voices 选项卡中，您需要在电脑上安装适当的振动设备驱动程序。

**Game Object** 、 **Target** 、 **Wwise Object** 和 **Bus** 列均显示相应的 Mute 和 Solo 按钮（如 [“Busses”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=busses) 选项卡文档中所述）。但请注意，游戏对象将对 Wwise 对象产生特殊的影响。如果游戏对象被静音，则其关联的所有 Wwise 对象都将会静音。如果游戏对象被独奏，那么只有其相关的 Wwise 对象可能会播放；这是由它们自身的 Mute 和 Solo 设置决定的。

#### 3.Busses

#### 4.Sends

#### 5.Audio Devices

#### 6.Emitter-Listener Associations

#### 7.Memory

#### 8.Streams

#### 9.Streaming Devices

#### 10.CPU

#### 11.Obs/Occ

#### 12.Listeners

#### 13.SoundBanks

#### 14.Loaded  Media

#### 15.Prepared Events

#### 16.Prepared Game Sync

#### 17.API

### 2.4 Voice Inspector

### 2.5 Audio Object 3D Viewer

### 2.6 Audio Object List

### 2.7 Audio Object Metadaa

### 2.8 Audio Object Meter

### 2.9 Voice Monitor

### 2.10 Voice Explorer

### 2.11 Game Object 3D Viewer

### 2.12 Game Sync Monitor

### 2.13 Profiler Statistics视图

## 3.Soundbank Layout

Wwise 中有两类 Bank：

- **Initialization (Init) bank** -- 初始化库。一种特殊库，其中包含有关工程的所有通用信息，包括有关总线层级结构、状态、切换开关和 RTPC 的信息。如果条件具备，则它还可能包括音频设备 ShareSet。

  在每次 Wwise 生成 SoundBank 都会自动创建 Initialization Bank。Initialization Bank 通常在游戏开始时加载一次，以便在游戏期间能轻松地获取工程的所有通用信息。它必须是在启动游戏时最先加载的声音包；否则其它声音包（内容）可能会无法加载。Initialization Bank 的文件名为“Init.bnk”。

- **SoundBank** —— 此文件中同时包含事件数据、声音、音乐和振动结构数据或音频文件。与 Initialization Bank 不同，SoundBank 一般在游戏的不同时间加载和卸载，以提高平台内存的利用率。

### 3.1 Soundbank Manager

#### 1.Soundbank Settings

- Override Project Soundbank Settings.不沿用工程 SoundBank 设置，用来为 SoundBank 创建自定义设置。

- Allow SoundBanks to exceed max size.决定是否允许生成超出大小上限的 SoundBank。

- Use SoundBank names.

  决定在进行以下任务时，使用 SoundBank 名称还是 ID：

  - 命名生成的 SoundBank 文件（例如“Init.bnk”或“1355168291.bnk”）

  - 当一个 SoundBank 引用了另一个 SoundBank 中的媒体时，识别目标文件。

  - 根据游戏中使用的低级 I/O 实现，该选项可能会影响如何调用 AK::SoundBank::LoadBank()。有关详细信息，请参阅 Wwise SDK 文档中的 [使用 SoundBank 名称 ](https://www.audiokinetic.com/library/edge/?source=SDK&id=soundengine__banks__general.html)。路径为“声音引擎集成纵览 > 将 Wwise 元素集成到游戏中 > 集成 SoundBank > 集成详情 – SoundBank > 一般信息”。

  Default value: true。

- Generate header file.

  - 创建包含名称与 ID 映射关系的头文件。头文件用于映射 Event、State、Switch 和 Game Parameter。

  - 该文件将保存在头文件路径中，其命名为“Wwise_IDs.h”。

  - 如果程序员倾向于在代码中使用 ID，则必须生成头文件。

  Default value: false

- Generate Bank Content TXT Files.

  - 创建文件，其中列出各个 SoundBank 内容。内容文件包括有关 Event、总线、状态和切换开关的信息，以及流播放文件和内存媒体文件的完整列表。

  - 您可以通过关联列表指定 SoundBank 内容文件的文本文件类型。

    - ANSI
    - Unicode

    如果文件路径、对象名称或备注中包含非 ANSI 字符，那么应使用 Unicode格式。

- Generate Metadata File.

  为所有音频包生成元数据文件。为每个指定类型（XML 或 JSON）创建一个文件 (SoundbanksInfo.xml/SoundbanksInfo.json)，并在其中列出所有生成的 SoundBank 的信息。此文件包含 SoundBank 名称、路径、语言、所含 Event 和文件（AMB、WAV 和 WEM）以及为 Metadata Options 指定的各项详细信息。此选项还会创建一个 PluginInfo 文件（XML 或 JSON），并在其中列出工程中所用全部插件的信息。

  | 在生成 SoundBank 时可能会收到 Invalid info file 错误         |
  | :----------------------------------------------------------- |
  | 对于流播放文件，Wwise 默认使用 SoundbanksInfo.xml 来完成 SoundBank 生成后步骤。假如无法找到该文件，系统会发出错误消息。因此，请一定要启用 Generate Metadata File 选项。同时，还要启用 Generate XML Metadata 选项。 |

- Generate Per Bank Metadata File.

  为每个音频包生成元数据文件。为每个指定类型（XML 或 JSON）和单独生成的 SoundBank 创建文件，并在其中列出相关信息。这些文件包含 SoundBank 名称、路径、语言、所含 Event 和文件（AMB、WAV 和 WEM）以及为 Metadata Options 选项指定的各项详细信息。

- Generate XML Metadata.

  - 生成 XML 元数据。无论启用了哪个 Generate Metadata File 选项，都可在此指定创建 XML 版本。若未启用任何 Generate Metadata File 选项，则将禁用此选项。

  - 正如 Generate Metadata File 选项部分所述，Wwise 需要使用 SoundbanksInfo.xml 文件来处理流播放文件。因此，请务必启用此选项。

- Generate JSON Metadata.

  生成 JSON 元数据。无论启用了哪个 Generate Metadata File 选项，都可在此指定创建 JSON 版本。若未启用任何 Generate Metadata File 选项，则将禁用此选项。

- Metadata Options

  - Include GUID.包含 GUID。添加 Event 和 SoundBank 的全局唯一标识符。

  - Max Attenuation.

    最大衰减。决定是否在每个 Event 的元数据文件中包含最大衰减信息（详见 [Attenuation Editor](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=positioning_attenuation_editor) 中的 Max distance 属性）。只有 “Play Event” 和 “Post Event” Action 可以设为非零值。

    Default value: false

  - Estimated duration.

    将尝试计算 SoundBank 中每个 Event 的预期时长。对于各个事件，soundbanksinfo.xml 文件中将包含 DurationType、MinDuration 和 MaxDurations 属性。DurationType 可以是以下值：“OneShot”、“Infinite”、“Mixed”或“Unknown”。“OneShot”表示非循环声音；“Infinite”表示循环声音；“Mixed”表示声音可能无限循环（可能基于随机因素或切换开关）；“Unknown”表示 Wwise 无法确定时长。MinDuration 和 MaxDuration 属性表示事件的最短和最长时长。请注意，这些范围仅为估算值，根据运行时条件不同，可能并不完全准确。

    Default value: false

  - Header file path:

    保存 SoundBank 头文件的路径或位置。

    根据您使用的是工程位置还是当前用户的自定义位置，该路径或位置可能不同。

  - Browse.浏览。打开 Windows 资源管理器或 Mac Finder，以便更改 SoundBank 头文件的默认保存位置。

### 3.2 Soundbank Editor

此布局包含为工程创建、管理和生成 SoundBank 所需的全部视图，包括 SoundBank Manager、Object Tab Group、Project Explorer 和 Event Viewer。

为帮助您识别 SoundBank 中手动添加或弃用的工程元素，在 SoundBank Editor 的 Add 和 Game Sync 选项卡上，条目旁边会添加一个星号（*）。

- **Add** —— 仅显示实际添加到 SoundBank 中的事件、对象层次结构、工作单元和文件夹。自动添加到 SoundBank 中的相应子对象将只显示在 Edit选项卡中。在 Add选项卡中，用户还可以决定各个层级结构元素中那些素信息或媒体类型将包含在 SoundBank 中。

  - Event-事件。确定是否在 SoundBank 中包含与特定工程元素关联的 Event。
  - Structure-结构。确定是否在 SoundBank 中包含与特定工程元素关联的声音、音乐和振动结构。
  - Media-媒体。决定是否在 SoundBank 中包含与该工程元素相关联的媒体。

- **Game Sync** —— 您可以在该列表中调整 SoundBank 的内容，基于特定游戏同步器来启用或弃用特定事件、对象结构或媒体文件。

  例如，假设您为主要角色的脚步声使用了切换容器，且该切换容器包含木头、瓦片、毯子和水四个切换开关。水切换开关仅用于游戏中的一个区域，因此为了避免不必要地加载“水材质脚步”声，除了仅包含水材质的 SoundBank 外，您可以为其它材质域的 SoundBank 将水材质切换开关弃用。

- **Edit** —— 显示了将包含在 SoundBank 中的各个工程元素。该列表详细列出了被添加至 Add 选项卡的对象层级包含的各个元素。您可通过该详细列表来调整 SoundBank 中的内容，弃用特定事件、对象结构或媒体文件。

  显示 SoundBank 中媒体文件的详情，包括采样率、音频格式和文件大小。通过掌握这些附加信息，您可以轻松地微调各个文件的转码设置，以遵守特定平台的限制。要更改媒体文件的转码设置，只需右键点击列表中的条目并选择 Conversion Settings 即可。

  

  | Object（对象） | 对象。当前 SoundBank 中所有对象结构、事件和媒体文件的列表。点击列标题栏，可以按字母升序或降序来排序信息。 |
  | -------------- | ------------------------------------------------------------ |
  | 采样率         | 采样率。SoundBank 中的音频文件的采样率。采样率定义了每秒内对数字音频信号进行采样的次数，单位为赫兹 (Hz)。各个平台的采样率范围各不相同，最高 48,000 Hz。如果媒体文件尚未针对当前平台转码，则采样率信息将以蓝色显示；如果工程缺失原始媒体文件，则以红色显示。 |
  | Format         | SoundBank 中的音频文件的格式，例如 PCM、Vorbis 或 ADPCM。请参阅 [“关于音频格式”一节](https://www.audiokinetic.com/zh/library/2022.1.4_8200/?source=Help&id=creating_audio_conversion_settings_sharesets#about_audio_formats) 了解关于为平台选择音频文件格式的详细信息。如果媒体文件尚未针对当前平台转码，则格式信息将以蓝色显示；如果工程缺失原始媒体文件，则以红色显示。 |
  | Memory Size    | 内存大小。当 SoundBank 加载至内存中时，已转码的媒体文件将占用的平台内存量。**Unit**：字节如果选择流播放媒体文件，则内存大小将为零。 |
  | Decoded Size   | 解码后大小。在当前所选平台下，包含解码后 Opus 或 Vorbis 媒体的 SoundBank 所要使用的游戏内存量。若 SoundBank 不含 Opus 或 Vorbis 媒体文件，则此值与 Data Size 相同。*info_outline*备注如果 SoundBank 包含 Sound Voice 对象，则与 Data Size 列一样， 根据当前所选语言不同，Decoded Size 列也将更新。**Units**：字节 |
  | Prefetch Size  | 预取大小。当 SoundBank 加载至内存中时，流播放媒体文件的预取数据将占用多少平台内存量。预取数据将被加载至内存中，以确保流播放媒体文件没有延迟。**Unit**：字节 |
  | File Size      | 已转码媒体文件的总大小。该数字不代表将生成 SoundBank 中的数据量，因为还取决于是否流播放媒体文件及其预取状态。**Unit**：字节 |
  | ID             | 媒体文件的 9 位唯一识别号。                                  |

- **Detail** —— 显示与选定 SoundBank 中不同元素的大小相关的详细信息，以及报告可能缺失或已被替换的任何文件。

  | Types（类型）         | 类型。对应 SoundBank 中包含的各种对象类型，如 SFX、Voice 和 Music。如果 SoundBank 包含所有四种对象，则该列将显示“All”（全部）。 |
  | --------------------- | ------------------------------------------------------------ |
  | Languages（语言）     | 语言。当前工程中使用的语言列表。如果使用了多种语言，则这些语言将按字母顺序显示。如果 SoundBank 中不包含 Voice 对象，则该栏为空。 |
  | SFX（音效）           | 音效。当前 SoundBank 中声音 SFX 对象占用的内存量。**Units**：字节 |
  | Music                 | 当前 SoundBank 中音乐对象占用的内存量。**Units**：字节       |
  | Voice                 | 语音。特定语言的 Sound Voice 对象占用的内存量。**Units**：字节 |
  | Data Size（数据大小） | 数据大小。音效声和特定语言的 Sound Voice 对象占用的内存量。如果超出为该 SoundBank 设置的最大内存量，则当前大小将显示为红色。**Units**：字节*info_outline*备注即使超出您指定的最大内存量，仍然可以成功生成 SoundBank。 |
  | Free Space            | 剩余空间。SoundBank 中的剩余空间量。剩余空间是由最大内存量减去数据大小得到的。如果数据大小超出最大内存量，则剩余空间值将显示为红色。**Units**：字节 |
  | Missing Files         | 缺失文件。该语言缺失的媒体文件（音效声和 Sound Voice ）数量。 |
  | Files Replaced        | 已替换的文件。该语言中缺失且被参考语言的音频文件所取代的 Sound Voice 音频文件数量。 |
  | Memory Size           | 内存大小。要加载至内存的 SoundBank 数据所占用的空间量。**Units**：字节 |
  | Prefetch Size         | 预取大小。流播放媒体文件的预取数据占用的空间量。当加载 SoundBank 时会将该信息加载至内存，以确保流播放媒体文件没有延迟。**Units**：字节 |
  | File Size             | 文件大小。生成的 SoundBank 文件的总大小。**Units**：字节     |



## 4.Mixer Layout

### 4.1 Mixing desk

界面元素：

- ![choose](image/Typora/choose.png)：选择混音会话。

- Editing States：

  编辑状态。与混音会话中的对象相关联的各个 State Group 的名称和相应 State。

  当前所选状态会更改 Mixing Desk 中正在编辑的 State。

- Follow State：

  跟随状态。决定混音会话中的 State 是否跟随游戏中的 State。

  若启用，则混音会话跟随游戏中的 State 变化。若禁用，则混音会话中的 State 将不跟随游戏中的 State 变化。

- Push State:

  推送状态。决定混音会话中的状态变化是否会同时更改游戏中的 State。

  若启用，则每次混音会话中的编辑状态变化时，游戏中的 State 也会变化。若禁用，则更改混音会话中的 State 时不会更改游戏中的 State。

- Copy Custom States...:

  打开 Copy Custom States 对话框，您可以在此将属性设置从一个自定义状态复制到另一个自定义状态。

- Activity

  活动。使用一系列图标指示总线或对象内是否存在任何音频活动。

  您可以使用以下图标来监视各个对象或总线的活动：

  - **![playback](image/Typora/playback.png)Voice Playback** —— 指示正在播放对象。对于总线，它指示正在播放的声部信号正在通过总线。
  - **![ducking](image/Typora/ducking.png)Ducking** —— 指示是否正在闪避总线。
  - **![bypass](image/Typora/bypass.png)Effect Bypass**（效果器旁通)：指示插入的 Effect 是否被旁通。

- ![mute](image/Typora/mute.png)控制对象的 Mute（静音）和 Solo（单独播放)状态，也会显示该对象是否处于被动静音和 Solo 状态。

### 4.2 Mixing Desk Settings

| 界面元素                                                | 描述                                                         |
| :------------------------------------------------------ | :----------------------------------------------------------- |
| 监视                                                    | /                                                            |
| Activity                                                | 决定 Mixing Desk 中是否显示监视信息。在 Mixing Desk 中可对以下活动进行监视：声部播放总线闪避效果器旁通 |
| Mute/Solo                                               | 确定静音或 Solo 功能是否在 Mixing Desk 中可用。              |
| Meter                                                   | 确定电平表是否在 Mixing Desk 中可用。                        |
| Bus                                                     |                                                              |
| Output Bus                                              | 输出总线。决定是否在 Mixing Desk 中显示用于输出对象的 Bus 的名称。 |
| Is Ducking                                              | 正在闪避。在当前 Bus 输出音频时，决定是否在 Mixing Desk 中显示正在闪避的 Bus 名称。 |
| Output Bus Volume (horizontal slider)                   | 输出总线音量(横向滑杆)。决定是否在调音台通道条中显示音量滑杆。此滑杆可用来控制对象 Output Bus 的音量电平。 |
| Output Bus Low-pass Filter (horizontal slider)          | 输出总线低通滤波器(横向滑杆)。决定是否在调音台通道条中显示 Low-pass filter 滑杆。此滑杆可用来控制对象 Output Bus 的滤波器值。 |
| Output Bus High-pass Filter (horizontal slider)         | 输出总线高通滤波器(横向滑杆)。决定是否在调音台通道条中显示 High-pass filter 滑杆。此滑杆可用来控制对象 Output Bus 的滤波器值。 |
| Game-Defined Auxiliary Sends Volume (horizontal slider) | 游戏定义的辅助发送音量(横向滑杆)。决定是否在 Mixing Desk 中显示指定编号的 Auxiliary Send Bus。 |
| User-Defined Auxiliary Send 0-3                         | 用户定义的辅助发送 0-3。决定是否在 Mixing Desk 中显示指定编号的 Auxiliary Send Bus。 |
| User-Defined Auxiliary Send Volume 0-3                  | 用户定义的辅助发送音量 0-3。决定是否在 Mixing Desk 中显示指定编号的 Auxiliary Send Volume。 |
| 定位                                                    | /                                                            |
| Attenuation                                             | 衰减。确定是否在 Mixing Desk 中显示衰减共享集信息。          |
| Center %                                                | 中置百分比。确定是否在 Mixing Desk 中显示Center % 信息。     |
| Effects                                                 |                                                              |
| Effect 0-3                                              | 效果器 0-3。决定是否在 Mixing Desk 中显示 Effect 插槽 0-3 中的 Effect 信息。 |
| States                                                  | /                                                            |
| State Group                                             | 状态组。决定是否在 Mixing Desk 中显示各个对象的 State Group 信息。 |
| State Properties                                        | 状态属性。决定是否在 Mixing Desk 中显示添加至各个对象的 State Properties。 |
| Properties                                              | /                                                            |
| RTPC                                                    | 决定是否在 Mixing Desk 中显示 RTPC 控件。                    |
| Bus Volume (horizontal slider)                          | 总线音量(横向滑杆)。决定是否在 Mixing Desk 中显示 Bus Volume 信息。 |
| Voice Volume (horizontal slider)                        | 音量水平滑杆。确定是否在混音器工具栏中显示水平音量滑杆。此滑杆可让您控制各个对象或总线的音量电平。 |
| Voice Pitch (horizontal slider)                         | 声部音高(横向滑杆)。决定是否在调音台通道条中显示横向 Pitch 滑杆。此滑杆可让您控制各个对象或总线的音高值。 |
| Voice Low-pass filter (horizontal slider)               | 声部低通滤波器(横向滑杆)。决定是否在调音台通道条中显示横向 Low-pass filter 滑杆。此滑杆可让您控制各个对象或总线的 LPF 值（有关详细信息，请参见[“Wwise 中 LPF 和 HPF 值对应的截止频率”一节](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=associating_low_pass_filter_values_with_their_corresponding_cutoff_frequencies#wwise_lpf_value_cutoff_frequencies)）。 |
| Voice High-pass filter (horizontal slider)              | 声部高通滤波器(横向滑杆)。决定是否在调音台通道条中显示横向 High-pass filter 滑杆。此滑杆可让您控制各个对象或总线的 HPF 值。 |
| Make-up Gain (vertical fader)                           | 补偿增益(纵向推子)。决定是否在 Mixing Desk 中显示对象的纵向 Make-up Gain 推子。 |
| Bus Volume (vertical fader)                             | 音量纵向推子。确定是否在混音器工具栏中显示垂直音量推子。此推子可用来控制各个 Bus 的音量电平。 |
| Voice Pitch (vertical fader)                            | 音高纵向推子。确定是否在混音器工具栏中显示垂直音高推子。此推子可让您控制各个对象或总线的音高值。 |
| Voice Low-pass filter (vertical fader)                  | 低通滤波器纵向推子。确定是否在混音器工具栏中显示垂直低通滤波器推子。此推子可让您控制各个对象或总线的 LPF 值（有关详细信息，请参见[“Wwise 中 LPF 和 HPF 值对应的截止频率”一节](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=associating_low_pass_filter_values_with_their_corresponding_cutoff_frequencies#wwise_lpf_value_cutoff_frequencies)）。 |
| Voice High-pass filter (vertical fader)                 | 高通滤波器纵向滑杆。确定是否在混音器工具栏中显示垂直高通滤波器推子。此推子可让您控制各个对象或总线的 HPF 值。 |
| Make-up Gain (vertical fader)                           | 补偿增益。确定是否在 Mixing Desk 中显示 Make-up Gain 信息。  |
| 其它                                                    | /                                                            |
| Object Name                                             | 对象名。确定是否在各个混音器工具栏的底部显示对象或总线的名称。 |
| Vertical fader height                                   | 纵向推子高度。决定 Mixing Desk 中纵向推子的高度（像素）。    |
| Meter height                                            | 电平表高度。决定 Mixing Desk 中电平表的高度（像素）。        |

### 4.3 Soundcaster

利用 Soundcaster 控件，可快速更改、编辑和应用对象、Event、State、Switch、Trigger 和 RTPC 的属性，然后在 Wwise 和游戏中试听结果。

#### 1.Soundcaster控件

- Original

  确定播放的是原始音频源还是转码音频源。

  如果勾选该选项，则对象会以原始的未转码形式播放。如果未勾选该选项，则播放对象的转码版本（如果存在的话）。

  在针对特定平台对某个音频文件做转码时，转码的目的是要满足该平台的特定硬件要求。因此，如果 Wwise 设计工具的运行平台不支持该文件类型，可能会无法播放这些转码文件。

- Inc,Only

  当勾选该选项时，只有所选平台中包含的对象才可在 Soundcaster 中播放。

- ![stop](image/Typora/stop.png)Stop

  停止所有 Soundcaster 模块的播放。

- ![pause](image/Typora/pause.png)Pause

  暂停所有 Soundcaster 模块的播放。再次点击此按钮将恢复播放。

- ![Delay](image/Typora/Delay.png)Delay

  延迟。指示是否对 Event Action、Random Container 或 Sequence Container 应用了延迟。如果存在延迟，则此图标在延迟期间会变成蓝色。

- ![Fade](image/Typora/Fade.png)Fade

  淡变。指示是否对 Event Action、Random Container 或 Sequence Container 应用了淡变。如果存在淡变，则此图标在淡变期间会变成蓝色。

- Reset Selector菜单

  打开 Reset Selector 菜单，其中有以下选项：

  - **Reset All**（重置所有），停止播放并将所有属性和设置恢复至原始值。
  - **Reset All Random and Sequence Containers**（重置所有随机容器和序列容器），让序列容器返回播放列表的开头，或重置随机容器的随机播放属性。
  - **Reset All Game Parameters**（重置所有Game Parameter），将所有的 Game Parameter 恢复至原始设置。
  - **Reset All Set Mute**（重置所有设置的静音），清除对象的所有 Mute（静音）动作。
  - **Reset All Set Voice Pitch**（重置所有音高设置），清除针对对象的所有 Set Voice Pitch 动作。
  - **Reset All Set Voice Volume**（重置所有音量设置），清除针对对象的所有 Set Voice Volume 动作。
  - **Reset All Set Bus Volume**（重置所有的总线音量设置），清除针对对象的所有 Set Bus Volume 动作。
  - **Reset All Set Voice Low-pass Filter**（重置所有的声部低通滤波器设置），清除针对对象的所有 Low-Pass Filter 动作。
  - **Reset All Bypass Effect**（重置所有旁通效果器），重置针对对象的所有 Bypass Effect（旁通效果器）动作。
  - **Reset All States**（重置所有状态），清除针对对象的所有 **Set State**（设置状态）动作。
  - **Reset All Switches**（重置所有切换开关），清除针对对象的 Switch（切换开关）动作。
  - **Reset All Music Tracks Force Usage**（重置所有强制音乐轨），在 Transport Control（播放控制）中清除强制播放的音乐轨。
  - **Reset All Source Editor Play Cursors**（重置所有源文件的播放光标），清除针对对象的所有 Source Editor Play Cursor 操作。
  - **Reset Position** 可以将 Attenuation Preview（衰减预览）控件内的听者位置重置为其默认位置。

- ![voice volume](image/Typora/voice volume.png)Set Voice Volume

  指示是否对对象应用了 Voice Volume 动作。若应用了 Voice Volume 动作，则此图标在播放期间将变为蓝色，并在 Soundcaster 中一直显示为蓝色，直至重置音量。

- ![Pitch](image/Typora/Pitch.png)Set Pitch

  设置音高。指示是否对对象应用了 Pitch 动作。当应用了 Pitch 动作时，此图标在播放期间变成蓝色，并在重置音高之前在 Soundcaster 中一直保持蓝色。

- ![caster_mute](image/Typora/caster_mute.png)Mute

  静音。指示是否对对象应用了 Mute 动作。当应用了 Mute 动作时，此图标在播放期间变成蓝色，并在重置 Mute 之前在 Soundcaster 中一直保持蓝色。

- ![Lowpassfilter](image/Typora/Lowpassfilter.png)Set Low Passfilter

  声部低通滤波器。基于指定频率值针对高频进行衰减的递归滤波器。

  此滤波器的单位表示低通滤波的百分比，其中 0 表示无低通滤波（信号不受影响）而 100 表示最大衰减。

- ![nable_bypass](image/Typora/nable_bypass.png)Enabe Bypass

#### 2.Soundcaster模块 

内容比较常规，不再赘述 。

## 5.Schematic View Layout

会以图形形式显示工程中总线、振动、音乐和声音对象的层次结构。在默认情况下，Schematic View 显示整个工程的层级结构和不同通路连接，但您可以使用 Search 字段筛选视图，只显示某对象的层级结构。

| 连接类型                           | 描述                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| ![line_1](image/Typora/line_1.png) | 定义两个节点间的层级结构关系。左侧节点是右侧节点（子节点）的父节点。 |
| ![line_2](image/Typora/line_2.png) | 定义两个节点之间的通路关系。左侧节点为 Audio Bus（音频总线），右侧节点为音频对象。 |
| ![line_3](image/Typora/line_3.png) | 定义两个节点之间的通路关系，其中右侧节点不沿用父对象的通路。左侧节点是总线。右侧节点为音频对象。 |
| ![line_4](image/Typora/line_4.png) | 定义两个节点之间的辅助发送通路关系，其中右侧节点（音频对象）将音频发送到左侧 Auxiliary Bus（辅助总线）。 |
| ![add](image/Typora/add.png)       | 点击将展开示意图，并显示对象的所有子项。                     |

| （图标栏）                                              |                                                              |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| ![master_bus](image/Typora/master_bus.png)（总线)       | 显示对象通路所使用的总线。                                   |
| ![auxiliary_bus](image/Typora/auxiliary_bus.png)（总线) | 显示该对象的用户定义型辅助发送的列表。                       |
| ![effect_1](image/Typora/effect_1.png)（效果器)         | 效果器。显示对象所用 Conversion Settings（转码设置）共享集的名称。 |
| ![effect_2](image/Typora/effect_2.png)（效果器)         | 显示已作用于对象的任何效果器。                               |
| ![position](image/Typora/position.png)（定位类型)       | 显示应用于对象的定位类型（No Positioning、3D Emitter、3D Emitter with Automation、3D Listener with Automation）。 |
| ![rtpc](image/Typora/rtpc.png)（游戏参数)               | 游戏参数。显示通过 RTPC 影响对象的 Game Parameter。          |
| ![stateGroup](image/Typora/stateGroup.png)（状态组)     | 显示对象所属的 State Group。                                 |
| ![setting](image/Typora/setting.png)（高级设置)         | 显示对某一对象的高级设置所作的更改（如播放数限制或音量阈值）。 |

### Schematic View Settings

Schematic View Settings（对象网络视图设置）对话框可让您筛选与 Schematic View 中显示的属性值相关的信息量。

| 界面元素                        | 描述                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| General                         |                                                              |
| Icon Strip                      | 图标栏。显示一行图标，这些图标表示对对象属性所作的更改。     |
| Mute/Solo                       | 静音／ Solo。显示对象的 Mute 和 Solo 按钮。                  |
| Bus                             | 总线。显示对象的总线通路。                                   |
| Conversion Settings（转码设置） | 转码设置。显示应用于对象的 Conversion Settings 共享集（如存在）。 |
| Effect（效果器）                | 效果器。显示应用于对象的 Effect（如存在）。                  |
| Game Parameters                 | 游戏参数。显示通过 RTPC 影响对象的 Game Parameter。          |
| State Group                     | 显示会影响对象的 State Group（状态组）。                     |
| Advanced Settings               | 高级设置。显示已为对象指定的高级设置。                       |
| 属性                            |                                                              |
| Volume                          | 显示各个工程对象的音量设置。                                 |
| Pitch                           | 显示各个工程对象的音高设置。                                 |
| Low-Pass                        | 显示各个工程对象的声部低通滤波器设置。                       |
| High-Pass                       | 显示各个工程对象的声部高通滤波器设置。                       |

## 6.Interactive Music Layout

内容在**<u>*1.1.1 Elements Type*</u>**中**<u>*1.Folder Type*</u>**中 的**<u>*8.Music Switch Container*</u>**讲到，不再赘述。

## 7.Dialogue Event Editor

可以排列和组合状态或切换开关来创建不同“路径”，用于在游戏中动态选择要播放哪个单词或词组。在该编辑器中，可添加并排列想要包含在 Event 中的 State Group或 Switch Group。随后可以选择状态或切换开关的不同组合来定义路径，每条路径可以与一个特定声部或声音对象相关联。在运行时，声音引擎会根据预定义路径来匹配触发的 State 或 Switch，以便确定要播放哪个声音。您还可以使用权重值来决定特定路径被选择的可能性高低。

> 内容多与Music Switch Container重合，所以只会选择介绍，不多赘述重复内容。

**Probability：**

- 概率。使用该路径播放音频的可能性。
- 最终的播放概率是所选路径概率和对白事件概率的组合。
- 单位：%
- Default value: 100
- Range: 0 to 100

## 8.File package

File Packager是独立的应用程序，可以将 Wwise 生成的 SoundBank、零散媒体以及流播放文件分组，并放到若干个文件包中。

文件包是将文件系统抽象化之后的独立单元，加载文件包会在内存中创建查找表，它会为 Wwise 指明媒体在 PCK 文件中的位置。您可以通过使用文件包来规避平台文件系统的一些限制，包括文件名长度和文件数量限制。文件包还可帮您更好地管理多语言版本，以及发布后可供下载的内容（DLC）。

通过将 SoundBanksInfo.xml 文件导入 File Packager 来进行检索，可以获取Wwise 工程的 SoundBank 和流文件的所有信息。

> **生成 SoundbanksInfo.xml 文件**
>
> 在 Project Settings 或 SoundBanks Settings 的 SoundBanks 选项卡下，须始终启用 Generate Metadata Files 选项。同时，还要启用 Generate XML Metadata 选项。利用这些设置，Wwise 可在每次成功生成 SoundBank 时自动生成 SoundbanksInfo.xml 文件。

默认情况下，所有文件都会添加到 default.pck 文件包中，该文件包将生成至各平台 SoundBank 路径的根目录下。也可以创建新的文件包，手动将文件添加至其中并修改文件包的储存位置。

可以使用 File Packager 手动创建文件包，或者在 SoundBank 生成时使用命令行运行 File Packager 来自动创建。该命令行可以在工程级别设置，也可以在 SoundBank 的 User Setting 中自定义。

### 8.1 使用File Packager

#### 1.将SoundBank信息导入File Packager

1. 点击 SoundBanks Info file（ SoundBank 信息文件）旁边的 **Browse**（浏览）按钮

   Windows 系统的 Open对话框将会打开。

2. 浏览至 SoundBanksInfo.xml 文件位置，选择该文件并点击 **Open**。

   所有 SoundBank 和流文件将被加载到 Files to package 视图中。

#### 2.加载file packager工程文件

选择 .wfpproj 文件并点击 **Open**。

工程信息将被加载到 File Packager 中。

### 8.2 File packager窗口



| 菜单                                                | 描述                                                         |
| :-------------------------------------------------- | :----------------------------------------------------------- |
| Tools > Create and assign packages for all language | 工具 > 为所有语言创建和指派包。为语言创建一个新包。针对创建的各个包，将流播放文件、零散媒体和 SoundBank 指派给相应语音的包。如果该语言的包已经存在，则不创建任何包并且不会修改现有包。 |
| Tools > Freeze selected packages                    | 工具 > 冻结所选包。获取所选包并确保所有内容都非常明确（的确在手动添加的文件中）并且没有隐含的内容。该过程将确保没有指派规则引用所选包。做好的包内容将不会更改。在您可为其准备可下载内容的游戏版本之后，可使用该功能。它将确保传输的动态创建包已冻结。 |



| **Files to package**                                         | /                                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Filename                                                     | 文件名。可添加到若干个包的 SoundBank、流播放文件或零散媒体的名称。 |
| File type                                                    | 文件类型：SoundBank、流播放文件或零散媒体。                  |
| Language/SFX                                                 | 语言/音效。与该文件关联的语言。如果文件与任何语言都不关联，则该文件会被认为是音效。 |
| File size                                                    | 文件的大小。                                                 |
| Packages                                                     | 已经包含了文件的包。                                         |
| ![add to current packkage](image/Typora/add to current packkage.png) | 将所选文件添加到 Package Manager 中所选的包中去              |



| **Packages**                                                 | /                                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Name                                                         | 名称。文件包的名称。该名称是输出目录的相对路径，可包含子文件夹，例如“my_folder/package1.pck”。 |
| Added                                                        | 手动添加到包里的文件数量。                                   |
| Resulting                                                    | 包中包含的结果文件总数。                                     |
| Missing                                                      | 包中缺失的文件数量。如果文件是手动添加到包中的并且之后从原始 Wwise 工程中删除掉了的话，则这些文件可能会缺失。 |
| Data size                                                    | 数据大小。列在包结果内容中的所有文件总和。                   |
| Header size                                                  | 包的近似文件头大小。它表示将包加载到声音引擎中的近似内存占用量。 |
| ![fp_add](image/Typora/fp_add.png)                           | 创建新的包                                                   |
| ![fp_remove](image/Typora/fp_remove.png)                     | 删除当前包                                                   |
| ![add to current packkage](image/Typora/add to current packkage.png) | 打开 File Order Editor（文件顺序编辑器），您可以在其中以特定顺序排列包内的文件。排列包内文件的顺序可优化磁盘寻道事件。 |
| Block Size                                                   | 块大小。包内数据对齐的字节数。指定什么数字将取决于平台 I/O 设备的要求。有关块大小和各个平台 I/O 设备限制的详细信息，请参阅 SDK 文档的 [Streaming > Low-Level I/O](https://www.audiokinetic.com/library/2021.1.12_7973/?source=SDK&id=streamingmanager__lowlevel.html)（流式 > 底层 I/O）部分。每个平台的默认值都是 1。默认值1表示无需对齐。 |



| **Default file assignment**   |                                                              |
| ----------------------------- | ------------------------------------------------------------ |
| Assign Language/SFX files to: | 将语言/音效文件指派给。此版块内的选项用来将与特定语音或音效关联的 SoundBank、零散媒体和流播放文件指派给特定包。仅在文件并非手动指派给包时才会使用默认指派。 |
| Language                      | 语言。SoundBank和流播放文件存在的语言/音效列表。             |
| SoundBanks                    | 音频包。与相应语言/音效关联的 SoundBank 自动指派到的包。仅在文件并非手动指派给包时才会使用默认指派。 |
| Streams                       | 流播放。与相应语言/音效关联的流播放文件自动指派到的包。仅在文件并非手动指派给包时才会使用默认指派。 |
| LooseMedia                    | 零散媒体。如果文件没有与相应语言/音效存在特定关联，则会自动指派到这种包中。仅在文件并非手动指派给包时才会使用默认指派。 |
| /                             | /                                                            |
| Assign remaining files to:    | 将剩余文件指派给。此版块内的选项用来将所有剩余 SoundBank、流播放文件、外部源和零散媒体连通到特定包。 |
| SoundBanks                    | 任何剩余的 SoundBank 会自动指派给这种包。                    |
| Streamed files                | 流播放文件。任何剩余流播放文件会自动指派给这种包。           |
| External source               | 外部源。任何剩余外部源都会自动指派给这种包。                 |
| Loose media                   | 零散媒体。任何其它剩余文件都会自动指派给这种包。             |



| **Package contents**                                       |                                                              |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| Manually added file                                        | 已手动添加到包的 SoundBank、流播放文件、外部源或零散媒体。   |
| Filename                                                   | SoundBank、流播放文件、零散媒体或外部源的名称。              |
| File type                                                  | 文件类型：SoundBank、流播放文件、外部源或零散文件。          |
| Language/SFX                                               | 语言/音效。与该文件关联的语言。如果文件与任何语言都不关联，则该文件会被认为是音效。 |
| File size                                                  | 文件的大小。                                                 |
| Inclusion Mode（仅在 Manually added file 选项卡中可用）    | 包含模式。决定SoundBank、其关联的流播放文件或所有零散媒体是否包含在包中。您可以选择以下选项之一：**SoundBank only（仅有SoundBank）****Streamed file（流播放文件）****SoundBank 和流播放文件****Loose media（零散媒体）****SoundBank 和零散媒体****Streamed files and loose media（流播放文件和零散媒体）****All files** |
| ![fp_remove](image/Typora/fp_remove.png)                   | 所有文件。从包中删除所选文件。                               |
|                                                            |                                                              |
| Resulting content（结果内容）                              | SoundBank、流播放文件或零散媒体都包含在包里。此选项卡标题栏中显示的数字表示包中包含的文件总数。此选项卡上的数字可能与 Manually added file 选项卡上的数字不同，因为当手动添加 SoundBank 时，会有多个流播放文件自动添加到相同的包的，而 SoundBank 可能包含对这些流播放文件的引用。 |
| Filename                                                   | 可添加到包中的SoundBank、零散媒体或流播放文件的名称。        |
| File type                                                  | 文件的类型：SoundBank、流播放文件、零散媒体或混合类型。      |
| Language/SFX                                               | 语言／音效。相应文件关联的语言。如果文件与任何语言都不关联，则该文件会被认为是音效。 |
| File size                                                  | 文件的大小。                                                 |
| ![fp_copytoclipboard](image/Typora/fp_copytoclipboard.png) | 将 Resulting Content 窗格中的信息复制到 Windows 剪贴板，以便您可以将信息粘贴到新的文件中。 |

### 8.3  File Order Editor视图

可以按特定顺序排列包内的文件。对包内的所有文件排序非常耗时而且没有必要。如果包内只有几个文件需要以特定顺序排列，则您可以排列这些文件，然后使用“Remaining files inserted here”（在此处插入剩余文件）占位符来放置包内的所有其它文件。

| **Package files（包文件）**                  |                                                              |
| -------------------------------------------- | ------------------------------------------------------------ |
| Filename                                     | 添加到包中的SoundBank、流播放文件或零散媒体的名称。          |
| File Type                                    | 文件的类型：SoundBank、流播放文件、零散媒体或混合类型。      |
| Language/SFX                                 | 语言／音效。相应文件关联的语言。如果文件与任何语言都不关联，则该文件会被认为是音效。 |
| File size                                    | 文件的大小。                                                 |
| ![addtolayout](image/Typora/addtolayout.png) | 将所选文件添加到有序列表内的下一个位置。您还可以从 Package file 视图中直接将文件拖到有序列表内的特定位置。 |



| **File order（文件顺序）**                                   |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Filename                                                     | 包内已按特定顺序排列的SoundBank、零散媒体或流播放文件的名称。“Remaining files inserted here”占位符表示包内尚未按具体顺序排列的所有文件。 |
| File Type                                                    | 文件的类型：SoundBank、流播放文件、零散媒体或混合类型。      |
| Language/SFX                                                 | 语言／音效。相应文件关联的语言。如果文件与任何语言都不关联，则该文件会被认为是音效。 |
| File size                                                    | 文件的大小。                                                 |
| ![fp_remove](image/Typora/fp_remove.png)                     | 从有序列表中删除所选文件。当某个文件从有序列表中显式删除时，该文件仍保留在包中，但已添加回“Remaining files inserted here”占位符。 |
| ![fp_removeallmissingfiles](image/Typora/fp_removeallmissingfiles.png) | 从有序列表中删除标记为“missing”的所有文件。                  |

## 词汇表

### 通用术语

- AAC

  Wwise 中可用于 Mac 和 iOS 平台上的一种感知编码音频压缩方法。据说在比特率相似的情况下，AAC 声音品质优于 MP3。压缩是可变的，取决于内容，品质设置可由“quality”（品质）滑杆控制。在 iOS 中，如果有可用的硬件辅助编解码器，则 AAC 会通过该硬件解码器解码。注意，iOS 硬件一次只可解码一个 AAC 声音。

- AC-3

  针对声音编码算法的一种压缩标准。有关详细信息，请参阅详述此[数字音频压缩标准](http://atsc.org/standard/a522012-digital-audio-compression-ac-3-e-ac-3-standard-12172012/)的文档。

- ADPCM (Adaptive Differential Pulse Code Modulation)

  自适应差分脉冲编码调制。一种音频文件编码方法，用于量化声音信号与根据该声音信号所做的预测之间的差异。ADPCM 量化步长是自适应的，与直接量化信号的 PCM 编码不同。基本上，ADPCM 以音质为代价，显著缩减了容量和 CPU 占用量。因此，它通常用于移动平台。

- Ambisonics

  Ambisonics 是一种环绕声技术，可以覆盖水平面声场以及听者上方和下方的区域。通过其 B-format 声场表示法，Ambisonics 能够独立于扬声器配置发挥效果。

- ADSR (Attack-Decay-Sustain-Release)

  包络的形状由该声音的起音时间（和 Wwise 中的起音曲线）、衰减时间、延音电平和释音时间来决定。

- 位深

  用于描述数字音频文件内每个采样点的位数。在 PCM 音频中，位深决定信号的最大可能动态范围。

- Bit Rate（比特率）

  每秒发送或接收的数据量（即位数）。比特率越高，处理的文件数据越多，通常分辨率也越高。

- Cent（音分）

  一种音高度量单位，相当于半音程的 1/100。八度音阶由 1200 个音分组成。

- DC Offset（直流偏置）

  直流表示声音波形的中心。偏置表示到波形中心 0.0 点的距离百分比。

- dBFS（满量程分贝）

  （分贝满量程）数字系统中的分贝振幅电平，这些系统具有最高的可用电平（如 PCM 编码）。

- Delay Line（延迟线）

  Delay Line 处理器单元可以将信号延迟若干采样点后输出。

- Depot（文档库）

  Perforce 版本控制服务器中的中央文件存储库。它包含提交到服务器的所有文件的所有版本。

- Dithering（抖动）

  在量化前添加到信号中的噪声，用于减少量化过程导致的失真和噪声调制。虽然噪声电平会略有上升，但频谱形状的抖动可让这种上升变得尽可能不明显。这种噪声没有失真的危害大，可以让低电平信号听得更清楚。

- Downmix（下混）

  将多声道源中的所有声道混合成具有较少声道的兼容版本（例如立体声或单声道）的过程。

- 干声路径

  从声音直接到总线的声部信号基本路径。

- Dry Signal（干声信号）

  完全由未经处理的原始信号构成的输出。

- Echo Density（回声密度）

  混响算法每秒产生的回声量。

- Frequency（频率）

  单位时间对应的循环次数。

- Gain（增益）

  信号功率或振动的变化。

- Harmonics（谐波）

  基频的倍数。举例来说，如果基频是 50 Hz，则第二谐波是 100 Hz，第三谐波是 150 Hz，以此类推。

- Headroom（裕量空间）

  放大器或音频设备中的正常工作电平与削波电平之间的电平差（单位：dB）。

- High-Pass Filter（高通滤波器）

  一种递归滤波器，用于衰减低于截止频率的频率。此滤波器的单位代表已应用的高通滤波效果器的百分比，其中 0 表示高通滤波（信号不受影响），100 表示最大衰减。

- HMD (Head Mounted Device)

  头戴显示器，为虚拟现实（VR）开发的设备，旨在基于玩家的头部运动来提供视觉与音频输入。

- 高动态范围音频（HDR 音频）

  一种利用自然声音所跨越的大动态范围电平值来进行混音设计的技术。HDR 也是一个实时系统，可以将宽泛的电平范围动态地映射至更适合于您的声音系统数字输出的范围。

- HRTF（头部相关传递函数）

  声音通过不同物理介质从空间某点传递到耳朵的方式。成对的 HRTF 可合成双耳声，使声音好像源自于空间中的特定某点。

- Inharmonicity（不和谐度）

  局部谐波音高与真正谐波之间的偏置比例（基频的整数倍）。

- Intensity Stereo（强度立体声）

  一种失真音频编码技术，它去除高频内容的相位校准信息，以将它合并到单声道信号中，同时保留强度信息来重构立体声信号。

- Latency（延迟）

  在计算机内部处理或生成音频信号时所固有的延迟。

- LFE

  低频效果声道。专门针对 10-120 Hz 低沉声音设计的音频声道的名称。LFE 声道是完全独立的声道，必须以 x.1 媒体文件形式导入 Wwise。

- Limiting（限幅）

  一种极端的压缩形式，其中输入/输出关系变得非常扁平（10:1 或更高）。它对信号电平做出硬性限制。

- Low-Pass Filter（低通滤波器）

  一种递归滤波器，用于衰减高于截止频率的频率。此滤波器的单位代表已经应用的低通滤波效果器的百分比，其中 0 表示低通滤波（信号不受影响），100 表示最大衰减。

- Modal density（模态密度）

  模态指音频信号在频域表示法中的峰值。当模拟大多数的声学空间时，增加模态密度可改善混响的逼真度。减低模态密度可能导致嗡嗡声。

- Noise Shaping（噪声整形）

  一种有助于将数字信号位深减少所造成的杂音最小化的技术。

- Nyquist 频率

  可准确采样的最高音频频率，相当于采样率的二分之一。Nyquist 采样定理表明，采样率必须至少是输入样本最高频率的两倍才能准确地重构原始信号。

- Opus

  一个低延迟的音频编解码器，针对语音和通用音频进行了优化，压缩时音质损失方面优于其他编解码器。详请参阅“[“转码技巧和窍门”一节](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=versions_tips_and_best_practices)”。

- Outro（结尾部分）

  声音的末尾部分，对应于开头部分。

- Passband（通带）

  基本无衰减地通过滤波器的频带。

- PCM（脉冲编码调制）

  一种音频文件编码方法，采用独特的二进制表示法即脉冲编码。通过测量两个编码点之间的值对它们进行量化；选择与最近点相关的值。

- PCM 帧（PCM frame）

  PCM 帧包括在给定时刻上所有声道的采样。每帧代表 1/采样率秒。

- Pitch

  声部音高。对象的播放速度。

- Prediction filter（预测滤波器）

  根据语音信号行为的可预测性对语音信号进行建模的方法，可用作有损音频压缩技术。

- Quantization（量化）

  将音频文件的值域细分为子域的过程，各个子域由指定值表示。

- Recursive Filter（递归滤波器）

  一种滤波器，使用先前计算的输出值和当前输入值计算最近的输出。也称为反馈滤波器。

- Ripple (in passband)（（通带内）波纹）

  通带内最大衰减和最小衰减之差。

- RMS power（RMS 功率）

  均方根（Root Mean Square）。它是一种测量信号平均振幅的方法，在多数情况下，信号功率比使用峰值振幅时更接近。获得此值的方法是将给定时间窗口期间的采样值的平方加起来，然后计算结果的平方根。

- Side-Chaining（旁链）

  监视音频信号的电平，并使用音频信息实时操控另一个音频信号。此技术用于自动控制不重要声音的音量，从而在最终混音中更多地强调更加重要的声音。

- Source Control（版本控制）

  用于管理各种文件（如 Wwise 工程, 工作单元和音频文件）的修改，其中每个文件的修订版本都标记了版本识别号、修改时间和修改人。大多数版本控制系统中都可以比较、还原和合并修订版本。

- 标准配置

  标准离散声道配置，如立体声，4.0，5.1，7.1 和 7.1.4。Ambisonics 不属于标准配置。

- Tab-delimited File（用制表符分割的文件）

  一种特殊类型的纯文本文件，其中信息通过制表符分割成列。

- Tap（延迟线分流）

  信号从 [Delay Line（延迟线）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_delay_line) 中被提取出来的分流点。

- VBAP (Vector Base Amplitude Panning)

  矢量基幅值平移。一种常用算法，用于多个扬声器配置下，对若干不同方向的虚拟声源进行精准 3D 定位。

- Volume

  音量。音频输出的振幅或强度级别。

- Vorbis

  一种感知编码方法，支持以各种比特率编码音频文件，同时保持非常好的感知声音品质。通过使用 Quality Factor（品质因数）或指定每声道的最大、最小和平均比特率来控制数据压缩效率和感知声音品质的平衡。*info_outline*备注Audiokinetic 的 Vorbis 专用版本针对所有平台进行了高度优化。

- WAVEFORMATEXTENSIBLE（扩展型波形文件格式定义结构）

  为具有两个以上声道的格式定义波形音频数据格式的结构。为了在输入时保留 WAV 文件的特定多声道配置，必须将 WAV 头文件中的声道信息定义为扩展型波形文件格式定义结构的声道掩码的一部分。

- 湿声路径

  从基本干声路径中分支后通过 Auxiliary Bus 的声部信号路径，一般会先应用 Effect（如 Reverb）和其他修改，然后再发送到总线。

- Wet Signal（湿声信号）

  完全由经过处理的声音构成的输出。

- World Builder（世界编辑器）

  用于创建游戏虚拟环境的应用程序。

### Wwise 专用术语

- Absolute Propert（绝对属性）

  这些属性通常在顶层父对象上定义，它们会自动下传到各个父项的子对象，例如定位和播放优先级。您可以不沿用顶层父属性，而在层级结构的不同层级定义这些属性。

- Actor-Mixer（角色混音器）

  角色混音器。若干个声音、振动对象、容器或角色混音器所构成的层级结构。您可以使用角色混音器来控制它下面的所有对象的属性。

- Additive Type Property（累积型属性）

  此类 Wwise 属性可以通过如 RTPC 和 State 这样的不同设置进行多次更改，各种属性偏置值经过累积得到最终数值。

- Audio Input（音频输入）

  此处的 Audio Input 是一种源插件的示例，允许通过 Wwise 管线发送游戏生成的音频内容，并由声音引擎处理。

- Audio Object

  音频对象。由音频缓冲区和 *Metadata* 构成，在满足所有条件的情况下可传给终端来进行渲染以获得空间化效果。

- Attenuation（衰减）

  当远离发声源时，声音、音乐或者振动对象的音量会相应地减小。

- Attenuation ShareSet （衰减共享集）

  基于音源与听者之间相对距离而进行的音量衰减设定。这些设定保存为共享集，可在工程间共享。

- Audio Bus（音频总线）

  一种可以在 Master Audio Bus 下添加的总线，允许和其他音频总线、辅助总线一起进行编组，以便有效地组织混音。您可重命名、移动和删除这些总线；还可以在总线上加效果器。

- Audio Source

  音频源。音频文件和声音对象之间的独立抽象层。音频源链接到导入到工程中的音频文件。音频源保持链接到您导入工程中的音频文件，以便您可以在随时引用它。

- Audio Object

  音频对象。Project Explorer 内 Audio 选项卡中 Master-Mixer Hierarchy、Actor-Mixer Hierarchy 或 Interactive Music Hierarchy 下的所有对象。

- Auto-ducking

  自动闪避。此动作将降低一个音频信号的音量电平，以便让另一并行音频信号更加突出。

- Auxiliary Bus

  一种特殊类型的音频总线，通常用于应用混响和延迟等效果器来模拟环境效果器或进行动态混音（*[旁链](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_side_chaining)*）。

- Auxiliary Send（辅助发送）

  用于将音频信号发送到辅助总线的音频信号通路技术。可根据声音对象来控制辅助发送，在使用 SDK API 时，也可根据游戏对象来控制辅助发送。

- Blend Container（混合容器）

  该组中的对象或容器将被同时播放。此容器中的对象可编组到 Blend Track（混合轨）中，然后使用 RTPC 将属性映射到游戏参数。在同一条Blend Track上的各个对象之间也可以基于游戏参数值来进行交叉淡变。

- Cache（缓存）

  一个工程文件夹，其中包含您正在开发的平台的所有转码结果文件。在默认情况下，此文件夹存储在本地，但您可以修改它的位置。多个用户不得同时访问缓存文件夹。

- Child Object（子对象）

  层级结构中位于上层对象或父对象内的对象。

- Clip

  代表音频源的音乐对象。片段按音乐轨进行排列。

- Compressor（压缩器）

  一种音频效果插件，通过削弱输入信号高出预定义阈值的任何部分来缩小信号的动态范围。

- Container（容器）

  若干个对象的组合，包括根据某中既定行为播放的声音、振动对象或容器。 Wwise 中有多种不同类型的 Container，包括 [Random Container （随机容器）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_random_container)、[Sequence Container （序列容器）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_sequence_container)、 [Switch Container（切换容器）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_switch_container)、 [Blend Container（混合容器）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_blend_container)、 [Music Switch Container（音乐切换容器）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_music_switch_container) 以及 [Music Playlist Container （音乐播放列表容器）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_music_playlist_container)。

- Control Surface Session（控制器会话）

  一种可保存的工程元素，用来定义 Wwise 如何与使用了 MIDI 或 Mackie 协议的外部控制器设备进行交互。连接了兼容设备并设置了绑定后，即可打开 Control Surface Session（控制器会话）并使用该设备来控制 Wwise 的功能。

- Conversion Settings（转码设置）

  音频文件参数的组合，包括采样率、音频格式和声道数量，它们定义各个平台的音频文件的总体品质、内存和 CPU 占用。

- Convolution Reverb（卷积混响）

  一种音频效果器，它利用 IR（冲激响应）模拟真实空间（例如音乐厅、建筑物、街道、汽车内饰、房间、户外、森林等）的声学特性。

- Cue（提示点）

  附加在音乐段落上的标记，用于指示关键点，例如进入点或退出点。

- Custom Cue（自定义提示点）

  附加到音乐段落上的用户创建的标记，用于指示关键同步点。Entry cue（入口提示点）和Exit cue（出口提示点）不属于自定义提示点。

- dB Scaling（分贝定标）

  在表示以分贝为单位的属性时，用于以对数刻度显示坐标图视图 X 轴的选项。

- Default Work Unit

  XML 文件，其中包含工程中与它们所针对的特定元素相关的所有信息。例如，Event 的 Default Work Unit.wwu 文件包含与 Event 相关的所有信息，State 的 Default Work Unit.wwu 文件包含与 State 相关的所有信息，以此类推。在创建工程时将创建默认工作单元。

- Definition File（定义文件）

  按 SoundBank 分类列出游戏中所有事件的文本文件。

- Delay

  一种音频效果器插件，通过将音频信号延迟指定时间段来添加回声。

- Dialogue Event（对白事件）

  使用一组规则或条件来触发游戏音频的方法，这些规则或条件通过与游戏内的可能条件相匹配的切换开关或状态值来表达。这些切换开关或状态值被安排在路径中，然后被指派到 Wwise 对象。当游戏调用 Dialogue Event 时，游戏先将其当前情形与 Dialogue Event 中定义的情形进行匹配，然后播放相应的对白片段。

- Diffraction

  对 Wwise 来说，衍射是内建的 Game Parameter（游戏参数） ，由 [Spatial Audio](https://www.audiokinetic.com/library/edge/?source=SDK&id=spatial__audio.html) 计算声音和 *[Shadow Boundary](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_shadow_boundary)*（阴影边界）之间的偏差角度，大致模拟声波遇到转角或其他障碍物时，衍射声路径的偏移角度。

- Effect Chain（效果器链）

  以特定顺序应用到对象或总线的一系列效果器。

- Effect Instance（效果器实例）

  可保存并应用到其他对象或总线上的一组自定义效果器属性。效果器实例属性还可跨对象共享。

- Effect ShareSet （效果共享集）

  效果共享集。音频效果插件设定，可用于增强游戏中音频效果。这些设定保存为共享集，可在工程间共享。

- Empty Event

  不包含任何动作或对象的事件。

- Environmental Effect（环境效果器）

  根据游戏对象在游戏几何图形中的位置更改该对象所生成声音的属性集合的效果器。

- Event（Action Event）（动作事件）

  触发游戏中音频的方法，使用一个或一系列动作（如播放、静音和暂停）来控制若干个 Wwise 对象。

- 扩展器（Expander）

  Wwise Expander 插件通过削弱输入信号低于预定义阈值的任何部分来扩大信号的动态范围。当信号很弱并低于阈值时，扩展器开始降低信号的增益。当信号等于或高于阈值时，不对信号应用增益衰减。

- External Source（外部源）

  在运行时将声音对象与音频文件相关联的源插件。通过它可管理原本需要大量开销的大量对白行。它有助于节省运行时的内存，简化在为游戏生成 DLC 内容时有时候需要更换音频文件的过程。

- File Manager（文件管理器）

  显示有关工程文件和原始导入源文件的信息，以及在适用的情况下管理许多版本控制插件功能的对话框。

- Flanger（镶边效果器）

  将两个相同信号混合在一起的音频效果，其中一个信号以微小的渐变时间量延迟，以产生扫频式梳状滤波效果。

- Flat View（扁平视图）

  停靠在布局中的视图。

- Focus

  百分比值，用于收缩由扩散值生成的虚拟发声体。对于 0% 焦点，虚发声体保持不变，但值越高，各个虚拟点距离源声道原始位置越近。

- Folder

  Actor-Mixer Hierarchy 中的上层结构，用于管理角色混音器、容器等其它结构。

- Game Object

  界面、触发器或声音等元素可附加到其上的游戏实体。

- Game Parameter（游戏参数）

  使用 RTPC 可映射到 Wwise 属性值的游戏参数，例如赛车游戏中的速度和 RPM。

- Game Sync

  一组 Wwise 元素，包括 State、切换开关、RTPC、触发器和变量，根据游戏中的条件调用它们，并相应地修改音频和振动。

- Game Unit（游戏计量单位）

  用于计算游戏几何构造的基本长度单位。例如， 秘密潜入游戏的 FPS 可使用节拍作为游戏计量单元，而太空征服游戏可使用光年。

- Guitar Distortion（吉他失真效果器）

  一种音频效果器，它更改音频波形的形状，引入原始信号中不存在的频率分量。Wwise 吉他失真效果器模拟常用失真“stomp boxes”的行为来获取典型的吉他失真声音。

- Harmonizer（和声效果器）

  将几个有音调的声部添加到输入信号的音频效果器。

- Imported folder（导入文件夹）

  隐藏的工程 .cache 文件夹，其中包含导入工程中的、经过特别导入转码过程的音频文件。

- Image Source（声源虚像）

  发声体相对于房间墙壁所成镜像的位置，在 Wwise Reflection 中根据[声源成像技术](http://interactiveacoustics.info/html/GA_IS.html)计算得出。

- 标志

  Wwise 界面中的特定图标，用于指示特定属性值的状态。例如，RTPC 标志显示属性值是否具有关联的 RTPC。

- Initialization Bank（初始化库）

  一种特殊类型的库，其中包含工程的所有常规信息（包括有关总线层级结构的信息）和有关状态、切换开关和 RTPC 的信息。每个工程只有一个初始化库，默认情况下它被命名为“Init.bnk”。初始化库必须是启动游戏时加载的第一个库。如果第一个加载的不是它，则后续的 SoundBank 可能会拒绝加载。

- Input Range（输入范围）

  可为属性输入的完整值域，与滑杆范围相反。

- Integrity Report（完好度报告）

  Wwise 中生成的一种报告，它显示工程中的错误或问题，并提供修复方案建议。

- Interactive Music

  一种音乐创作和编曲方法，用于创建响应于游戏内动作的、模块化的配乐。

- Invalid Events（无效事件）

  已从工程中删除但 SoundBank 中仍包含的 Event。

- IR（冲激响应）

  测量某个位置（例如音乐厅）真实声学特性所生成的音频文件。卷积混响效果中使用冲激响应来启用对输入信号应用特定位置的声学特性。

- Layout（布局）

  为方便完成特定任务或工作而组合在一起的一系列视图。

- Listener

  游戏中的虚拟话筒或振动传感器，帮助将声音指定到特定扬声器或特定马达的振动来模拟 3D 环境。

- Look-ahead time（预读时间）

  在流播放中，它指为声音引擎查找流播放数据所预留的时间。

- Master-Mixer Hierarchy

  工程层级结构顶层的总线或一系列总线，您可以根据游戏中的主要类别来编组许多不同的声音、音乐和振动结构。例如，您可以将所有语音或音乐结构组合在一条音频总线下，所有声音效果器组合在另一条音频总线下，以此类推。

- 该总线是主音频总线

  Master Audio Bus 位于嵌套 Work Unit 和 Virtual Folder 的顶层，层级结构中的所有其他子级 Audio Bus 都要设在它的下面。声音信号通过复杂的子总线结构之后，最终的音频输出由 Master Audio Bus 的设置及其应用的效果器决定。

- master secondary bus（主二路总线）

  Master Secondary Bus 是供二路输出（如游戏控制器）使用的总线，它位于嵌套 Work Unit 和 Virtual Folder 的顶层，层级结构中的所有其他子级 Audio Bus 都要设在它的下面。

- Matrix Reverb（矩阵混响器）

  专为游戏制作而优化的独特混响效果，它可以平衡品质和性能，包括实时编辑和 RTPC 映射功能。

- Metadata

  元数据。一组与 *[Audio Object](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_side_chaining)* 关联的属性，专门供终端或对象处理器用来生成空间化效果。典型的 Metadata 包括 3D Position、Azimuth、Elevation、Focus 和 Spread。

- Meter Effect（电平表效果器）

  在不修改信号的情况下测量信号电平的音频效果，另可将此电平作为游戏参数输出。该插件非常实用，可实现*[旁链](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_side_chaining)*，测量后的总线电平可通过 RTPC 驱动其它总线的音量。

- Meter (Peak)（电平表（峰值））

  显示各个声道的音频信号电平的一系列电平表。虽然音频和辅助总线显示输出信号，但动态效果（例如压缩器和限幅器）通常显示音频输入、音频输出和已应用的增益衰减。

- Mixing Desk

  一种灵活强大的调音控制台，它将各种总线和对象属性编组到单一视图中，用于实时优化游戏的音频混音。

- Mixing Session（混音会话）

  您选择的在调音台中使用的一组 Wwise 对象，可在任何时候保存和重新使用它们。

- 调制器包络

  在 MIDI 或常规音频播放中，使用预定义的 ADSR（包络）调整波形的一种技巧。

- 调制器 LFO

  在 MIDI 或常规音频播放中，使用 LFO（低频振荡器）调整波形或音频属性的一种技巧。

- 振动对象

  Wwise 中您为工程创建的单个振动素材的一种表示。通常，这些对象控制着主机的游戏控制器的振动。每个振动对象可以包含数个声源，这些声源决定了游戏中实际生成什么样的振动。

- Music Clip（音乐片段）

  音乐轨的基本组件，显示为矩形区域，表示一个 WAV 文件。

- Music Playlist Container （音乐播放列表容器）

  随机或按顺序播放的若干个音乐对象或容器的组合。

- Music Segment （音乐段落）

  一种多声轨音乐对象，它是 Interactive Music 层级结构的基本单元。

- Music Switch Container（音乐切换容器）

  根据调用的 Switch 或 State 播放的若干个音乐对象或容器的组合。

- Music Track （音乐轨）

  可以包含多个独立音乐片段音乐对象，并以波形形式显示它们，使您能够在音乐片段中以视觉方式进行调整。Wwise 中有多种不同类型的 Music Track，包括 [Random Music Track](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_random_musictrack)、 [Sequence Music Track](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_sequence_musictrack) 和 [Switch Music Track（切换开关音乐轨）](https://www.audiokinetic.com/zh/library/2021.1.12_7973/?source=Help&id=glossary#glossary_switch_musictrack)。

- Nested Object（嵌套对象）

  位于另一个对象内的对象。

- Nested Object（嵌套工作单元）

  嵌套在另一个工作单元内的工作单元。它可以让工程文件的粒度更细，减少在合并版本控制文件时工作组环境中存在的潜在冲突。

- Noise Gate（噪声门效果器）

  在 Expander 效果插件中创建的一种效果器，它几乎可以完全消除输出信号中的声音。创建噪声门效果器的方式是，设置高扩展率（大于 10:1），阻拦增益衰减到此程度的声音。

- Object（对象）

  Wwise 中如声音、Motion、角色混音器和容器这样的元素，用于容纳、组合和定义工程层级结构中的声音、语音、振动和音乐。

- Obstruction

  指游戏中对象（如柱子等）部分挡住声音对象与听者之间空间的一种状况。

- Occlusion（声笼）

  指游戏中对象（如墙壁等）完全挡住声音对象与听者之间空间的一种状况。

- Originals（原始音频）文件夹

  此文件夹中包含导入到工程中的音频文件未经改动的副本。此文件夹通常要做版本控制。

- Orphan file（落单文件）

  不再与声音Motion 或音乐对象相关联的音频文件。在您删除声音对象时不会自动删除这些文件。要删除这些文件，则要清空音频 .cache 文件夹。

- Output buffer latency（输出缓冲延迟）

  音频播放期间引入的延迟，由 Wwise 使用的输出缓冲去数量决定。

- Parametric EQ（参数均衡器）

  一种音频效果器插件，通过它可以应用各种滤波器来对声音频谱塑形。

- Parent Object（父对象）

  层级结构中的对象，其中包含子对象。

- Peak Limiter（峰值限幅器）

  控制音频信号动态范围的音频效果器。实现方法是削弱对音频信号中暂时超出预定义信号峰值阈值的部分。

- Physical Folder（实文件夹）

  硬盘上位于 Wwise 工程根下的目录，可包含工程中使用的其他实文件夹或工作单元。实文件夹不能作为容器、 Motion 或声音对象的子对象。

- Physical Voice（实声部）

  声音引擎播放和处理音频和振动所在的游戏物理环境。当音量电平变得极低时，声音和振动对象可转移到虚拟环境中，由声音引擎进行管理和监视，但不执行音频处理。

- Pitch Shifter（移调器）

  一种音频效果，可更改音高，但不影响所获音频信号的时长。

- Point Source（点声源）

  如同从单点发出声音或振动的声源。

- Post-exit（后尾段）

  出口提示点（exit cue）后的段落区域，可用于互动音乐中的过渡。

- Pre-entry（前导段）

  入口提示点（entry cue）前的段落区段，可用于互动音乐中的过渡。

- Prefetch Time（预取时间）

  在流播放中，它指一种小型缓冲区，照顾预取剩余文件数据所需要的延迟时间。

- Preset（预设）

  对象、效果和声音传播的一组自定义属性，可保存起来供随时复用。

- Query（查询）

  包含一组特定的搜索条件，用于查找特定对象或工程元素。

- Random Container （随机容器）

  按随机顺序播放的若干个声音、振动对象或容器的组合。

- Randomizer（随机化器）

  Wwise 中作用于属性值的一种特殊效果器，通过它您可以定义每次播放对象时可随机使用的可能值域。

- Random Music Track

  每次播放其父段落时，将按随机顺序播放其子音乐轨。

- Relative Property（相对属性）

  可在工程层级结构的各个层级上定义的属性，例如音量和音高。这些属性值是累加的，即父项的属性值将累加到子项的属性值上。

- Reverb

  一种音频效果器插件，用于模拟特定房间或空间的声学特性。

- Room Coupling（房间配对）

  Wwise Spatial Audio 中的概念，描述声音如何受到其所在房间的声学特性影响，并通过门户（房间开口）和墙壁（经过不同程度的阻塞和遮挡）以漫射场形式传播到相邻房间。

- RoomVerb 混响器

  一种多功能高品质混响效果器插件，用于模拟特定房间或空间的声学特性。

- RTPC（实时参数控制）

  实时参数控制（Real Time Parameter Controls）的缩写。一种互动方法，用于通过将游戏参数值映射到 Wwise 中的属性来驱动游戏中的音频。RTPC 还可用于通过将游戏参数映射到切换开关组来驱动切换开关的切换。

- 采样率

  在将模拟信号转换成数字信号或数字转码期间，每秒采样音频信号的频率。

- Send Volume

  发送到辅助总线的音频信号的电平或振幅。

- Sequence Container （序列容器）

  根据特定播放列表播放的若干个声音、振动对象或容器的组合。

- Sequence Music Track

  每次播放其父级段落时，都将按照顺序播放其子音轨。

- Shadow boundary

  View Region（照明区域，即声波路径未发生改变的区域）和 Shadow Region（阴影区域，即声波由于衍射而发生偏折的区域）的分界线。

- ShareSet（共享集）

  可在对象之间共享、用于定义效果器或衰减等属性的一组属性。

- ShortID

  短 ID。在 Wwise 工程内添加对象时，会在相关 Work Unit 文件中创建相应条目，并为其赋予一个 128 位的全局唯一标识符 (GUID) 编号。ShortID 可以是对象 GUID 的 FNV 哈希值，也可以是对象名称的 FNV 哈希值，即生成 32 位标识符。ShortID 的创建取决于对象的类型：对于 Event、Game Sync 和 SoundBank，哈希值为隐式值，不会在 Work Unit 文件 (XML) 中声明。它是 Wwise 工程中对象名称的 32 位 FNV 哈希值。对于所有其他对象（WorkUnit、Sound、Container、Bus 等），ShortID 为 GUID 字节的 32 位 FNV 哈希值。您可以在 SDK 中查看 FNV 哈希算法的 C++ 实现方式：/SDK/include/AK/Tools/Common/AkFNVHash.h

- Silence Source（空白源）

  一种插件音源，可以指定时长，但在此期间内不会产生声音和振动。

- Slider Range（滑杆范围）

  使用滑块设置属性时的默认值范围，如果在字段中手动输入此范围之外的值，将会改变滑块范围。

- SoundBank

  事件数据、声音、音乐、振动结构数据或媒体的编组，可在游戏中特定时刻一起加载至游戏平台内存中。

- Soundcaster

  Wwise 中的一种视图，可以在其中根据需要插入和移除对象或事件，提供了播放控制、试听和在 Wwise 或游戏中的实时混音等功能。

- Soundcaster Session（声音选角器会话）

  可被保存的工程元素，包含使用 Soundcaster 创建的特定模拟中所用的 Wwise 对象和事件。

- WAAPI

  一种允许外部应用程序与 Wwise 进行无缝通信的编程接口。

- 声音对象

  Wwise 工程中您创建的单个音频素材的一种表示。每个声音对象都包含若干声源，即游戏中播放的实际音频内容。请注意，Wwise 文档中使用的大写 “Sound” 是指音频对象，可以是 Sound SFX 或 Sound Voice。

- Sound SFX（音效声）

  Actor-Mixer Hierarchy 中包含音效、音乐和环境声的的声音对象。

- Sound Voice（语音声）

  包含对话或游戏语音的声音对象。

- Source Plug-in（源插件）

  由 Wwise 外部的插件创建的音频或振动源。

- Spatialization

  Wwise 中用于确定声音或音乐对象在游戏 3D 环境中的实际位置的一种功能。

- Spread

  扩散到附近扬声器的音频量或百分比，以使声音能够随着距离的增加从低扩散的点声源变为完全扩散的传播源。对于多声道声音，各个声道单独扩散。

- State（状态）

  根据游戏中的物理和环境条件的变化，对游戏音频属性进行全局偏置或调整。

- State Group

  将相关的状态进行分组，用来管理游戏环境中的全局更改。

- Stereo Delay（立体声延迟器）

  一种音频效果器，与内置滤波器一起提供双声道延迟。它有反馈和交叉回馈控件来控制延迟信号从一个声道发送到另一个声道，创建立体声效果。

- Stinger（插播乐句）

  与当前正在播放音乐叠加并混音的简短乐句。

- Switch（切换开关）

  Switch 代表游戏中特定元素的替代项，用于帮助管理这些替代项的相应对象。举例而言，如果角色在混凝土表面上奔跑，然后进入草地，则切换容器中的脚步声应随之改变，以匹配地面的变化。

- Switch Container（切换容器）

  该容器应用了一系列 Swtich 或 State，每一个都包含对应于游戏环境特定变化的一组声音、振动对象或容器。例如，角色脚步声的 Switch Container 可包含角色在游戏中可能行走的草地、混凝土、木材等表面所对应的 Switch。

- Switch Group （切换开关组）

  将相关的切换开关进行分组，用来管理游戏内指定元素在不同条件下的替代选项。

- Switch Music Track（切换开关音乐轨）

  根据相关联的切换开关组播放其子音乐轨。

- Time Stretch

  一种音频效果器，可改变时长但不影响所获音频信号的音高。

- 过渡

  在互动音乐中，使用 Transition 的目的是在源音乐段落和目标音乐段落之间实现无缝过渡。

- Transition Time（过渡时间）

  同一状态组中从一个状态过渡到另一个状态所使用的时间。在过渡期间，将发生两个状态属性的插值。

- Transmission（传输）

  Wwise Spatial Audio 中的一个概念，指的是声能通过障碍物后剩下的比例。

- Tremolo（震音）

  使用无极载波信号调制输入信号振幅的音频效果。

- Trigger （触发器）

  一种 Game Sync，响应游戏中的突发事件并播放 Stinger（插播乐句）。

- Virtual Folder（虚拟文件夹）

  起组织作用的对象，显示为一个文件夹，位于工作单元或其子对象中。您可以在虚拟文件夹中放置其他对象，如虚拟文件夹、角色混音器、容器、振动对象或声音。虚拟文件夹不能作为容器、振动或声音的子对象，且在硬盘上没有对应的目录。

- Virtual Voice（虚声部）

  一种虚拟环境，在此声音和振动由声音引擎管理和监视但不执行处理。当对象的音量电平低于音量阈值时，它们将进入虚声部。

- Voice

  单独或离散的音频或振动播放实例。

- Voice Starvation（声部匮乏）

  当声音引擎无法及时向平台硬件缓冲区提供音频数据时，Capture Log 中显示的一种错误消息类型。这种问题发生在 CPU 使用过量的场合。例如，平台 CPU 尝试混合过多声源或者同时使用过多音频效果器。

- Volume Threshold

  一个特定音量电平，低于此电平，可以明确地确定声音、音乐和振动对象的行为。例如，低于音量阈值的声部可以继续播放、终止或者将其发送到虚声部列表。这些行为可以在对象的 Property Editor 的 Advanced Settings 选项卡中来定义。

- Workgroup（工作组）

  同时处理同一 Wwise 工程的一组人员。

- Work Unit（工作单元）

  一个独特的 XML 文件，其中包含与工程中特定部分或元素相关的信息。
