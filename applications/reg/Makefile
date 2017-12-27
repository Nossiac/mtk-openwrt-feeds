#
# hua.shao@mediatek.com
#
# MTK Property Software.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=reg
PKG_RELEASE:=1

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)
PKG_CONFIG_DEPENDS:=

PKG_MAINTAINER:=Hua Shao <nossiac@163.com>

include $(INCLUDE_DIR)/package.mk

define Package/reg
  SECTION:=MTK Properties
  CATEGORY:=MTK Properties
  DEPENDS:=
  TITLE:=todo
  SUBMENU:=Applications
endef

define Package/reg/description
  todo
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

TARGET_CFLAGS += -Wall
TARGET_CFLAGS += -Wno-error=format-security 

define Package/reg/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/reg $(1)/usr/bin
endef


$(eval $(call BuildPackage,reg))
