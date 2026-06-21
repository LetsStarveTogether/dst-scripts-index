# `0x71010000` Frontend 与输入

本页解释输入进入 Lua 后，为什么会先被前端 screen 栈尝试消费。
它也说明 HUD 如何在 screen 体系中处理地图、聊天、暂停和制作入口。

## `0x71011000` 本页定位

### `0x71011100` 要回答的运行时问题

#### `0x71011110` 输入先到哪里

##### `0x71011111` 验证点

`Input:OnControl` 位于 `dst-scripts/input.lua`。
它先调用 `TheFrontEnd:OnControl(control, digitalvalue)`。
只有当前端没有消费输入时，才继续触发 `self.oncontrol:HandleEvent(...)`。

### `0x71011200` 前端能消费什么

#### `0x71011210` Screen 栈与 Debug 快捷键

##### `0x71011211` 边界条件

`FrontEnd:OnControl` 会把 `CONTROL_PRIMARY` 映射为 `CONTROL_ACCEPT` 给顶部 screen。
它也处理 `CONTROL_OPEN_DEBUG_CONSOLE`、`CONTROL_OPEN_DEBUG_MENU`、console log 和 debug render 切换。

## `0x71012000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/input.lua` | `Input:OnControl` | 设备输入进入 Lua 的分发点 |
| `dst-scripts/frontend.lua` | `FrontEnd:OnControl` | 顶部 screen 的控制消费点 |
| `dst-scripts/frontend.lua` | `FrontEnd:PushScreen` | screen 入栈、聚焦和激活 |
| `dst-scripts/frontend.lua` | `FrontEnd:PopScreen` | screen 出栈、销毁和恢复焦点 |
| `dst-scripts/widgets/screen.lua` | `Screen:OnBecomeActive` | 设置 UI root 并恢复焦点 |
| `dst-scripts/widgets/widget.lua` | `Widget:OnControl` | 只向有焦点的子 widget 递归 |
| `dst-scripts/screens/playerhud.lua` | `PlayerHud:OnControl` | 游戏内 HUD 快捷键入口 |
| `dst-scripts/components/playercontroller.lua` | `PlayerController:ShouldPlayerHUDControlBeIgnored` | HUD 输入是否应直接吞掉 |

### `0x71012100` 主锚点

#### `0x71012110` `dst-scripts/input.lua`

##### `0x71012111` 搜索信号

先搜 `function Input:OnControl`。
然后确认 `not TheFrontEnd:OnControl(...)` 这个分叉。

### `0x71012200` 前端栈锚点

#### `0x71012210` `dst-scripts/frontend.lua`

##### `0x71012211` 验证点

`FrontEnd:PushScreen` 会把 screen 加入 `self.screenstack`。
它会调用旧顶部 screen 的 `OnBecomeInactive`，再调用新 screen 的 `OnBecomeActive`。
入栈后还会执行一次 `FrontEnd:Update(0)`，让新 screen 立即刷新焦点和布局。

## `0x71013000` 运行流程

~~~mermaid
flowchart TD
    A["engine control callback"]
    A --> B["TheInput:OnControl"]
    B --> C{"TheFrontEnd:OnControl consumed?"}
    C -->|yes| D["top Screen / focused Widget handled it"]
    C -->|no| E["Input oncontrol handlers"]
    D --> F["PlayerHud:OnControl when HUD is active screen"]
    F --> G["pause / map / chat / crafting shortcuts"]
    F --> I["playercontroller ignore check"]
    G --> H["TheFrontEnd:PushScreen or HUD state change"]
~~~

### `0x71013100` 输入分段

#### `0x71013110` `Input:OnControl`

##### `0x71013111` 边界条件

鼠标 primary 和 secondary control 会受 `self.mouse_enabled` 限制。
如果前端消费成功，`Input.oncontrol` 事件不会被调用。

### `0x71013200` 前端分段

#### `0x71013210` `FrontEnd:OnControl`

##### `0x71013211` 验证点

顶部 screen 的 `OnControl` 返回 `true` 时，前端会阻止后续输入派发。
这类问题通常表现为 HUD、弹窗或文本框截断了 gameplay 操作。
如果 `FrontEnd:IsControlsDisabled()` 为真，前端会返回 `false`，但不会让顶部 screen 消费输入。
文本输入框还有 `textProcessorWidget` 分支，鼠标点击其它位置时会先退出强制文本输入。

### `0x71013300` HUD 分段

#### `0x71013310` `PlayerHud:OnControl`

##### `0x71013311` 验证点

`PlayerHud:OnControl` 会处理 `CONTROL_PAUSE`、`CONTROL_MAP`、聊天开关和制作 pin 翻页。
它还会调用 `playercontroller:ShouldPlayerHUDControlBeIgnored` 判断某些输入是否应被 HUD 吞掉。
地图按键只在 `CONTROL_MAP` 松开时触发 `Controls:ToggleMap`。
制作入口在 `CONTROL_OPEN_CRAFTING` 按下时打开或关闭 `CraftingMenuHUD`。

## `0x71014000` 结构细节

### `0x71014100` `FrontEnd` 结构

#### `0x71014110` `screenstack`

##### `0x71014111` 需要核对的字段

`screenstack` 是 screen 生命周期的核心。
`PushScreen` 会 `AddChild`、`MoveToFront`、设置默认焦点，并执行一次 `Update(0)`。
`PopScreen` 会 `OnBecomeInactive`、`OnDestroy`、`RemoveChild`，再恢复新的顶部 screen。
如果被弹出的不是当前顶部 screen，`PopScreen(screen)` 会从 `screenstack` 中移除指定 screen。

### `0x71014200` `Input` 结构

#### `0x71014210` 事件分发器

##### `0x71014211` 需要核对的字段

`Input` 内部维护 `oncontrol`、`onmousebutton`、`onkeydown` 等事件集合。
这些事件不是前端 screen 栈的一部分。
它们只在前端没有消费或对应输入路径允许时继续派发。

### `0x71014300` `PlayerHud` 结构

#### `0x71014310` HUD 是 Screen

##### `0x71014311` 需要核对的字段

`PlayerHud` 继承自 `Screen`。
它把 `Controls(self.owner)` 加到 `self.root`，再由 `Controls` 聚合背包、状态、地图、制作菜单和 toast。

## `0x71015000` 阅读与验证路线

### `0x71015100` 从哪里开始读源码

~~~bash
rg -n "function Input:OnControl|not TheFrontEnd:OnControl|self.oncontrol:HandleEvent" \
  dst-scripts/input.lua

rg -n "function FrontEnd:OnControl|function FrontEnd:PushScreen|function FrontEnd:PopScreen" \
  dst-scripts/frontend.lua

rg -n "function PlayerHud:OnControl|ShouldPlayerHUDControlBeIgnored|CONTROL_MAP|CONTROL_PAUSE|CONTROL_TOGGLE_SAY" \
  dst-scripts/screens/playerhud.lua dst-scripts/components/playercontroller.lua
~~~

#### `0x71015110` 推荐顺序

##### `0x71015111` 最小闭环

先追 `CONTROL_MAP`。
预期路径是 `Input:OnControl`、`FrontEnd:OnControl`、`PlayerHud:OnControl`、`Controls:ToggleMap`。
如果地图 screen 已打开，最终应看到 `TheFrontEnd:PopScreen`。
如果地图 screen 未打开，并且 `IsMapControlsEnabled()` 与 game mode 条件允许，最终应看到 `TheFrontEnd:PushScreen(MapScreen(self.owner))`。
