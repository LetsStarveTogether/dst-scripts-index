# `0x50000000` 世界模拟

本区按 worldgen、Level/Task/Room/Layout、天气季节、洞穴海洋组织。

目录级语义由本 README 承载，独立专题文件只解释具体运行链路。

## `0x50001000` 区域定位

### `0x50001100` 阅读问题

#### `0x50001110` 运行时入口

##### `0x50001111` 验证点

读者要把生成阶段和运行阶段分开阅读。

## `0x50002000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/worldgen_main.lua` | `GenerateNew` | 世界生成入口 |
| `dst-scripts/map/levels.lua` | `AddWorldGenLevel` | 注册 worldgen preset |
| `dst-scripts/map/level.lua` | `Level` | 包装 preset 并选择 task set |
| `dst-scripts/map/storygen.lua` | `Story:GenerateNodesFromTasks` | 构造拓扑 graph |
| `dst-scripts/map/forest_map.lua` | `Generate` | WorldSim 烘焙与实体表写入 |
| `dst-scripts/components/worldstate.lua` | `WorldState` | 运行时世界状态 |

### `0x50002100` 入口选择

#### `0x50002110` `dst-scripts/worldgen_main.lua`

##### `0x50002111` 搜索信号

用 `GenerateNew` 判断本区域从哪个运行时对象开始。

## `0x50003000` 运行关系图

~~~mermaid
flowchart TD
    A["worldgen params"]
    A --> B["worldgen_main.GenerateNew"]
    B --> C["Level(level_data)"]
    C --> D["Task / Room"]
    D --> E["storygen graph"]
    E --> F["forest_map + WorldSim"]
    F --> G["savedata.map / savedata.ents"]
    G --> H["runtime worldstate"]
~~~

### `0x50003100` 导航原则

#### `0x50003110` 专题入口

##### `0x50003111` 本区不放穷举清单

本区索引只放运行关系和源码入口，完整文件目录统一进入 `0x8000-reference`。

## `0x50004000` 目录索引

### `0x50004100` README 载体

#### `0x50004110` 二级目录

##### `0x50004111` 链接校验

以下入口先进入目录 README，再进入具体专题文件。

- [世界生成](0x5100-worldgen/README.md)
- [Worldgen Main](0x5100-worldgen/0x5101-worldgen-main.md)
- [Levels Tasks Rooms](0x5100-worldgen/0x5102-levels-tasks-rooms.md)
- [Layouts](0x5100-worldgen/0x5103-layouts.md)
- [世界状态](0x5200-world-state/README.md)
- [天气与季节](0x5200-world-state/0x5201-weather-seasons.md)
- [洞穴海洋与遗迹](0x5200-world-state/0x5202-caves-ocean-ruins.md)

## `0x50005000` 阅读与验证路线

### `0x50005100` 从哪里开始读源码

~~~bash
rg -n "GenerateNew|AddWorldGenLevel|function Level:ChooseTasks|GenerateNodesFromTasks|function Generate|WorldState" \
  dst-scripts/worldgen_main.lua \
  dst-scripts/map/levels.lua \
  dst-scripts/map/level.lua \
  dst-scripts/map/storygen.lua \
  dst-scripts/map/forest_map.lua \
  dst-scripts/components/worldstate.lua
~~~

#### `0x50005110` 最小闭环

##### `0x50005111` 抽样动作

抽样从 `GenerateNew` 追到 `GenerateNodesFromTasks` 和 `Room` 内容。
