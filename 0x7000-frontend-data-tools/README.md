# `0x70000000` 前端数据与工具

本区把前端栈、HUD、制作界面、静态数据、媒体特效和调试工具放在同一条阅读线上。
阅读时先判断 Lua 文件是在消费输入、维护 screen 栈、展示数据、注册数据，还是请求权威端改变实体。

目录级语义由本 README 承载，独立专题文件只解释具体运行链路。

## `0x70001000` 区域定位

### `0x70001100` 读者要解决的问题

#### `0x70001110` 运行时入口与数据入口

##### `0x70001111` 验证点

`dst-scripts/frontend.lua` 和 `dst-scripts/input.lua` 是前端运行入口。
`dst-scripts/tuning.lua`、`dst-scripts/recipes.lua`、`dst-scripts/strings.lua` 是数据入口。
`dst-scripts/fx.lua`、`dst-scripts/skin_assets.lua`、`dst-scripts/screens/redux/scrapbookdata.lua` 更接近注册表或展示素材。

### `0x70001200` 本区边界

#### `0x70001210` 不把 UI 误读成权威 Gameplay

##### `0x70001211` 边界条件

HUD 和 widget 可以消费输入、播放声音、打开 screen、改变本地 UI 状态，或通过 replica / `playercontroller` 请求远端制作。
真正改变世界的权威逻辑通常仍落在 server component、`playercontroller` RPC 或 prefab 行为里。

## `0x70002000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/input.lua` | `Input:OnControl` | 先让 `TheFrontEnd` 消费输入 |
| `dst-scripts/frontend.lua` | `FrontEnd` | 维护 `screenstack`、焦点、fade 和 debug panel |
| `dst-scripts/screens/playerhud.lua` | `PlayerHud` | 游戏内 HUD screen |
| `dst-scripts/widgets/widget.lua` | `Widget` | UI 树、焦点和 `OnControl` 递归 |
| `dst-scripts/widgets/controls.lua` | `Controls` | HUD 控件集合 |
| `dst-scripts/components/playercontroller.lua` | `PlayerController` | 地图控制、放置模式和远端制作请求 |
| `dst-scripts/recipe.lua` | `Recipe2` | 配方对象与 `AllRecipes` |
| `dst-scripts/recipes.lua` | `Recipe2(...)` | 官方配方注册 |
| `dst-scripts/tuning.lua` | `TUNING` | 数值常量与 modifier |
| `dst-scripts/strings.lua` | `STRINGS` | 文本表 |
| `dst-scripts/translator.lua` | `TranslateStringTable` | 递归替换文本表 |
| `dst-scripts/skin_assets.lua` | `skin_assets` | 皮肤资产列表 |
| `dst-scripts/screens/redux/scrapbookdata.lua` | generated table | Scrapbook 展示数据 |
| `dst-scripts/fx.lua` | `fx` table | 通用 FX 数据表 |
| `dst-scripts/prefabs/fx.lua` | `MakeFx` | 把 `fx` table 变成 prefab |
| `dst-scripts/util.lua` | `DebugSpawn` | 调试生成实体 |
| `dst-scripts/consolecommands.lua` | `c_` functions | 控制台命令入口 |

### `0x70002100` 锚点读取顺序

#### `0x70002110` 从输入到展示

##### `0x70002111` 搜索信号

先读 `Input:OnControl`、`FrontEnd:OnControl`、`FrontEnd:PushScreen`。
再读 `Widget:OnControl`、`PlayerHud:OnControl` 和 `Controls:ToggleMap`。

### `0x70002200` 从数据到展示

#### `0x70002210` 配方、文本和素材

##### `0x70002211` 验证点

数据页应先找注册点，再找读取方。
例如 `Recipe2` 写入 `AllRecipes`，`CraftingMenuHUD:RebuildRecipes` 生成 `valid_recipes`，制作 UI 再按搜索、过滤和详情面板展示。

## `0x70003000` 运行关系图

~~~mermaid
flowchart TD
    A["engine input callback"]
    A --> B["TheInput:OnControl"]
    B --> C["TheFrontEnd:OnControl"]
    C --> D["top screen OnControl"]
    D --> E["Widget focus tree"]
    D --> F["PlayerHud shortcuts"]
    B --> G["Input event handlers when UI did not consume"]
    F --> H["Controls / crafting menu / map / chat"]
    H --> I["replica builder or playercontroller request"]
    I --> K["server component / RPC / placement"]
    J["TUNING / STRINGS / Recipe2 / fx table"]
    J --> H
~~~

### `0x70003100` 关系图读法

#### `0x70003110` 前端消费优先

##### `0x70003111` 边界条件

如果 `TheFrontEnd:OnControl` 返回 `true`，输入不会继续派发到 `Input.oncontrol`。
这也是排查 UI 挡住 gameplay 操作时最先验证的分叉。

### `0x70003200` 数据只解释展示

#### `0x70003210` 数据表与执行方分离

##### `0x70003211` 验证点

`TUNING`、`STRINGS`、`skin_assets`、`scrapbookdata` 自身不执行 gameplay。
需要顺着读取方追到 component、prefab、screen、widget 或 `playercontroller` 请求入口。

## `0x70004000` 目录索引

### `0x70004100` README 载体

#### `0x70004110` 二级目录

##### `0x70004111` 链接校验

以下入口先进入目录 README，再进入具体专题文件。

- [Frontend UI](0x7100-frontend-ui/README.md)
- [Frontend 与输入](0x7100-frontend-ui/0x7101-frontend-input.md)
- [Screens Widgets HUD](0x7100-frontend-ui/0x7102-screens-widgets-hud.md)
- [Crafting UI](0x7100-frontend-ui/0x7103-crafting-ui.md)
- [数据媒体与工具](0x7200-data-media-tools/README.md)
- [Tuning 与 Recipes](0x7200-data-media-tools/0x7201-tuning-recipes.md)
- [Localization Skins Scrapbook](0x7200-data-media-tools/0x7202-localization-skins-scrapbook.md)
- [媒体 FX 与 Audio](0x7200-data-media-tools/0x7203-media-fx-audio.md)
- [Tools Debug](0x7200-data-media-tools/0x7204-tools-debug.md)

## `0x70005000` 阅读与验证路线

### `0x70005100` 从哪里开始读源码

~~~bash
rg -n "function Input:OnControl|function FrontEnd:OnControl|function FrontEnd:PushScreen" \
  dst-scripts/input.lua dst-scripts/frontend.lua

rg -n "function PlayerHud:OnControl|function Controls:ToggleMap|IsMapControlsEnabled|function DoRecipeClick" \
  dst-scripts/screens/playerhud.lua dst-scripts/widgets/controls.lua \
  dst-scripts/components/playercontroller.lua dst-scripts/widgets/widgetutil.lua

rg -n "Recipe2\\(|TranslateStringTable|function DebugSpawn|local fx =" \
  dst-scripts/recipes.lua dst-scripts/translator.lua dst-scripts/util.lua dst-scripts/fx.lua
~~~

#### `0x70005110` 最小闭环

##### `0x70005111` 抽样动作

抽样一条 `CONTROL_MAP`。
从 `Input:OnControl` 追到 `PlayerHud:OnControl`，再追到 `Controls:ToggleMap`、`IsMapControlsEnabled` 和 `TheFrontEnd:PushScreen`。
抽样一条制作按钮。
从 `CraftingMenuDetails:_MakeBuildButton` 追到 `DoRecipeClick`、`replica.builder:MakeRecipeFromMenu` 和 `playercontroller:RemoteMakeRecipeFromMenu`。
