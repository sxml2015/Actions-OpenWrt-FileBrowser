# ==============================================================================
#  filebrowser-bin —— FileBrowser 官方预编译二进制包
#
#  本包只做一件事：把官方 linux-arm64 tar.gz 里的 filebrowser 可执行文件
#  装到 /usr/bin/filebrowser。不装配置、不装启动脚本 ——
#  那两样由同仓库的 luci-app-filebrowser 提供，避免同名文件冲突。
#
#  为什么需要单独一个包：
#    luci-app-filebrowser 只提供 LuCI 界面，ipk 里不带可执行文件
#    （官方二进制 33 MB，界面作者刻意不集成）。没有本包，界面能出来
#    但永远启动不了。
#
#  为什么不用 feeds 里的 utils/filebrowser：
#    · 它从源码编译，Build/Compile 里有 npm ci + npm run build，需要
#      node/host，构建慢且依赖 npm 网络。
#    · 它自己装 /etc/config/filebrowser 与 /etc/init.d/filebrowser，
#      与 luci-app-filebrowser 的同名文件冲突。
#
#  升版本只需改两处：PKG_VERSION、PKG_HASH。
#  PKG_HASH 取自
#    https://api.github.com/repos/filebrowser/filebrowser/releases/latest
#  对应 linux-arm64-filebrowser.tar.gz 资产的 digest 字段（去掉 "sha256:" 前缀）。
#
#  已知事实（实测 2026-11-05，v2.63.23）：
#    · 压缩包内是扁平结构：CHANGELOG.md / LICENSE / README.md / filebrowser
#      → 不能用 --strip-components，直接 -C 解到 PKG_BUILD_DIR。
#    · filebrowser 二进制 34,537,634 字节，静态链接，musl 可直接运行。
# ==============================================================================

include $(TOPDIR)/rules.mk

PKG_NAME:=filebrowser-bin
PKG_VERSION:=2.63.23
PKG_RELEASE:=1

PKG_SOURCE:=linux-arm64-filebrowser.tar.gz
PKG_SOURCE_URL:=https://github.com/filebrowser/filebrowser/releases/download/v$(PKG_VERSION)
PKG_HASH:=c55b3450b6ac07ef73b5b49f0b2955a8e755100b71af1df73f9e5d25b12fd27a

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKGARCH:=aarch64_generic

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE

include $(INCLUDE_DIR)/package.mk

define Package/filebrowser-bin
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=FileBrowser web file manager (official prebuilt binary, arm64)
  URL:=https://github.com/filebrowser/filebrowser
  DEPENDS:=
endef

define Package/filebrowser-bin/description
  FileBrowser provides a web interface to manage files inside a directory:
  upload, delete, preview, rename and edit.

  This package installs only the official prebuilt linux-arm64 binary to
  /usr/bin/filebrowser. The LuCI front end, the UCI config and the init
  script are provided by luci-app-filebrowser in the same feed.
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(TAR) -xzf $(DL_DIR)/$(PKG_SOURCE) -C $(PKG_BUILD_DIR)
endef

define Build/Compile
	# 无需编译：二进制已由 Build/Prepare 解出
endef

define Package/filebrowser-bin/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/filebrowser $(1)/usr/bin/filebrowser
endef

$(eval $(call BuildPackage,filebrowser-bin))
