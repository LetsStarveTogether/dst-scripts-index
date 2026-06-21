# `0x40000000` AI 与动画

本区说明实体如何把意图交给 StateGraph，把长期选择交给 Brain，再由 behaviour tree 产出下一步动作。

目录级语义由本 README 承载，独立专题文件只解释具体运行链路。

## `0x40001000` 区域定位

### `0x40001100` 先分清两条运行链

#### `0x40001110` `StateGraph` 负责短时表现

##### `0x40001111` 验证点

在 `dst-scripts/stategraph.lua` 中确认 `StateGraphInstance:StartAction`、`GoToState` 和 `UpdateState`。
它们处理动作入口、状态切换、timeline、timeout 和 per-state event。

#### `0x40001120` `Brain` 负责长期选择

##### `0x40001121` 验证点

在 `dst-scripts/brain.lua` 和 `dst-scripts/behaviourtree.lua` 中确认 Brain 调度 behaviour node。
Brain 不直接播放动画，而是推动实体产生可被 StateGraph 消化的意图。

## `0x40002000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/stategraph.lua` | `SGManager` | 管理 SG 实例更新、休眠和事件队列 |
| `dst-scripts/stategraph.lua` | `StateGraphInstance` | 绑定实体的状态机实例 |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.Add*` | 向 `states` 表追加通用状态 |
| `dst-scripts/stategraphs/SGwilson.lua` | `StateGraph("wilson")` | 玩家服务端 StateGraph |
| `dst-scripts/stategraphs/SGwilson_client.lua` | `StateGraph("wilson_client")` | 玩家客户端预测 StateGraph |
| `dst-scripts/brain.lua` | `BrainManager` | Brain 更新入口 |
| `dst-scripts/behaviourtree.lua` | `PriorityNode` | 行为树选择节点 |

### `0x40002100` 入口选择

#### `0x40002110` `dst-scripts/stategraph.lua`

##### `0x40002111` 搜索信号

先搜 `StateGraphInstance:StartAction` 和 `StateGraphInstance:UpdateState`。
这两个函数能把动作入口和动画帧副作用连起来。

#### `0x40002120` `dst-scripts/brain.lua`

##### `0x40002121` 搜索信号

再搜 `BrainManager`、`BT` 和 `OnUpdate`。
这一步用于确认 AI 决策如何在 StateGraph 之外运行。

## `0x40003000` 运行关系图

~~~mermaid
flowchart TD
    A["Prefab 设置 sgname 与 brainname"]
    A --> B["StateGraphInstance"]
    A --> C["Brain"]
    C --> D["BehaviourNode 选择意图"]
    D --> E["BufferedAction / PushEvent / direct component call"]
    E --> B
    E --> I["combat / locomotor / homeseeker 等组件"]
    B --> F["State.onenter"]
    F --> G["timeline / timeout / events"]
    G --> H["PerformBufferedAction 或 GoToState"]
~~~

### `0x40003100` 导航原则

#### `0x40003110` 先读表现层，再读决策层

##### `0x40003111` 边界条件

`StateGraph` 可以没有 Brain，例如许多物件或简单生物。
有 Brain 的实体常把 `BufferedAction` 交给 SG。
部分 behaviour leaf 也会直接驱动 `combat`、`locomotor`、`homeseeker` 等组件。

#### `0x40003120` 先读服务端，再读客户端预测

##### `0x40003121` 边界条件

`SGwilson.lua` 是权威状态。
`SGwilson_client.lua` 用 `server_states` 和 `forward_server_states` 描述哪些客户端状态可匹配服务端状态。

## `0x40004000` 目录索引

### `0x40004100` README 载体

#### `0x40004110` 二级目录

##### `0x40004111` 链接校验

以下入口先进入目录 README，再进入具体专题文件。

- [StateGraph](0x4100-stategraph/README.md)
- [StateGraph Runtime](0x4100-stategraph/0x4101-stategraph-runtime.md)
- [CommonStates](0x4100-stategraph/0x4102-commonstates.md)
- [SGWilson](0x4100-stategraph/0x4103-sgwilson.md)
- [Brain 与 Behaviour](0x4200-brain-behaviour/README.md)
- [Brain Runtime](0x4200-brain-behaviour/0x4201-brain-runtime.md)
- [Behaviour Tree](0x4200-brain-behaviour/0x4202-behaviour-tree.md)
- [追踪样例](0x4300-tracing/README.md)
- [AI Examples](0x4300-tracing/0x4301-ai-examples.md)

## `0x40005000` 阅读与验证路线

### `0x40005100` 从哪里开始读源码

~~~bash
rg -n "StateGraphInstance:StartAction|StateGraphInstance:UpdateState|BrainManager|PriorityNode" \
  dst-scripts/stategraph.lua \
  dst-scripts/brain.lua \
  dst-scripts/behaviourtree.lua
~~~

#### `0x40005110` 最小闭环

##### `0x40005111` 抽样动作

从 `dst-scripts/stategraphs/SGrabbit.lua` 的 `ActionHandler(ACTIONS.EAT, "eat")` 开始。
再读 `eat` 状态里的 `SetTimeout`、`ontimeout` 和 `PerformBufferedAction`。

##### `0x40005112` 抽样 AI

从 rabbit prefab 追到 rabbit brain，再回到 `SGrabbit`。
这样能验证 Brain 选择意图，StateGraph 执行表现的边界。
