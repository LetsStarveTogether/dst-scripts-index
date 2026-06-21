# `0x23020000` 平台工具

平台工具页收束 `tools/` 脚本、用户命令和更新循环的工具性入口。

这里的 `update.lua` 是帧更新循环，不是游戏下载更新器。

## `0x23021000` 本页定位

### `0x23021100` 要回答的运行时问题

#### `0x23021110` 哪些 Lua 文件服务工具链

##### `0x23021111` 验证点

`dst-scripts/tools/` 当前只包含 `generate_worldgenoverride.lua` 和 `getmissingstrings.lua`。

前者通过 `require 'tools/generate_worldgenoverride'` 在控制台执行，并直接写出 `worldgenoverride.lua`。

后者执行 `TestStrings()`，生成缺失字符串报告。

### `0x23021200` 哪些入口仍会进入运行时

#### `0x23021210` 用户命令和更新循环

##### `0x23021211` 验证点

`usercommands.lua` 和 `builtinusercommands.lua` 是玩家或管理员命令系统。

`update.lua` 是每帧循环入口，负责 component、stategraph、brain、frontend 等更新。

## `0x23022000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/tools/generate_worldgenoverride.lua` | top level script | 生成 `worldgenoverride.lua` |
| `dst-scripts/tools/getmissingstrings.lua` | `TestStrings` | 检查角色和 prefab 缺失字符串 |
| `dst-scripts/usercommands.lua` | `AddUserCommand` | 注册用户命令 |
| `dst-scripts/usercommands.lua` | `RunUserCommand` | 执行解析后的用户命令 |
| `dst-scripts/builtinusercommands.lua` | `AddUserCommand` | 注册内置 `help`、`kick`、`rollback` 等命令 |
| `dst-scripts/update.lua` | `WallUpdate` | 墙钟更新、输入和前端更新 |
| `dst-scripts/update.lua` | `Update` | server 未暂停时的模拟更新 |
| `dst-scripts/update.lua` | `PostUpdate` | emitter 和 update looper 后处理 |
| `dst-scripts/stats.lua` | `GetClientMetricsData` | 给 C++ 和 Lua 统计系统提供客户端指标 |
| `dst-scripts/stats.lua` | `BuildContextTable` | 构造 tracking context |
| `dst-scripts/platformpostload.lua` | top level script | 平台后置补丁 |

### `0x23022100` 工具脚本锚点

#### `0x23022110` `generate_worldgenoverride.lua`

##### `0x23022111` 搜索信号

这个文件没有名为 `generate` 的函数。

它在 top level 读取 `map/customize` 和 `map/levels`，拼接文本，最后用 `io.open` 写文件。

### `0x23022200` 用户命令锚点

#### `0x23022210` `usercommands.lua`

##### `0x23022211` 搜索信号

搜索 `AddUserCommand`、`RunUserCommand` 和 `RunTextUserCommand`。

命令会经过权限、投票、确认和 local/server 执行类型判断。

### `0x23022300` 更新循环锚点

#### `0x23022310` `update.lua`

##### `0x23022311` 搜索信号

搜索 `WallUpdate`、`StaticUpdate`、`Update`、`LongUpdate` 和 `PostUpdate`。

这些函数属于运行时循环，应与下载更新或补丁系统区分。

## `0x23023000` 运行流程

~~~mermaid
flowchart TD
    A["tool script required from console"]
    A --> B["top level Lua code runs"]
    B --> C["writes generated file or report"]
    D["user text command"]
    D --> E["RunTextUserCommand"]
    E --> F["RunUserCommand"]
    F --> G["localfn / serverfn / vote"]
~~~

### `0x23023100` 工具脚本阶段

#### `0x23023110` `tools/`

##### `0x23023111` 边界条件

`generate_worldgenoverride.lua` 依赖当前 Lua 环境中的地图配置表。

它直接写当前工作目录下的 `worldgenoverride.lua`。

`getmissingstrings.lua` 会加载 prefab 与角色 speech 数据，最后调用 `TestStrings()`。

### `0x23023200` 用户命令阶段

#### `0x23023210` `builtinusercommands.lua`

##### `0x23023211` 边界条件

内置命令通过 `AddUserCommand` 注册。

命令数据可包含 `localfn`、`serverfn`、权限函数、投票配置和参数定义。

### `0x23023300` 更新阶段

#### `0x23023310` `update.lua`

##### `0x23023311` 边界条件

`WallUpdate` 即使 server 暂停也会处理部分 UI、音频和输入更新。

`Update` 在 server 暂停时断言不应被调用。

`SGManager` 和 `BrainManager` 的更新在 `Update` 中分频执行。

## `0x23024000` 结构细节

### `0x23024100` `tools/` 清单

#### `0x23024110` 当前文件

##### `0x23024111` 需要核对的字段

`tools/` 不是大型工具目录。

当前只有 2 个 tracked Lua 文件。

新增工具时应同步更新本页和 reference 清单。

### `0x23024200` 用户命令表

#### `0x23024210` 命令数据

##### `0x23024211` 需要核对的字段

`AddUserCommand(name, data)` 把命令登记到用户命令系统。

`AddModUserCommand(mod, name, data)` 给 mod 命令加命名空间。

`RemoveUserCommand(name)` 可移除命令。

### `0x23024300` 更新注册表

#### `0x23024310` 静态组件更新

##### `0x23024311` 需要核对的字段

`RegisterStaticComponentUpdate` 和 `RegisterStaticComponentLongUpdate` 把类级更新函数登记到表里。

`Update` 和 `LongUpdate` 会遍历这些表。

### `0x23024400` 统计支持

#### `0x23024410` `stats.lua`

##### `0x23024411` 需要核对的字段

`GetClientMetricsData` 是给 C++ 调用的全局函数。

`BuildContextTable` 会读取 `TheNet`、`TheWorld`、`KnownModIndex` 和 `Profile` 生成统计上下文。

`SendTrackingStats` 是 `stats.lua` 内部局部函数，不作为外部调用锚点。

### `0x23024500` 平台后置补丁

#### `0x23024510` `platformpostload.lua`

##### `0x23024511` 需要核对的字段

`gamelogic.lua` 在加载内置用户命令后 require `platformpostload`。

该文件按平台注入或移除用户命令，并调整部分投票规则。

## `0x23025000` 阅读与验证路线

### `0x23025100` 从哪里开始读源码

~~~bash
rg -n "AddUserCommand|RunUserCommand|RunTextUserCommand|TestStrings|io.open|platformpostload" \
  dst-scripts/tools \
  dst-scripts/usercommands.lua \
  dst-scripts/builtinusercommands.lua \
  dst-scripts/update.lua \
  dst-scripts/stats.lua \
  dst-scripts/gamelogic.lua \
  dst-scripts/platformpostload.lua
rg -n "WallUpdate|StaticUpdate|Update|LongUpdate|PostUpdate|GetClientMetricsData|BuildContextTable" \
  dst-scripts/tools \
  dst-scripts/usercommands.lua \
  dst-scripts/builtinusercommands.lua \
  dst-scripts/update.lua \
  dst-scripts/stats.lua
~~~

#### `0x23025110` 推荐顺序

##### `0x23025111` 最小闭环

先确认 `tools/` 下两个文件都是 top level 脚本式工具。

再读 `builtinusercommands.lua` 的一个内置命令。

最后读 `update.lua`，把 `Update` 理解为帧循环而不是版本更新。
