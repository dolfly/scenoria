# scenoria releases

scenoria 的**发布产物镜像**。源码仓库是私有的，这里只放可以公开分发的产物：

- 各平台的 garble 混淆二进制（`scenoria-<os>-<arch>-<version>.gz` / `.zip`）
- `SHA256SUMS`，覆盖同一个 release 里的每个文件

**没有源码**：这个仓里没有 scenoria 的代码、没有源码 tarball/zip，也没有构建脚本。
安装脚本 `install.sh` 只用于把产物装到机器上。

> 本仓由 scenoria 私有仓库的 CI 自动同步，手工提交会被覆盖。问题请回到私有仓库跟踪。

## 安装

产物公开，**不需要 GitHub 账号、token 或登录**：

```bash
# 最新版本（装到 /usr/local/bin/scenoria，需要 root）
sudo bash <(curl -fsSL https://github.com/dolfly/scenoria/raw/main/install.sh) --install

# 指定版本
sudo bash <(curl -fsSL https://github.com/dolfly/scenoria/raw/main/install.sh) --install v0.0.2

# 交互选择版本
sudo bash <(curl -fsSL https://github.com/dolfly/scenoria/raw/main/install.sh)
```

脚本按 `uname -s`/`uname -m` 选产物，下载后先比对 `SHA256SUMS` 再安装；校验不通过不会落盘。
非 root 或想装到别处时用 `SCENORIA_DEST`：

```bash
SCENORIA_DEST="$HOME/.local/bin" bash install.sh --install v0.0.2
```

## 手工下载与校验

```bash
tag=v0.0.2
curl -fsSLO "https://github.com/dolfly/scenoria/releases/download/${tag}/SHA256SUMS"
curl -fsSLO "https://github.com/dolfly/scenoria/releases/download/${tag}/scenoria-linux-amd64-0.0.2.gz"
sha256sum -c --ignore-missing SHA256SUMS   # 只需要其中一个产物时可以这样校验
gzip -dc scenoria-linux-amd64-0.0.2.gz > scenoria && chmod +x scenoria
./scenoria   # 起服务后 GET /healthz → {"code":200,...}
```

## 容器镜像

```
ghcr.io/dolfly/scenoria:0.0.2   (= :latest)
```

镜像与 release 附件同源（garble + `-trimpath`，构建阶段过纯净度门禁）。镜像是**公开**
package，`docker pull ghcr.io/dolfly/scenoria:0.0.2` 不需要登录。

## 平台

| 平台 | 产物 |
| --- | --- |
| linux/amd64 | `scenoria-linux-amd64-<ver>.gz` |
| linux/arm64 | `scenoria-linux-armv8-<ver>.gz` |
| linux/armv7 | `scenoria-linux-armv7-<ver>.gz` |
| linux/386 | `scenoria-linux-386-<ver>.gz` |
| darwin/amd64 · darwin/arm64 | `scenoria-darwin-{amd64,arm64}-<ver>.gz` |
| windows/amd64 · windows/arm64 | `scenoria-windows-{amd64,arm64}-<ver>.zip` |

## 运行

scenoria 是 `api.toonflow.net` 官方接口的 1:1 自建替代服务，默认以本地实现（`SCENORIA_MODE=selfhost`）运行，无需 `SCENORIA_BASE_URL`；只有回退到透明代理模式（`SCENORIA_MODE=proxy`）时才需要配置它。其余配置见私有仓库 README：

```bash
./scenoria
curl -i http://127.0.0.1:8080/healthz
# → {"code":200,"data":{"status":"ok"},"message":"success"}
```
