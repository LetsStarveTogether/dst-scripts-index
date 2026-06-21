# `0x33000000` Prefab 装配

本目录解释 prefab 从声明、注册、生成到网络投影的运行契约。

它不复制 `dst-scripts/prefabs/` 的完整清单。

完整清单统一进入 `0x8000-reference`。

## `0x33001000` 目录职责

### `0x33001100` 文档边界

#### `0x33001110` 目录载体

##### `0x33001111` 验证点

本 README 只说明目录定位、子页面边界和推荐入口。

具体源码行为进入本目录下的独立专题文件。

不要让第一个独立文件替代目录 README。

#### `0x33001120` 覆盖口径

##### `0x33001121` 验证点

`dst-scripts/prefabs/` 的全量文件覆盖由 `0x8201-prefab-catalog.md` 承担。

`dst-scripts/components/*_replica.lua` 的全量文件覆盖由 `0x8202-component-catalog.md` 承担。

本目录只维护运行链路、关键源码锚点和容易误读的边界。

## `0x33002000` 子页面索引

### `0x33002100` 推荐顺序

#### `0x33002110` 从声明到网络投影

##### `0x33002111` 链接校验

- [Prefab 装配契约](0x3301-prefab-contract.md)
- [Replica 与 Classified](0x3302-replica-classified.md)

## `0x33003000` 阅读入口

### `0x33003100` 最小路径

#### `0x33003110` 先定位再展开

##### `0x33003111` 抽样动作

先读 `0x3301-prefab-contract.md`。

确认 `Prefab` 对象、`LoadPrefabFile`、`RegisterPrefabsImpl` 和 `SpawnPrefabFromSim` 的闭环。

再读 `0x3302-replica-classified.md`。

确认 `components/*_replica.lua`、`prefabs/*_classified.lua`、netvars 和 `Network:SetClassifiedTarget` 的关系。

如果要查完整路径，回到 `0x8000-reference/README.md`。

## `0x33004000` 当前核对结论

### `0x33004100` 与源码一致的事实

#### `0x33004110` 目录布局

##### `0x33004111` 验证点

源码没有 dst-scripts/replica 目录。

内建 replica 文件位于 `dst-scripts/components/*_replica.lua`。

classified 文件位于 `dst-scripts/prefabs/*_classified.lua`。

prefab helper 位于 `dst-scripts/prefabutil.lua` 和 `dst-scripts/standardcomponents.lua`。

#### `0x33004120` 数量口径

##### `0x33004121` 验证点

`entityreplica.lua` 当前列出 19 个内建可复制组件。

`dst-scripts/components` 当前有 19 个 tracked `_replica.lua` 文件。

`dst-scripts/prefabs` 当前有 15 个 tracked `_classified.lua` 文件。
