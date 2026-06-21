# `0x10040000` 术语表

术语表只解释源码阅读中的高频运行时名词，不替代 reference 清单。

## `0x10041000` 本页定位

### `0x10041100` 要回答的运行时问题

#### `0x10041110` 源码阅读目标

##### `0x10041111` 验证点

读者要能区分 `Prefab`、`EntityScript`、`Component`、`Replica`、`StateGraph` 和 `Brain` 的职责边界。

术语解释必须能回到源码文件，不按游戏概念泛泛解释。

## `0x10042000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/prefabs.lua` | `Prefab = Class` | Prefab 对象定义 |
| `dst-scripts/entityscript.lua` | `AddComponent` | 组件挂载入口 |
| `dst-scripts/entityreplica.lua` | `ReplicateComponent` | client replica 附加入口 |
| `dst-scripts/stategraph.lua` | `StateGraphInstance` | 表现状态机实例 |
| `dst-scripts/brain.lua` | `BrainWrangler` / `BrainManager` | AI 更新调度 |

### `0x10042100` 主锚点

#### `0x10042110` `dst-scripts/entityscript.lua`

##### `0x10042111` 搜索信号

先在 `dst-scripts/entityscript.lua` 搜索 `AddComponent`。

再搜索 `ReplicateComponent`，确认 server 组件和 client replica 的分界。

## `0x10043000` 运行流程

~~~mermaid
flowchart TD
    A["Prefab"]
    A --> B["EntityScript"]
    B --> C["Component"]
    C --> D["Replica"]
    D --> E["StateGraph"]
    E --> F["Brain"]
~~~

### `0x10043100` 流程分段

#### `0x10043110` 入口到副作用

##### `0x10043111` 边界条件

- `Prefab` 是装配函数与依赖声明，不等同于运行中的实体实例。
- `EntityScript` 是 Lua 侧实体壳，保存 `components`、`replica`、事件监听和 buffered action。
- `Component` 是 server 权威行为状态，`*_replica` 是 client 可读接口。
- `StateGraph` 处理动作表现、状态标签、事件和动画窗口。
- `Brain` 通过 behaviour tree 产生意图，通常不直接播放动画。

## `0x10044000` 结构细节

### `0x10044100` 数据结构与生命周期

#### `0x10044110` 具体 Lua 结构

##### `0x10044111` 需要核对的字段

| 术语 | 源码锚点 | 阅读边界 |
| --- | --- | --- |
| `Prefab` | `dst-scripts/prefabs.lua` | 装配实体、资产依赖和子 prefab |
| `EntityScript` | `dst-scripts/entityscript.lua` | 实体生命周期、组件、事件、动作和存档 |
| `Component` | `dst-scripts/components/*.lua` | server 侧权威行为状态 |
| `Replica` | `dst-scripts/components/*_replica.lua` | client 侧只读或预测接口 |
| `Classified` | `dst-scripts/prefabs/*_classified.lua` | 网络变量承载体 |
| `StateGraph` | `dst-scripts/stategraph.lua` | 动画状态、动作窗口和事件响应 |
| `Brain` | `dst-scripts/brain.lua` | AI 调度和 behaviour tree 入口 |

## `0x10045000` 阅读与验证路线

### `0x10045100` 从哪里开始读源码

~~~bash
rg -n "Prefab = Class|AddComponent|ReplicateComponent|SetStateGraph|SetBrain" \
  dst-scripts/prefabs.lua \
  dst-scripts/entityscript.lua \
  dst-scripts/entityreplica.lua
rg -n "StateGraphInstance|BrainWrangler|BrainManager" \
  dst-scripts/stategraph.lua \
  dst-scripts/brain.lua
~~~

#### `0x10045110` 推荐顺序

##### `0x10045111` 最小闭环

先用术语表建立职责边界，再进入对应专题页。

如果术语需要列出大量实例，转入 reference，不在本页扩表。
