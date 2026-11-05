# Actions-OpenWrt-FileBrowser

独立的 OpenWrt **feed**，提供完整的 [FileBrowser](https://github.com/filebrowser/filebrowser) 插件：

| 包名 | 作用 | 架构 |
| --- | --- | --- |
| `luci-app-filebrowser` | LuCI 界面 + UCI 配置 + procd 启动脚本 | `all` |
| `filebrowser-bin` | 官方预编译 `filebrowser` 可执行文件 → `/usr/bin/filebrowser` | `aarch64_generic` |

拆成两个仓库的目的：固件项目只负责「编译固件」，插件本身的修改、升级、
调默认值都在本仓库里做，改完推送，固件下次编译自动取最新，不需要再动固件项目。

---

## 一、为什么拆成两个包

上游 `luci-app-filebrowser` 只是界面，ipk 里**不带可执行文件**（官方二进制 33 MB，
作者刻意不集成）。不补一个二进制包，界面能显示但永远启动失败。

也不用 OpenWrt feeds 里的 `utils/filebrowser`：
- 它从源码编译，`Build/Compile` 里有 `npm ci` + `npm run build`，需要 `node/host`，
  构建慢且依赖 npm 网络；
- 它自己安装 `/etc/config/filebrowser` 与 `/etc/init.d/filebrowser`，
  会与 `luci-app-filebrowser` 的同名文件冲突。

---

## 二、在固件项目里启用（标准 feeds 方式）

本仓库是一个标准 feed，根目录下每个子目录就是一个包。在固件源码根目录执行：

```bash
# 1) 添加 feed —— 必须放在 feeds.conf 的第一行，见下面的「同名包冲突」
sed -i '1i src-git filebrowser https://github.com/sxml2015/Actions-OpenWrt-FileBrowser.git' \
    feeds.conf.default

# 2) 拉取并安装
./scripts/feeds update filebrowser
./scripts/feeds install -a -p filebrowser

# 3) 选中
#    CONFIG_PACKAGE_luci-app-filebrowser=y
#    CONFIG_PACKAGE_filebrowser-bin=y
```

`luci-app-filebrowser` 已声明 `DEPENDS:=+filebrowser-bin +luci-compat`，只选界面包也会自动
带上二进制包和 Lua 兼容层。

### 同名包冲突（25.12 必须看）

`coolsnowwolf/luci` 的 `openwrt-25.12` 分支自带一个同名包 `luci-app-filebrowser`
（stokito 的 JS 版文件管理器，菜单也是 `NAS -> File Browser`，界面为「文件浏览器」+ 删除按钮）。

`scripts/feeds` 对同名源包的规则是「`feeds.conf` 里**先出现者胜**」，后出现的静默跳过，
没有任何提示。因此：

- 本 feed 追加在末尾 → luci feed 的包先命中 → 固件里装的是别人的包，本插件不生效。
- 本 feed 必须在 `feeds.conf` 的第一行。

### 本项目的做法（工作流自动完成）

`aarch64/Lede2512/2512diy-part1.sh` 把 feeds 源插到 `feeds.conf.default` 第一行，
位于 `feeds update -a` 之前；`2512diy-part2.sh` 再兜底删掉 luci feed 的同名包链接。
`plugins.cfg` 的「自定义追加区」把三个包改成 `on` 即可。

---

## 三、单独升级 / 修改 FileBrowser

### 升级二进制版本

只改 `filebrowser-bin/Makefile` 两处：

```make
PKG_VERSION:=2.63.23          # 改成新版本号（不带 v）
PKG_HASH:=c55b3450…2fd27a     # 取自 release API 的 digest 字段
```

取 hash：

```bash
curl -s https://api.github.com/repos/filebrowser/filebrowser/releases/latest \
  | grep -A3 'linux-arm64-filebrowser.tar.gz' | grep digest
```

`digest` 形如 `sha256:xxxx`，去掉前缀即为 `PKG_HASH`。官方**不发布** `.sha256` 附件。

### 改默认配置 / 启动行为

- 默认值：`luci-app-filebrowser/root/etc/config/filebrowser`
- 启动逻辑：`luci-app-filebrowser/root/etc/init.d/filebrowser`
- 界面：`luci-app-filebrowser/luasrc/**`

注意 `/etc/config/filebrowser` 是 conffile，设备上装过旧版时 sysupgrade 会保留
旧配置，新的默认值只在全新刷机时生效。

---

## 四、本仓库对上游做的四处修正

1. **默认可执行目录** `/tmp` → `/usr/bin`
   原值指向 tmpfs，重启即清空，每次开机都得重新下载 33 MB，实际不可用。

2. **启动脚本改为 procd**
   原脚本 `start()` 用 `&` 丢后台（进程死了没人拉起），`stop()` 用 `kill -9`
   （不给进程写数据库的机会）。现在 `USE_PROCD=1` + `respawn 3600 5 5`。

3. **补可执行文件校验**
   二进制不存在时明确写日志并拒绝启动，而不是静默失败。

4. **补挂载点校验**
   `root_path` 以 `/mnt/` 开头时先查 `/proc/mounts`，盘没挂上就拒绝启动 ——
   否则 FileBrowser 会把文件写进 rootfs 把系统盘塞满。

盘是后挂的（开机时比 filebrowser 晚）：确认挂载后执行
`/etc/init.d/filebrowser restart` 即可。

---

## 五、已知限制

- 只提供 **arm64**（`aarch64_generic`）二进制，官方仅发布 `linux-arm64`。
  其他架构需要自行替换 `PKG_SOURCE` 与 `PKG_HASH`。
- 压缩包内是扁平结构（无顶层目录），`Build/Prepare` 不能用 `--strip-components`。
- 界面包 `LUCI_PKGARCH:=all`，界面仍是 Lua CBI（`luasrc/`），24.10 以后的 LuCI 是 JS 前端，
  所以依赖 `luci-compat`（并自动带入 `luci-lua-runtime`）。这一条已写进 Makefile，
  不再依赖固件里其他 Lua 插件带来的间接依赖。

---

## 六、许可

- `luci-app-filebrowser` 部分：Apache-2.0，原作者 Lienol / xiaozhuai。
- `filebrowser` 二进制：Apache-2.0，来自 [filebrowser/filebrowser](https://github.com/filebrowser/filebrowser)。
