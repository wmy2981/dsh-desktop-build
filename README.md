# dsh-desktop-build

**非官方项目。与 DeepSeek 没有任何关系，也不代表官方立场。**

本仓库只包含一条 GitHub Actions 工作流：定时拉取 [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)
的官方源码，编译桌面端（`apps/desktop`），产出 Windows 安装程序，上传为工作流产物并创建对应的发行版。

## 它做什么

1. 查询上游仓库最新的发行版（含预发布），得到版本号、标签与其对应的提交。
2. 如果本仓库已有该版本号的发行版，直接跳过编译与发行。
3. 完整克隆上游仓库，检出该发行版对应的提交。
4. 按官方仓库自身的构建脚本编译 Windows x64 安装程序。
5. 上传构建产物到 GitHub Actions artifacts（保留 30 天），创建发行版并附带
   安装程序与 `SHA256SUMS.txt`。

触发方式：

- 定时任务：每 2 小时一次（UTC）。
- 手动触发：`workflow_dispatch`，可选 `force` 以重建已存在的版本。

## 安装程序

- 文件名：`deepseek-harness-<版本>-win-x64-unsigned.exe`（NSIS 安装程序）。
- **未签名**：本仓库不持有代码签名证书，无法执行官方发布流程中的签名与时间戳步骤。
  因此 Windows SmartScreen 会对安装程序给出警告。
- **自动更新不可用**：无签名构建不配置更新源，应用内的「检查更新」不可用。
- 由于沿用官方的应用标识 `com.deepseek.harness`，本安装程序与官方安装程序会被
  Windows 视为同一产品：安装其一会影响另一个，且两者共用同一用户数据目录。

## 构建是否与官方一致

源码与编译方式保持一致：

- 不修改上游任何源码文件（`source/` 目录内只有一处新增的 `apps/desktop/.env.windows`
  发布设置文件，该文件在上游被 `.gitignore` 忽略，属于官方打包脚本要求的本地配置）。
- 使用官方仓库自身的打包脚本 `package:win:x64:unsigned`、锁定的依赖版本
  （`pnpm-lock.yaml`、`scripts/primary-runtime/lock.json`）与官方 CI 相同的
  Node 24 / pnpm 11.7.0。
- 构建元数据记录上游提交号，可回溯到确切的源码版本。

与官方发布流程的差异仅有两处，且都由「没有证书」导致：不签名，以及运行时不配置更新源。

## 免责声明

- 上游项目为 MIT 许可，本仓库同样以 MIT 许可发布，见 [LICENSE](LICENSE)。
- 安装程序中包含的上游代码、名称、图标与商标归其各自权利人所有。
- 构建产物按「原样」提供，不提供任何担保。使用者自行承担风险。

## 手动触发

在 Actions 页面选择 **Build DeepSeek Harness Windows installer**，点击 *Run workflow* 即可。
勾选 `force` 会在必要时先删除同名旧发行版再重新编译。
