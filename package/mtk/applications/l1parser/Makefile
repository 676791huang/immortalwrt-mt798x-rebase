include $(TOPDIR)/rules.mk

PKG_NAME:=l1parser
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL=https://github.com/chasey-dev/l1parser.git
PKG_SOURCE_DATE:=2025-12-11
PKG_SOURCE_VERSION:=081bb31211efc74594d25bfd1bb5811f3408a205
# 上游的 PKG_MIRROR_HASH 是按其本机 tar/zstd 版本算出的压缩包字节哈希；
# 我们的构建环境压缩出的字节不同（zstd 版本差异），会导致
#   "Hash of the local file ... does not match" → 下载被删 → 编译失败。
# 源码提交号仍然钉死（PKG_SOURCE_VERSION），这里跳过压缩包字节校验。
PKG_MIRROR_HASH:=skip

PKG_LICENSE:=LGPL-2.1
PKG_MAINTAINER:=chasey-dev <ellenyoung0912@gmail.com>

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/l1parser/Default
  SECTION:=MTK Properties
  CATEGORY:=MTK Properties
  SUBMENU:=Applications
  TITLE:=MediaTek L1 Profile Parser Toolset (C++)
endef

define Package/libl1parser
  $(call Package/l1parser/Default)
  TITLE+= (Library)
  DEPENDS:=+libstdcpp
endef

define Package/l1util
  $(call Package/l1parser/Default)
  TITLE+= (CLI Tool)
  DEPENDS:=+libl1parser
endef

define Package/ucode-mod-l1parser
  $(call Package/l1parser/Default)
  TITLE+= (Ucode Binding)
  DEPENDS:=+libl1parser +libucode
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/l1parser.h $(1)/usr/include/
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libl1parser.so $(1)/usr/lib/
endef

define Package/libl1parser/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libl1parser.so $(1)/usr/lib/
endef

define Package/l1util/install
	$(INSTALL_DIR) $(1)/sbin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/l1util $(1)/sbin/
	$(LN) l1util $(1)/sbin/l1dat
endef

define Package/ucode-mod-l1parser/install
	$(INSTALL_DIR) $(1)/usr/lib/ucode
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/ucode/l1parser.so $(1)/usr/lib/ucode/
endef

$(eval $(call BuildPackage,libl1parser))
$(eval $(call BuildPackage,l1util))
$(eval $(call BuildPackage,ucode-mod-l1parser))