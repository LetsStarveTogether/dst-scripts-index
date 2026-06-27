# `0x10020000` 源码快照

当前跟踪的 `dst-scripts` 文件为 4045 个，其中 Lua 文件为 4030 个。

当前子模块快照为 `f731c442`，对应上游提交信息 `739495`。

上一轮文档统计对应 `281c027b`，对应上游提交信息 `736959`。

本轮净增 17 个 Lua 文件，其中 `dst-scripts/components/` 净增 6 个，`dst-scripts/prefabs/` 净增 11 个。

## `0x10021111` 本页定位 / 要回答的运行时问题 / 源码阅读目标 / 验证点

读者要先知道源码规模，再决定先读运行链路还是进入 reference 清单。

本页统计口径固定为 `git ls-files --recurse-submodules dst-scripts`。

## `0x10022000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/mainfunctions.lua` | `LoadScript` / `RunScript` | 运行时脚本缓存与执行 |
| `dst-scripts/worldgen_main.lua` | `LoadScript` / `RunScript` / `GenerateNew` | 世界生成脚本缓存与生成入口 |
| `dst-scripts/prefabs.lua` | `Prefab = Class` | Prefab 对象定义入口 |
| `dst-scripts/entityscript.lua` | `AddComponent` | 组件挂载入口 |

### `0x10022111` 主锚点 / `dst-scripts/mainfunctions.lua` / 搜索信号

先在 `dst-scripts/mainfunctions.lua` 搜索 `LoadScript` 与 `RunScript`。

再在 `dst-scripts/worldgen_main.lua` 搜索同名函数，确认 worldgen 使用独立加载上下文。

## `0x10023000` 运行流程

~~~mermaid
flowchart TD
    A["git ls-files"]
    A --> B["目录聚合"]
    B --> C["运行时专题"]
    C --> D["reference 覆盖"]
~~~

### `0x10023111` 流程分段 / 入口到副作用 / 边界条件

- 跟踪文件总数包含 15 个非 Lua 文件。
- 非 Lua 文件包括 `dst-scripts/.github/workflows/update.yml`、`controller.vdf`、13 个语言 `.po` 或 `.pot` 文件。
- 目录统计只用于分配阅读注意力，不替代 reference 覆盖清单。

## `0x10024111` 结构细节 / 数据结构与生命周期 / 具体 Lua 结构 / 需要核对的字段

| 范围 | Lua 文件数 | 阅读含义 |
| --- | ---: | --- |
| `dst-scripts/` 根层 | 218 | 启动、全局服务和数据入口 |
| `dst-scripts/prefabs/` | 1594 | 实体装配最大区 |
| `dst-scripts/components/` | 821 | server 侧行为状态主区 |
| `dst-scripts/stategraphs/` | 261 | 动作表现与动画状态机 |
| `dst-scripts/brains/` | 191 | AI 决策入口 |
| `dst-scripts/behaviours/` | 29 | behaviour tree 节点 |
| `dst-scripts/map/` | 444 | worldgen、布局与世界定义 |
| `dst-scripts/widgets/` | 272 | HUD 与 UI 组件 |
| `dst-scripts/screens/` | 135 | 前端 screen |
| `dst-scripts/scenarios/` | 50 | 场景脚本 |
| `dst-scripts/util/` | 8 | 小型工具模块 |
| `dst-scripts/languages/` | 2 | 语言加载工具 |
| `dst-scripts/nis/` | 2 | 过场脚本 |
| `dst-scripts/tools/` | 2 | 维护和导出工具 |
| `dst-scripts/cameras/` | 1 | camera Lua 实现 |

## `0x10025100` 阅读与验证路线 / 从哪里开始读源码

~~~bash
git ls-files --recurse-submodules dst-scripts
git ls-files --recurse-submodules dst-scripts | rg "\.lua$" | wc -l
git ls-files --recurse-submodules dst-scripts/prefabs | rg "\.lua$" | wc -l
git ls-files --recurse-submodules dst-scripts/components | rg "\.lua$" | wc -l
~~~

### `0x10025111` 推荐顺序 / 最小闭环

先复算总数和两个最大目录，再决定专题页是否应该改成解释型页面或 reference 清单页面。

如果 `git ls-files` 结果变化，必须同步更新 `0x8000-reference` 的完整覆盖清单。
