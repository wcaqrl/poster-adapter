# Poster 的懒猫应用适配仓库

这个仓库只存放懒猫微服应用包配置和审核资料。海报网页及 Docker 镜像由独立的 [Poster](https://github.com/wcaqrl/poster) 仓库负责；自动发现镜像、转存、打包、送审由 [lazycat-action](https://github.com/wcaqrl/lazycat-action) 负责。不要把这里的配置放进 Poster 源码仓库。

## 版本与镜像

`poster` 在推送 `vX.Y.Z` Git 标签后，由其 `publish-image.yml` 构建 `linux/amd64` 的公开镜像 `ghcr.io/wcaqrl/poster:X.Y.Z`。本仓库每天北京时间 11:17 的 GitHub Actions 定时任务会读取 GHCR 上符合稳定 SemVer 规则的最高标签，比较该平台镜像摘要与已提交的 `.lazycat-action.lock.yml`。只有发现新镜像或尚未完成提交的版本才继续。

当前 `package.yml` 和 `lzc-manifest.yml` 中的 0.1.1 是已知的旧版本基线；首次成功运行会发现 0.1.2（或当时最新的稳定版本），把官方转存后的镜像地址和新版本写回本仓库。不要手工改动锁文件状态或把未送审的版本标记为 `submitted`。

自动流程：发现 GHCR 新摘要 → 调用官方 copy-image 接口 → 更新 `lzc-manifest.yml` 与 `package.yml` → `lzc-cli` 构建、验证 `.lpk` → 将版本、镜像和锁文件推送回本仓库 → 用开发者 PAT 提交懒猫官方应用商店审核 → 提交完成状态。审核期间自动流程会暂停。生成的 `.lpk` 会附在对应的 Actions 运行记录中，不加入 Git。

## 一次性设置

1. 在 **poster-adapter** 仓库的 **Settings → Secrets and variables → Actions → Repository secrets** 添加 `LZC_API_TOKEN`，值为开发者平台 PAT。不要将 PAT 写入 YAML、提交到 Git 或设置到 Poster 源码仓库。可选 `LZC_API_HOST`，不填时使用工具默认的生产应用商店服务端。
2. 在 **poster-adapter** 的 **Settings → Actions → General → Workflow permissions** 选择 **Read and write permissions**，以便工作流把打包后的版本、官方镜像地址及锁文件提交回 `main`。若 `main` 受分支保护，需要允许 `github-actions[bot]` 提交或采用适合该保护规则的更新策略。
3. 确认 **lazycat-action** 的 `v1.3.0` release 和浮动 `v1` 标签已发布且对应新版源码。适配仓库的复用工作流引用 `wcaqrl/lazycat-action/.github/workflows/lazycat.yml@v1`；此 release 发布前不要用正式模式运行。
4. 将这里的文件推送到 `poster-adapter/main` 时，配置文件变更会自动运行一次 **dry-run**；也可以在 **Actions → Update Poster for LazyCat → Run workflow** 保持默认的 **dry-run** 勾选，验证是否发现最新版本。确认后取消勾选并手动运行一次，才会转存、打包和送审。之后每天自动检查。定时任务只在默认分支运行，GitHub 可能延迟调度。

Poster 的发布使用自己的 GitHub Actions：修改源代码并推送 `main` 会刷新供开发测试的 `edge` 镜像；需要正式发布时还要创建并推送新的 `vX.Y.Z` 标签。适配仓库只监视稳定版本镜像标签，不会因 `edge` 更新而送审。

本地只检查（依赖 `lazycat-action` 命令行及 Docker/网络）：

```bash
lazycat-action run --operation check --config lazycat-action.yml --dry-run
```

`lzc-build.yml`、`package.yml`、`lzc-manifest.yml`、图标与截图属于懒猫应用包；`lazycat-action.yml` 描述版本监控与商店发布。`dist/` 下是生成物，受 `.gitignore` 忽略。
