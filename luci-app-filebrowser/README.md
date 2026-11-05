# luci-app-filebrowser（本 feed 维护版）

FileBrowser 的 LuCI 界面 + UCI 配置 + procd 启动脚本。
可执行文件由同 feed 的 **filebrowser-bin** 包提供（装到 `/usr/bin/filebrowser`），
不再需要手动下载 33 MB 二进制。

- 上游界面来源：`sxml/luci-app-filebrowser`（源自 Lienol / xiaozhuai）
- 本版相对上游的改动见仓库根目录 `README.md` 第四节
- 默认端口 `8088`，默认根目录 `/root`，默认不自动启动（`enable '0'`）
- 需要 `luci-compat`（Lede 默认已选）

完整说明、升级方式、构建接线请看仓库根目录 [`README.md`](../README.md)。
