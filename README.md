# 岐黄万里 · DSH 插件合集

一个聚合 bundle，把六个插件装成一次。**插件代码不在这里**——本仓库只有一条 patch 和依赖声明，各插件仍在自己的仓库里独立开发、独立发版。

## 包含什么

| 行 id | 包 | 仓库 | 作用 |
| --- | --- | --- | --- |
| `dsh-baihua-bridge` | `baihua-dsh-plugin` | [luminsw/baihua-dsh-plugin](https://github.com/luminsw/baihua-dsh-plugin) | 百花桥 `/dsh-bridge/*`：家庭病历本、百花 MCP 中转 |
| `dsh-baihua-local-ai` | `baihua-local-ai-dsh-plugin` | [luminsw/baihua-local-ai-dsh-plugin](https://github.com/luminsw/baihua-local-ai-dsh-plugin) | 百花本地 AI（OpenVINO 小任务、状态卡片） |
| `dsh-hysteria-proxy` | `hysteria-dsh-plugin` | [luminsw/hysteria-dsh-plugin](https://github.com/luminsw/hysteria-dsh-plugin) | hysteria 代理与 auth-proxy 的启停/诊断 |
| `dsh-openvino` | `openvino-dsh-plugin` | [luminsw/openvino-dsh-plugin](https://github.com/luminsw/openvino-dsh-plugin) | OpenVINO 状态、模型扫描、基准与转换 |
| `dsh-dev-workbench` | `dsh-dev-workbench` | [luminsw/dsh-dev-workbench](https://github.com/luminsw/dsh-dev-workbench) | 多语种开发工作台：构建、真机安装、日志 |
| `icon-standard` | `@luminsw/icon-standard` | [luminsw/qihuang-icon-standard](https://github.com/luminsw/qihuang-icon-standard) | 三端图标标准：skill + blueprint/render/audit |

## 安装（新机器 / 一键部署）

在 DSH 里：

```
plugin_manager { action: "install_bundle", target: "github:luminsw/dsh-plugins" }
```

它会把本包和六个插件一起装进 profile，并把 `@luminsw/dsh-plugins` 写进 `dsh.profile.bundles`，patch 随之插入六行。

## ⚠️ 先卸掉单插件 bundle

聚合 patch 沿用了各插件**原来的行 id**（这样你按 id 覆盖 config 的写法继续有效）。所以同一个 id 不能插两次：

```bash
# 换成聚合包之前，逐个停用
plugin_manager { action: "set_bundle", target: "baihua-dsh-plugin", enabled: false }
# ……其余四个同理，然后再 install_bundle 聚合包
```

## 本机开发：保住 link 热改

装了聚合包之后，插件代码默认来自 GitHub 副本。要在本机边改边生效，**在 profile 的 `package.json` 里把六个包显式声明成 `link:` 直接依赖**（与聚合包并列）：

```json
{
  "dependencies": {
    "@luminsw/dsh-plugins": "link:C:/Users/lumin/src/dsh-plugins",
    "baihua-dsh-plugin": "link:C:/Users/lumin/src/baihua-dsh-plugin",
    "baihua-local-ai-dsh-plugin": "link:C:/Users/lumin/src/baihua-local-ai-dsh-plugin",
    "hysteria-dsh-plugin": "link:C:/Users/lumin/src/hysteria-dsh-plugin",
    "openvino-dsh-plugin": "link:C:/Users/lumin/src/openvino-dsh-plugin",
    "dsh-dev-workbench": "link:C:/Users/lumin/src/dsh-dev-workbench",
    "@luminsw/icon-standard": "link:C:/Users/lumin/game/dsh-plugins/icon-standard"
  }
}
```

然后 `dsh plugin --profile web install`，`node_modules` 里全是符号链接。

**为什么不用 `pnpm.overrides`**：实测无效。聚合包本身是 `link:` 时，它有自己的 `node_modules`，插件名会先在**聚合包自己的** `node_modules` 里命中（GitHub 副本），overrides 只作用于 profile 的安装图，管不到里面那层。用直接依赖让 profile 的解析先命中，才是可靠做法。

**为什么要显式声明**：只靠残留的旧符号链接也能碰巧工作，但一次 `pnpm prune` 就会静默回落到 GitHub 副本。判断实际加载的是哪一份，查它解析到的包目录：

```
cordis_inspect_query { provider: "Config", method: "listConfigs", input: { name: "@luminsw/icon-standard" } }
# packageDir 应指向你的工作树，而不是 …\src\dsh-plugins\node_modules\…
```

改了 `link:` 依赖指向的目录后，loader 会报 `restart-required`（`package-moved`），需要重启 DSH 才换新模块。

## 仓库可见性

两个仓库都是 public，安装不需要凭据：

- `@luminsw/dsh-plugins` — https://github.com/luminsw/dsh-plugins
- `@luminsw/icon-standard` — https://github.com/luminsw/qihuang-icon-standard

## 依赖锁定

`pnpm-lock.yaml` 入库，用来钉住六个插件各自的确切 commit——"这套组合装过、是好的"有据可查。改了某个插件后重新 `pnpm install` 刷新锁文件再提交。

## 维护

- 改包含哪些插件 → 改 `package.json` 的 `dependencies` + `cordis.patch.yml` 的 insert 行，两边要一致
- 改某个插件的默认配置 → 不要改这里，在 profile 的 `cordis.patch.yml` 里按行 id 覆盖 `config`
- 升级 → `plugin_manager { action: "install_bundle", target: "github:luminsw/dsh-plugins" }` 重装即可
