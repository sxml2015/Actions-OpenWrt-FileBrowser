# Copyright (C) 2018-2020 Lienol <lawlienol@gmail.com>
#
# This is free software, licensed under the Apache License, Version 2.0 .
#
# Improve by xiaozhuai <xiaozhuai7@gmail.com>
#
# Maintained in this feed (Actions-OpenWrt-FileBrowser) together with
# filebrowser-bin, so the whole plugin can be updated by editing one repo.
#

include $(TOPDIR)/rules.mk

LUCI_TITLE:=LuCI support for FileBrowser
LUCI_PKGARCH:=all
# luci-compat 提供 Lua CBI 兼容层（并自动带入 luci-lua-runtime）。
# OpenWrt 24.10 / 25.12 的 LuCI 是 JS 前端，本包仍是 luasrc 结构，
# 不显式依赖 luci-compat 时，若固件里没有其他 Lua 插件，页面会打不开。
LUCI_DEPENDS:=+filebrowser-bin +luci-compat

include $(TOPDIR)/feeds/luci/luci.mk

# call BuildPackage - OpenWrt buildroot signature
