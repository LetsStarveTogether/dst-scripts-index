# `0x10050000` 维护规则

维护规则约束新增页面必须先说明运行链路，再放源码锚点，最后才放清单。

## `0x10051000` 本页定位

### `0x10051100` 要回答的运行时问题

#### `0x10051110` 源码阅读目标

##### `0x10051111` 验证点

维护者要优先核对源码事实、文档边界和 reference 清单的一致性。

维护文档时只改索引 Markdown，不能为了让文档成立去改 `dst-scripts` Lua。

BBC 编码、标题层级和源码路径应保持简单、可读、可追踪。

## `0x10052000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/mainfunctions.lua` | `SaveGame` | 存档链路可审计 |
| `dst-scripts/worldgen_main.lua` | `CheckMapSaveData` | 生成结果可校验 |
| `dst-scripts/entityscript.lua` | `GetPersistData` | 实体状态可追踪 |
| `dst-scripts/entityreplica.lua` | `ReplicateEntity` | client replica 可追踪 |

### `0x10052100` 主锚点

#### `0x10052110` `dst-scripts/mainfunctions.lua`

##### `0x10052111` 搜索信号

先在 `dst-scripts/mainfunctions.lua` 搜索 `SaveGame`。

如果维护的是世界生成页面，再搜索 `CheckMapSaveData`。

如果维护的是实体或网络页面，再搜索 `GetPersistData`、`ReplicateEntity` 或 `ReplicateComponent`。

## `0x10053000` 运行流程

~~~mermaid
flowchart TD
    A["改文档"]
    A --> B["核对源码锚点"]
    B --> C["收敛专题边界"]
    C --> D["抽样关键链路"]
~~~

### `0x10053100` 流程分段

#### `0x10053110` 入口到副作用

##### `0x10053111` 边界条件

- 专题页解释运行关系，不承载大型完整目录。
- reference 页承载完整文件清单和目录型索引。
- 新增源码锚点必须真实存在于 `dst-scripts`。
- 标题中的 BBC 编码必须保持 code span，不能裸写。

## `0x10054000` 结构细节

### `0x10054100` 数据结构与生命周期

#### `0x10054110` 具体 Lua 结构

##### `0x10054111` 需要核对的字段

- 非 reference 专题页必须有 H2/H3/H4/H5。
- 源码路径必须指向真实文件。
- 完整覆盖清单只维护在 `0x8000-reference`。
- 每个非 reference 专题页至少保留源码锚点表、Mermaid 流程图和 `rg` 查询块。
- `rg` 查询块应优先搜索函数名、类名、表名或状态机名。

## `0x10055000` 阅读路线

### `0x10055100` 从哪里开始读源码

~~~bash
rg -n "SaveGame|CheckMapSaveData|GetPersistData|ReplicateEntity" \
  dst-scripts/mainfunctions.lua \
  dst-scripts/worldgen_main.lua \
  dst-scripts/entityscript.lua \
  dst-scripts/entityreplica.lua
~~~

#### `0x10055110` 推荐顺序

##### `0x10055111` 最小闭环

先从源码锚点确认事实，再回到专题边界清理重复叙述。
