# `0x41020000` CommonStates

CommonStates 页解释 `stategraphs/commonstates.lua` 如何用 helper 向具体 SG 的 `states` 表追加通用状态。
它不是运行时调度器，而是状态定义工厂。

## `0x41021000` 本页定位

### `0x41021100` 要回答的运行时问题

#### `0x41021110` 源码阅读目标

##### `0x41021111` 验证点

目标是看懂为什么许多生物 SG 很短。
例如 `SGrabbit.lua` 手写 `idle`、`eat`、`hop`、`run`，再用 CommonStates 追加 sleep、frozen、corpse 等状态。

#### `0x41021120` 生成边界

##### `0x41021121` 验证点

CommonStates helper 大多只修改传入的 `states` 表。
事件入口常来自同文件的 `CommonHandlers.On*`，实际跳转仍通过 `inst.sg:GoToState(...)`。

## `0x41022000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.AddIdle` | 追加 idle 状态和循环 timeout |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.AddRunStates` | 追加 `run_start`、`run`、`run_stop` |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.AddSleepStates` | 追加 `sleep`、`sleeping`、`wake` |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.AddFrozenStates` | 追加 `frozen` 与 `thaw` |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.AddCombatStates` | 追加 `hit`、`attack`、`death` |
| `dst-scripts/stategraphs/commonstates.lua` | `CommonStates.AddInitState` | 追加默认入口 `init` |
| `dst-scripts/stategraphs/SGrabbit.lua` | `CommonStates.Add*` | 简单生物使用样例 |

### `0x41022100` 主锚点

#### `0x41022110` `dst-scripts/stategraphs/commonstates.lua`

##### `0x41022111` 搜索信号

先搜 `CommonStates.AddRunStates` 和 `CommonStates.AddCombatStates`。
它们覆盖移动和战斗两类最常见复用模式。

#### `0x41022120` `dst-scripts/stategraphs/SGrabbit.lua`

##### `0x41022121` 搜索信号

搜 `CommonStates.AddSleepStates`、`AddFrozenStates` 和 `AddInitState`。
这能确认 helper 是在文件末尾把通用状态追加进同一个 `states` 表。

## `0x41023000` 运行流程

~~~mermaid
flowchart TD
    A["具体 SG 创建 states 表"]
    A --> B["手写专属 State"]
    B --> C["CommonStates.Add* helper"]
    C --> D["table.insert(states, State{...})"]
    D --> E["return StateGraph(name, states, events, defaultstate, actionhandlers)"]
    E --> F["StateGraph 构造器重建 states 索引"]
    F --> G["运行时 GoToState 使用 state.name"]
~~~

### `0x41023100` 生成阶段

#### `0x41023110` `table.insert(states, State{...})`

##### `0x41023111` 边界条件

CommonStates 不创建新的 `StateGraphInstance`。
它只在 `StateGraph(...)` 调用前追加状态对象。
因此同名状态会在 `StateGraph` 构造器按 `v.name` 重建索引时决定最终结果。

### `0x41023200` 事件阶段

#### `0x41023210` `CommonHandlers.On*`

##### `0x41023211` 边界条件

`CommonHandlers.OnSleep()`、`OnFreeze()` 等返回 `EventHandler`。
具体 SG 把这些 handler 放入全局 `events` 表，运行时由 `StateGraphInstance:HandleEvent` 后备处理。

### `0x41023300` 时间阶段

#### `0x41023310` Helper 内的 Timeout 与 Timeline

##### `0x41023311` 边界条件

`AddIdle` 在未 push 动画时用当前动画长度设置 timeout。
`AddSimpleActionState` 默认用 `TimeEvent(time, performbufferedaction)` 执行动作。
`AddShortAction` 使用 `SetTimeout`，但源码里状态名写成字符串 `"name"`，阅读时要警惕这个历史实现。

## `0x41024000` 结构细节

### `0x41024100` 移动状态

#### `0x41024110` `CommonStates.AddRunStates`

##### `0x41024111` 需要核对的字段

`AddRunStates` 追加 `run_start`、`run`、`run_stop`。
`run_start` 的 `animover` 进入 `run`。
`run` 在 `onenter` 调用 `locomotor:RunForward()`，并用动画长度设置 timeout。
`run_stop` 停止移动，等 `animqueueover` 回到 idle。

### `0x41024200` 战斗状态

#### `0x41024210` `CommonStates.AddCombatStates`

##### `0x41024211` 需要核对的字段

`AddCombatStates` 追加 `hit`、`attack`、`death`。
`attack` 的 `onenter` 调用 `components.combat:StartAttack()`。
它把 target 缓存在 `inst.sg.statemem.target`，供 timeline 中的真实攻击使用。

### `0x41024300` 入口状态

#### `0x41024310` `CommonStates.AddInitState`

##### `0x41024311` 需要核对的字段

`AddInitState` 追加名为 `init` 的状态。
该状态进入后立刻根据 `inst.is_corpse` 跳到 `corpse_idle` 或默认状态。

## `0x41025000` 阅读与验证路线

### `0x41025100` 从哪里开始读源码

~~~bash
rg -n "CommonStates.AddRunStates|CommonStates.AddCombatStates|CommonStates.AddInitState|CommonHandlers.On" \
  dst-scripts/stategraphs/commonstates.lua \
  dst-scripts/stategraphs/SGrabbit.lua
~~~

#### `0x41025110` 推荐顺序

##### `0x41025111` 最小闭环

先读 `SGrabbit.lua` 文件末尾的 CommonStates 调用。
再回到 `commonstates.lua` 找对应 helper。
最后确认 helper 追加的状态名是否会被 `return StateGraph("rabbit", states, events, "init", actionhandlers)` 使用。

##### `0x41025112` 抽样验证

用 rabbit 的 `OnFreeze()` 追到 `CommonHandlers.OnFreeze()`。
再确认 `AddFrozenStates` 是否向 `states` 表追加了 `frozen` 和 `thaw`。
