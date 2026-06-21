# `0x22020000` 存档与读档

存档页按世界快照、实体记录、组件持久化和 shard index 收尾阅读。

读档页按 `ShardGameIndex` 取数、升级存档、初始化世界和实体恢复阅读。

## `0x22021000` 本页定位

### `0x22021100` 要回答的运行时问题

#### `0x22021110` 世界为什么不是直接序列化 Lua 对象

##### `0x22021111` 验证点

`SaveGame` 遍历 `Ents`，但只保存满足 `persists`、`prefab`、`Transform` 和 parent 条件的实体。

实体通过 `GetSaveRecord` 输出记录。

组件通过 `OnSave` 输出自己的持久化表。

### `0x22021200` 读档的入口边界

#### `0x22021210` `ShardGameIndex`

##### `0x22021211` 验证点

`gamelogic.lua` 负责决定加载已有世界还是生成新世界。

实际读取世界数据通过 `ShardGameIndex:GetSaveData()` 或 `ShardGameIndex:GetSaveDataFile()` 完成。

## `0x22022000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/mainfunctions.lua` | `SaveGame` | server 侧世界保存入口 |
| `dst-scripts/mainfunctions.lua` | `SerializeWorldSession` | 把分块后的世界数据交给 `TheNet` |
| `dst-scripts/networking.lua` | `SerializeUserSession` | 保存玩家会话并可附带 `player_classified` |
| `dst-scripts/entityscript.lua` | `GetSaveRecord` | 输出单个实体的保存记录 |
| `dst-scripts/entityscript.lua` | `GetPersistData` | 调用组件 `OnSave` 收集持久化表 |
| `dst-scripts/entityscript.lua` | `SetPersistData` | 调用组件 `OnLoad` 恢复持久化表 |
| `dst-scripts/gamelogic.lua` | `DoLoadWorld` | 从 shard index 读取世界并初始化游戏 |
| `dst-scripts/saveindex.lua` | `SaveIndex:Save` | legacy 入口，当前实现不再写索引 |
| `dst-scripts/shardindex.lua` | `ShardIndex:Save` | 保存 shard 的 world、server 和 session 索引 |
| `dst-scripts/shardsaveindex.lua` | `ShardSaveIndex:GetShardIndex` | 管理 slot 到 shard index 的缓存 |

### `0x22022100` 保存锚点

#### `0x22022110` `dst-scripts/mainfunctions.lua`

##### `0x22022111` 搜索信号

搜索 `SaveGame` 后继续看 `TheNet:StartWorldSave()` 和 `TheNet:EndWorldSave()`。

这两个调用标记世界保存过程的开始和结束。

### `0x22022200` 实体锚点

#### `0x22022210` `dst-scripts/entityscript.lua`

##### `0x22022211` 搜索信号

搜索 `GetSaveRecord` 可以看到位置、平台和 prefab 信息如何进入实体记录。

搜索 `GetPersistData` 可以看到组件 `OnSave` 如何合并到 `data[k]`。

### `0x22022300` 读档锚点

#### `0x22022310` `dst-scripts/gamelogic.lua`

##### `0x22022311` 搜索信号

搜索 `DoLoadWorld` 可以看到 `ShardGameIndex:GetSaveData(onload)`。

`onload` 会执行 `UpgradeSaveFile`、`LoadAssets` 和 `DoInitGame`。

## `0x22023000` 运行流程

~~~mermaid
flowchart TD
    A["SaveGame"]
    A --> B["filter Ents"]
    B --> C["EntityScript:GetSaveRecord"]
    C --> D["EntityScript:GetPersistData"]
    D --> E["DataDumper per save section"]
    E --> F["SerializeWorldSession"]
    F --> G["ShardGameIndex:Save"]
    G --> H["ShardGameIndex:WriteTimeFile"]
~~~

### `0x22023100` 保存阶段

#### `0x22023110` 实体和地图

##### `0x22023111` 边界条件

`SaveGame` 会保存地图编码、道路、拓扑、世界组件、`world_network` 和可选 `shard_network`。

它也会把实体引用补回到对应保存记录的 `id`。

### `0x22023200` 玩家会话阶段

#### `0x22023210` `SerializeUserSession`

##### `0x22023211` 边界条件

玩家会话保存调用 `player:GetSaveRecord()`。

server 侧会把角色 prefab 写入 metadata。

`player_classified` 存在时会把其 entity 传给 `TheNet:SerializeUserSession()`。

### `0x22023300` 读档阶段

#### `0x22023310` `DoLoadWorld`

##### `0x22023311` 边界条件

读档不是从 `SaveGame` 反向返回。

它从 `gamelogic.lua` 的 `LoadSlot` 进入，再通过 `ShardGameIndex` 读取存档并调用 `DoInitGame`。

## `0x22024000` 结构细节

### `0x22024100` `GetPersistData`

#### `0x22024110` 组件 `OnSave`

##### `0x22024111` 需要核对的字段

组件返回非空 table 时会进入 `data[component_name]`。

组件返回 refs 时会追加到引用列表。

实体自身的 `OnSave` 可以在组件之后追加数据和引用。

### `0x22024200` `SetPersistData`

#### `0x22024210` 组件 `OnLoad`

##### `0x22024211` 需要核对的字段

`add_component_if_missing` 会让读档路径补加缺失组件。

`OnPreLoad` 在组件 `OnLoad` 前执行。

`LoadPostPass` 是第二阶段恢复引用的入口。

### `0x22024300` `SaveIndex`

#### `0x22024310` Shard 索引文件

##### `0x22024311` 需要核对的字段

`gamelogic.lua` 在当前链路里创建的是 `ShardGameIndex = ShardIndex()`。

`ShardIndex:Save` 通过 `TheSim:SetPersistentStringInClusterSlot` 或 `TheSim:SetPersistentString` 写出 `shardindex`。

`SaveIndex:Save` 是保留的 legacy 入口，当前实现只调用 callback，不再写索引文件。

世界主体数据通过 `SerializeWorldSession` 保存，保存后再更新 worldgen overrides、shard index 和时间文件。

## `0x22025000` 阅读与验证路线

### `0x22025100` 从哪里开始读源码

~~~bash
rg -n "SaveGame|GetSaveRecord|GetPersistData|SetPersistData|LoadPostPass|DoLoadWorld|ShardIndex:Save|SaveIndex:Save" \
  dst-scripts/mainfunctions.lua \
  dst-scripts/entityscript.lua \
  dst-scripts/gamelogic.lua \
  dst-scripts/saveindex.lua \
  dst-scripts/shardindex.lua \
  dst-scripts/shardsaveindex.lua \
  dst-scripts/networking.lua
~~~

#### `0x22025110` 推荐顺序

##### `0x22025111` 最小闭环

先追 `SaveGame` 的实体循环。

再跳到 `EntityScript:GetPersistData` 看组件 `OnSave`。

最后读 `DoLoadWorld`，确认读档从 `ShardGameIndex` 回到 `DoInitGame`。
