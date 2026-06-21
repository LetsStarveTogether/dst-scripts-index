# `0x71020000` Screen Widget HUD

Screen、Widget 和 HUD 是 DST Lua 前端的三层结构。
Screen 管页面生命周期，Widget 管树、焦点和输入递归，PlayerHud 把游戏内控件集中到 `Controls`。

## `0x71021000` 本页定位

### `0x71021100` 要回答的运行时问题

#### `0x71021110` 一个 UI 控件如何接收输入

##### `0x71021111` 验证点

输入先到 `FrontEnd:OnControl`。
然后进入顶部 screen 的 `OnControl`。
`Widget:OnControl` 只会递归给当前有 `focus` 的子 widget。

### `0x71021200` HUD 如何挂到 Screen 栈

#### `0x71021210` `PlayerHud` 与 `Controls`

##### `0x71021211` 边界条件

`PlayerHud` 是 `Screen` 子类。
它不只是显示状态条，也负责协调 map、chat、crafting、wardrobe、scrapbook 等 screen 的打开和关闭。

## `0x71022000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/widgets/screen.lua` | `Screen` | 继承 `Widget` 的页面基类 |
| `dst-scripts/widgets/screen.lua` | `Screen:OnBecomeActive` | 设置 UI root 并恢复焦点 |
| `dst-scripts/widgets/widget.lua` | `Widget:AddChild` | 建立 UI 树 |
| `dst-scripts/widgets/widget.lua` | `Widget:OnControl` | 焦点子树递归处理输入 |
| `dst-scripts/widgets/widget.lua` | `Widget:SetFocus` | 焦点流转入口 |
| `dst-scripts/screens/playerhud.lua` | `PlayerHud` | 游戏内 HUD screen |
| `dst-scripts/widgets/controls.lua` | `Controls` | HUD widget 聚合根 |
| `dst-scripts/widgets/controls.lua` | `Controls:ToggleMap` | map screen 打开和关闭 |
| `dst-scripts/components/playercontroller.lua` | `PlayerController:IsMapControlsEnabled` | 地图 screen 是否允许被 HUD 打开 |

### `0x71022100` 主锚点

#### `0x71022110` `dst-scripts/widgets/widget.lua`

##### `0x71022111` 搜索信号

先搜 `function Widget:OnControl`。
如果没有看到 `if not self.focus then return false end`，就不要判断具体控件会收到输入。

### `0x71022200` HUD 锚点

#### `0x71022210` `dst-scripts/screens/playerhud.lua`

##### `0x71022211` 验证点

`PlayerHud:CreateOverlays` 创建 overlay。
`PlayerHud` 构造后会把 `Controls(self.owner)` 加入 `self.root`。

## `0x71023000` 运行流程

~~~mermaid
flowchart TD
    A["FrontEnd screenstack top"]
    A --> B["Screen:OnControl"]
    B --> C["Widget:OnControl"]
    C --> D{"child has focus?"}
    D -->|yes| E["child:OnControl"]
    D -->|no| F["return false"]
    A --> G["PlayerHud when in-game HUD is active"]
    G --> H["Controls"]
    H --> I["MapScreen / crafting menu / inventory / toast"]
~~~

### `0x71023100` Widget 输入递归

#### `0x71023110` `Widget:OnControl`

##### `0x71023111` 边界条件

Widget 不会向所有子节点广播 control。
只有 `v.focus` 为真且子节点返回 `true` 时，控制才被消费。

### `0x71023200` Screen 生命周期

#### `0x71023210` `Screen:OnBecomeActive`

##### `0x71023211` 验证点

Screen 重新激活时会调用 `TheSim:SetUIRoot(self.inst.entity)`。
如果有 `last_focus` 并且实体仍有效，会恢复旧焦点。
否则会尝试 `default_focus`。

### `0x71023300` HUD 控件集合

#### `0x71023310` `Controls`

##### `0x71023311` 验证点

`Controls` 构造函数会创建 action hint、toast、状态显示、背包和 map/crafting 相关根节点。
`Controls:ToggleMap` 根据 `quagmire` game mode、`no_minimap` 和 `IsMapControlsEnabled()` 决定打开 `MapScreen` 或 `QuagmireRecipeBookScreen`。

## `0x71024000` 结构细节

### `0x71024100` UI 树

#### `0x71024110` `Widget:AddChild`

##### `0x71024111` 需要核对的字段

`AddChild` 把子 widget 放入父节点并维护层级关系。
后续位置、缩放、显示、更新和输入都依赖这棵树。

### `0x71024200` HUD 打开 Screen

#### `0x71024210` `Controls:ToggleMap`

##### `0x71024211` 需要核对的字段

地图不是 `Controls` 自己绘制的一个普通子 widget。
它会通过 `TheFrontEnd:PushScreen(MapScreen(self.owner))` 进入前端 screen 栈。

### `0x71024300` HUD 展示与 Gameplay 的边界

#### `0x71024310` `PlayerHud:OnControl`

##### `0x71024311` 边界条件

`PlayerHud:IsCraftingBlockingGameplay` 已经 deprecated，并且在当前源码里直接返回 `false`。
HUD 是否吞掉输入要看 `PlayerHud:OnControl`、当前 screen 栈和 `playercontroller:ShouldPlayerHUDControlBeIgnored`。
但 HUD 不应该被当作 server 权威状态来源。
遇到 gameplay 结果要继续追 component 或 RPC。

## `0x71025000` 阅读与验证路线

### `0x71025100` 从哪里开始读源码

~~~bash
rg -n "function Widget:AddChild|function Widget:OnControl|function Widget:SetFocus" \
  dst-scripts/widgets/widget.lua

rg -n "function Screen:OnBecomeActive|function Screen:SetDefaultFocus" \
  dst-scripts/widgets/screen.lua

rg -n "function PlayerHud:OnControl|self.controls = self.root:AddChild|function Controls:ToggleMap" \
  dst-scripts/screens/playerhud.lua dst-scripts/widgets/controls.lua

rg -n "function PlayerController:IsMapControlsEnabled" \
  dst-scripts/components/playercontroller.lua
~~~

#### `0x71025110` 推荐顺序

##### `0x71025111` 最小闭环

从一个 `CONTROL_MAP` 开始。
确认 `PlayerHud:OnControl` 触发 `self.controls:ToggleMap()`。
再确认 `Controls:ToggleMap` 先检查 game mode、`no_minimap` 和 `IsMapControlsEnabled()`。
最终路径才会对 `TheFrontEnd` 调用 `PushScreen` 或 `PopScreen`。
