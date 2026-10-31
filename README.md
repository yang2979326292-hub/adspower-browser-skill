# adspower-browser-skill

一个 AI Agent skill：让 豆包工作Agent 通过本机 AdsPower 客户端的 Local API 管理指纹浏览器，包括打开、关闭、新建、修改、删除、查询环境，以及分组、标签、代理、团队成员、内核下载等。

改写自 AdsPower 官方的 [AdsPower/adspower-browser](https://github.com/AdsPower/adspower-browser)（MIT）。原版通过 npm 包 `adspower-browser` 调用；这个版本改成一个 PowerShell 7 脚本，直接请求 Local API，不需要 Node.js。命令名和参数与原 CLI 一致。

## 目录结构

```
skills/adspower-browser/
├── SKILL.md                  # Agent 读取的说明：调用方式、命令速查、执行规则
├── scripts/adspower-api.ps1  # 调用 Local API 的脚本（41 个命令）
├── references/               # 各命令的完整参数与枚举值（16 份）
└── LICENSE.txt
```

## 安装

用 skills CLI 安装：

```bash
npx skills add https://github.com/yang2979326292-hub/adspower-browser-skill --skill adspower-browser
```

也可以手动安装：把 `skills/adspower-browser` 整个文件夹复制到你的 Agent 的 skill 目录下。

## 使用前准备

1. 安装并启动 AdsPower 客户端，在 设置 → Local API 中开启（默认端口 50325）。
2. 安装 PowerShell 7（`pwsh`）。Windows 自带的 Windows PowerShell 5.1 不支持。
3. 设置 API Key（在 AdsPower 的 Local API 设置页可以看到）：

   ```powershell
   [Environment]::SetEnvironmentVariable('ADS_API_KEY', '<你的 API Key>', 'User')
   ```

   端口不是 50325 时，再设置 `ADS_API_PORT`。

## 手动调用示例

```powershell
$ads = '.\skills\adspower-browser\scripts\adspower-api.ps1'
& $ads check-status                         # 检查 Local API 是否可用
& $ads get-browser-list                     # 列出环境（默认每页 200 条）
& $ads open-browser 3                       # 打开编号为 3 的环境
& $ads delete-browser '{"profile_id":["id1"]}' -DryRun   # 只预览请求，不执行
& $ads -ListCommands                        # 列出全部命令
```

## 和原版的区别

- 不依赖 Node.js 和 npm 包，只需要 PowerShell 7。
- 保留了原 CLI 的行为：简写参数、`get-browser-list` 默认 `limit=200`、新建环境时按本机系统补 UA、回收站恢复的参数转换。
- 新增：触发限流时自动重试（最多 4 次）、`-DryRun` 预览、`download-kernel -Poll` 跟踪下载进度，以及明确的退出码（0 成功 / 1 接口错误 / 2 用法或连接错误）。
- 不包含页面自动化（点击、输入、截图）。需要的话请用原仓库的 local-api-mcp。

## 已知限制

- `get-dynamic-proxy-traffic` 在 AdsPower Global 8.4.3 上返回 404，需要更新的客户端。
- 只在 Windows 11 + PowerShell 7.6 + AdsPower Global 8.4.3 上实测过。macOS、Linux 的代码路径（系统识别、环境变量读取）写了但没有实测。

## 许可证

MIT，见 [LICENSE](LICENSE)。原作者版权声明保留在 LICENSE 与 `skills/adspower-browser/LICENSE.txt` 中。
