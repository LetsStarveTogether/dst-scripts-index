# `0x72030000` 媒体 FX 与 Audio

媒体页解释动画、特效和音频如何从 Prefab、StateGraph 或 debug 命令触发。
核心阅读顺序是 `fx.lua -> prefabs/fx.lua -> SpawnPrefab(...) / SoundEmitter:PlaySound(...)`。

## `0x72031000` 本页定位

### `0x72031100` 要回答的运行时问题

#### `0x72031110` 源码阅读目标

##### `0x72031111` 验证点

目标是从视觉结果回到 `AnimState`、`SoundEmitter`、fx prefab 或触发它的 `FrameEvent`。
`dst-scripts/fx.lua` 多数情况下只是 FX 定义表。
`dst-scripts/prefabs/fx.lua` 才把定义表变成可生成的 prefab。

## `0x72032000` 源码锚点

| 文件 | 入口 | 用途 |
| --- | --- | --- |
| `dst-scripts/fx.lua` | `local fx` / `return fx` | 特效定义表 |
| `dst-scripts/prefabs/fx.lua` | `MakeFx` | 把 FX 定义变成 prefab |
| `dst-scripts/prefabs/fx.lua` | `startfx` | client 侧创建真实显示实体 |
| `dst-scripts/debugsounds.lua` | `SoundEmitter.SoundDebug` | 音频调试包装 |
| `dst-scripts/consolecommands.lua` | `c_sounddebug` / `c_sounddebugui` | 加载声音调试 |
| `dst-scripts/mainfunctions.lua` | `require("debugsounds")` | debug 音频启动路径 |
| `dst-scripts/stategraphs/SGwilson.lua` | `SoundEmitter:PlaySound` | SG 中按帧触发声音 |
| `dst-scripts/stategraphs/SGwilson.lua` | `SpawnPrefab(...)` | SG 中按帧触发 FX |

### `0x72032100` 主锚点

#### `0x72032110` `dst-scripts/fx.lua`

##### `0x72032111` 搜索信号

先在 `dst-scripts/fx.lua` 搜索 `local fx =`、`name =`、`bank =`、`build =`、`anim =` 和 `return fx`。
这个文件描述效果参数，不直接代表行为发生。

### `0x72032200` Prefab 锚点

#### `0x72032210` `dst-scripts/prefabs/fx.lua`

##### `0x72032211` 验证点

`MakeFx(t)` 会为 `fx.lua` 中每个定义创建 `Prefab(t.name, fn, assets)`。
`fn()` 先创建带 network 的 proxy 实体。
非 dedicated client 再延迟一帧进入 `startfx()`，创建带 `AnimState` 的本地显示实体。

### `0x72032300` Sound 锚点

#### `0x72032310` `dst-scripts/debugsounds.lua`

##### `0x72032311` 验证点

`debugsounds.lua` 会包装 `SoundEmitter.PlaySound`、`KillSound`、`KillAllSounds`、`SetParameter` 和 `SetVolume`。
`c_sounddebug()` 与 `c_sounddebugui()` 会按需 `require "debugsounds"`。

## `0x72033000` 运行流程

~~~mermaid
flowchart TD
    A["fx.lua defines table entries"]
    A --> B["prefabs/fx.lua MakeFx"]
    B --> C["network proxy prefab"]
    C --> D["client startfx local entity"]
    D --> E["AnimState bank/build/anim"]
    D --> F["optional SoundEmitter"]
    E --> G["animover / animqueueover Remove"]
    H["StateGraph FrameEvent"]
    H --> I["SpawnPrefab(fx name)"]
    H --> J["SoundEmitter:PlaySound"]
    I --> C
    K["c_sounddebug / mainfunctions"]
    K --> L["debugsounds wraps SoundEmitter"]
~~~

### `0x72033100` FX 表阶段

#### `0x72033110` `name`、`bank`、`build`、`anim`

##### `0x72033111` 边界条件

`name` 是可 `SpawnPrefab` 的 prefab 名。
`bank`、`build` 和 `anim` 只说明动画资源如何播放。
`sound`、`sound2`、`sounddelay`、`fn`、`fntime` 会改变生成后的副作用。

### `0x72033200` Proxy Prefab 阶段

#### `0x72033210` Server 与 Client 分工

##### `0x72033211` 验证点

`fn()` 创建的 proxy 带 `Network`，并加 `FX` tag。
dedicated server 不会生成本地显示实体。
master sim 会在 1 秒后移除 proxy。

### `0x72033300` StateGraph 触发阶段

#### `0x72033310` `FrameEvent`

##### `0x72033311` 搜索信号

在 `SGwilson.lua` 中同时搜索 `FrameEvent`、`SpawnPrefab` 和 `SoundEmitter:PlaySound`。
这能定位声音或特效出现在动画第几帧。

### `0x72033400` Debug 音频阶段

#### `0x72033410` `SoundEmitter.SoundDebug`

##### `0x72033411` 边界条件

声音调试包装会记录 callstack、position、looping sound 和 UI sound。
它是观察层，不改变 gameplay 权威逻辑。

## `0x72034000` 结构细节

### `0x72034100` 数据结构与生命周期

#### `0x72034110` 具体 Lua 结构

##### `0x72034111` 需要核对的字段

`fx.lua` 是大表，表项通常包含 `name`、`bank`、`build`、`anim`、`sound`、`fn`。
`prefabs/fx.lua` 的通用工厂会添加 `AnimState`，必要时再添加 `SoundEmitter`。
`startfx()` 创建的是 non-networked 本地显示实体，并通过 `animover` 或 `animqueueover` 清理。
`SGwilson.lua` 大量使用 `FrameEvent` 在指定动画帧调用 `SoundEmitter:PlaySound` 或 `SpawnPrefab`。
`debugsounds.lua` 会包装 `SoundEmitter` 方法，但由 console/debug 入口显式加载。
媒体专题只追触发链路，完整文件清单放 reference。

## `0x72035000` 阅读与验证路线

### `0x72035100` 从哪里开始读源码

~~~bash
rg -n "local fx =|return fx|name =|bank =|build =|anim =|sound =|fn =" \
  dst-scripts/fx.lua

rg -n "MakeFx|startfx|AddNetwork|AddAnimState|AddSoundEmitter|animover|animqueueover" \
  dst-scripts/prefabs/fx.lua

rg -n "FrameEvent|SpawnPrefab|SoundEmitter:PlaySound" \
  dst-scripts/stategraphs/SGwilson.lua

rg -n "c_sounddebug|c_sounddebugui|SoundEmitter.SoundDebug|require\\(\"debugsounds\"\\)" \
  dst-scripts/consolecommands.lua \
  dst-scripts/debugsounds.lua \
  dst-scripts/mainfunctions.lua
~~~

#### `0x72035110` 推荐顺序

##### `0x72035111` 最小闭环

抽样 `splash` 或 `die_fx`。
先在 `fx.lua` 看表项。
再到 `prefabs/fx.lua` 看 `MakeFx` 如何生成 prefab。
随后用 `rg "SpawnPrefab\\(\"splash"` 或 `rg "die_fx"` 找触发者。
如果是声音问题，先开 `c_sounddebugui()` 的源码路径，再回到触发 `PlaySound` 的 SG 或 prefab。
