# `0x72040000` 工具与 Debug

工具 Debug 页收束 console 命令、debug helpers、debug keys、debug render 和离线维护脚本。
这些入口适合验证源码理解，但不应被写成 gameplay 主流程。

## `0x72041000` 本页定位

### `0x72041100` 要回答的运行时问题

#### `0x72041110` 源码阅读目标

##### `0x72041111` 验证点

目标是给阅读源码时的验证动作提供入口，而不是把 debug 当 gameplay。
`DebugSpawn` 的定义在 `dst-scripts/util.lua`。
`debughelpers.lua` 主要提供 `DumpEntity`、`DumpComponent` 和 `DumpUpvalues`。

## `0x72042000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/util.lua` | `DebugSpawn` | 鼠标位置生成 prefab |
| `dst-scripts/consolecommands.lua` | `c_` | 控制台命令 |
| `dst-scripts/consolecommands.lua` | `c_spawn` / `c_give` | 基于 `DebugSpawn` 构造复现 |
| `dst-scripts/debugcommands.lua` | `d_` | 场景化调试命令 |
| `dst-scripts/debughelpers.lua` | `DumpEntity` | 实体和 component 检查 |
| `dst-scripts/debugtools.lua` | `DebugArcAttackHitBox` | debug render 辅助 |
| `dst-scripts/debugkeys.lua` | `AddGameDebugKey` | CHEATS_ENABLED 下的快捷键 |
| `dst-scripts/tools/getmissingstrings.lua` | `strings` | 文本工具 |
| `dst-scripts/tools/generate_worldgenoverride.lua` | `worldgen` | 世界设置工具 |

### `0x72042100` 主锚点

#### `0x72042110` `dst-scripts/util.lua`

##### `0x72042111` 搜索信号

先在 `dst-scripts/util.lua` 搜索 `function DebugSpawn`。
再到 `consolecommands.lua` 搜索 `c_spawn` 和 `c_give`，确认它们如何调用 `DebugSpawn`。

### `0x72042200` 检查锚点

#### `0x72042210` `dst-scripts/debughelpers.lua`

##### `0x72042211` 验证点

`debughelpers.lua` 只有 50 多行，重点是 `DumpComponent`、`DumpEntity` 和 `DumpUpvalues`。
它在 `main.lua` 启动后被 require，适合检查实体结构，而不是负责生成实体。

### `0x72042300` 维护脚本锚点

#### `0x72042310` `dst-scripts/tools`

##### `0x72042311` 边界条件

`dst-scripts/tools/getmissingstrings.lua` 通过 `package.path` 加载脚本并检查缺失文本。
`dst-scripts/tools/generate_worldgenoverride.lua` 需要在游戏 console 中 `require 'tools/generate_worldgenoverride'` 执行。
它们偏维护和导出，不是常驻 runtime。

## `0x72043000` 运行流程

~~~mermaid
flowchart TD
    A["main.lua requires consolecommands"]
    A --> B["CHEATS_ENABLED requires debugcommands/debugkeys"]
    C["console c_spawn / c_give"]
    C --> D["util.lua DebugSpawn"]
    D --> E["SpawnPrefab at ConsoleWorldPosition"]
    F["debughelpers DumpEntity"]
    F --> G["inspect entity/components/upvalues"]
    H["debugtools DebugRender"]
    H --> I["visualize hitbox/path/state"]
    J["tools/*.lua"]
    J --> K["offline or console maintenance output"]
~~~

### `0x72043100` 控制台阶段

#### `0x72043110` `c_spawn` 与 `c_give`

##### `0x72043111` 边界条件

`c_spawn(prefab)` 会把 prefab 转小写并调用 `DebugSpawn(prefab)`。
生成后会用 `SetDebugEntity(inst)` 选中新实体。
`c_give(prefab)` 同样生成实体，然后交给玩家 inventory。

### `0x72043200` Debug 命令阶段

#### `0x72043210` `d_` 命令族

##### `0x72043211` 搜索信号

`debugcommands.lua` 收集大量 `d_` 函数。
其中 `d_createscrapbookdata()`、`d_scanlayout()`、`d_testsound()` 这类命令能把运行时观察转成数据或导出文件。

### `0x72043300` Debug Key 阶段

#### `0x72043310` `CHEATS_ENABLED`

##### `0x72043311` 验证点

`main.lua` 只在 `CHEATS_ENABLED` 时 require `debugcommands.lua` 和 `debugkeys.lua`。
因此快捷键和部分 `d_` 命令不是普通运行环境的必经路径。

### `0x72043400` Tools 阶段

#### `0x72043410` 维护脚本输出

##### `0x72043411` 边界条件

`generate_worldgenoverride.lua` 会写出 `worldgenoverride.lua`。
`getmissingstrings.lua` 面向字符串覆盖检查。
审计这些脚本时要看它们读哪些表、写什么文件、是否依赖游戏内环境。

## `0x72044000` 结构细节

### `0x72044100` 数据结构与生命周期

#### `0x72044110` 具体 Lua 结构

##### `0x72044111` 需要核对的字段

`DebugSpawn` 定义在 `util.lua`，它会 `TheSim:LoadPrefabs({ prefab })`，然后在 `ConsoleWorldPosition()` 生成实体。
`consolecommands.lua` 提供 `c_spawn`、`c_give`、`c_select`、`c_find`、`c_sounddebug` 等可组合入口。
`debughelpers.lua` 用于 dump 已有对象结构，不承担 spawn。
`debugtools.lua` 用于 debug print、debug render 和 hitbox 可视化。
`debugkeys.lua` 把调试功能挂到键盘事件。
`tools` 目录目前只有 2 个 Lua 文件，偏维护，不应作为运行时主线。

## `0x72045000` 阅读与验证路线

### `0x72045100` 从哪里开始读源码

~~~bash
rg -n "function DebugSpawn|ConsoleWorldPosition|SpawnPrefab" \
  dst-scripts/util.lua

rg -n "function c_spawn|function c_give|function c_select|function c_sounddebug|DebugSpawn" \
  dst-scripts/consolecommands.lua

rg -n "function d_createscrapbookdata|function d_scanlayout|function d_testsound|function d_require" \
  dst-scripts/debugcommands.lua

rg -n "DumpEntity|DumpComponent|DumpUpvalues|DebugArcAttackHitBox|AddGameDebugKey" \
  dst-scripts/debughelpers.lua \
  dst-scripts/debugtools.lua \
  dst-scripts/debugkeys.lua

rg -n "package.path|generate_worldgenoverride|GetWorldSettingsOptions|GetMissing" \
  dst-scripts/tools
~~~

#### `0x72045110` 推荐顺序

##### `0x72045111` 最小闭环

复现 prefab 问题时，从 `c_spawn -> DebugSpawn -> SpawnPrefab` 开始。
观察实体结构时，用 `DumpEntity` 或 `DumpComponent` 对照 component 页面。
验证 hitbox 或范围时，看 `debugtools.lua` 的 debug render 辅助。
检查字符串或 worldgen override 时，再进入 `dst-scripts/tools`。
任何 debug 命令造成的状态变化，都要回到正式 component、SG 或 prefab 逻辑再确认一遍。
