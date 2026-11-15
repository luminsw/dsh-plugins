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
| `icon-standard` | `@local/icon-standard` | [luminsw/qihuang-icon-standard](https://github.com/luminsw/qihuang-icon-standard) | 三端图标标准：skill + blueprint/render/audit |

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

这台机器上六个插件都是 `link:C:\Users\lumin\src\*`，改代码即时生效。装了聚合包会把依赖换成 GitHub 副本，热改就失效。

解决办法是在 profile 的 `package.json` 加 `pnpm.overrides`，把要热改的包指回本地目录（**已实测可用**，其余仍走 GitHub）：

```json
{
  "pnpm": {
    "overrides": {
      "baihua-dsh-plugin": "link:C:/Users/lumin/src/baihua-dsh-plugin",
      "openvino-dsh-plugin": "link:C:/Users/lumin/src/openvino-dsh-plugin"
    }
  }
}
```

`pnpm install` 后 `node_modules` 里会是符号链接：

```
baihua-dsh-plugin  0.1.0  <- ..\..\..\src\baihua-dsh-plugin
```

## 私有仓库说明

本仓库是私有仓库，`qihuang-icon-standard` 也是私有。因此安装本包需要该机器上有 GitHub 凭据（`gh auth login` 或 git credential helper 里存好 token），否则拉取会失败。

## 依赖锁定

`pnpm-lock.yaml` 入库，用来钉住六个插件各自的确切 commit——"这套组合装过、是好的"有据可查。改了某个插件后重新 `pnpm install` 刷新锁文件再提交。

## 维护

- 改包含哪些插件 → 改 `package.json` 的 `dependencies` + `cordis.patch.yml` 的 insert 行，两边要一致
- 改某个插件的默认配置 → 不要改这里，在 profile 的 `cordis.patch.yml` 里按行 id 覆盖 `config`
- 升级 → `plugin_manager { action: "install_bundle", target: "github:luminsw/dsh-plugins" }` 重装即可
