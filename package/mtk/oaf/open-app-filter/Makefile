
# OpenAppFilter (OAF) - vendored from https://github.com/destan19/OpenAppFilter
#
# WHY THIS IS VENDORED INTO THE TREE INSTEAD OF PULLED FROM A FEED:
#
# The "packages" feed shipped by Ruijie contains net/open-app-filter, an
# older 2025.04.17 snapshot of the very same upstream project.  It defines
# the same binary package names as the upstream master branch:
#
#   appfilter   (userspace daemon)   -> upstream 7.0.1
#   kmod-oaf    (kernel module)       -> upstream 6.6.133 / 7.0.1
#
# Because both sides use the same package directory name ("open-app-filter")
# and the same output package names, only one of them can end up in the
# image.  The build resolved to the Ruijie copy, so the image got:
#
#   /usr/bin/oafd                  -> 2025.04.17 (registers ubus object "appfilter")
#   /lib/modules/6.6.133/oaf.ko    -> 2025.04.17 (no "fwx" netlink protocol)
#   luci-app-oaf (web UI)          -> 7.0-r1   (calls ubus object "fwx")
#
# The UI therefore talked to a daemon that never existed: every ubus call
# to "fwx" failed, so the dashboard showed zero devices, the user list was
# empty, the feature library was empty and saving a device did nothing.
#
# Vendoring the upstream master sources under package/mtk/oaf/open-app-filter
# makes the local package override the feed copy (include/scan.awk puts
# local packages into the OVERRIDELIST when a feed provides the same
# directory name), so all three pieces - kernel module, daemon and web UI -
# come from the same upstream revision.
#
# The kernel module lives in oaf-src/ and is built by the same Makefile so
# that both binaries stay in one package directory and therefore override
# the feed copy as a unit.

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=open-app-filter
PKG_VERSION:=7.0.1
PKG_RELEASE:=2

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)
include $(INCLUDE_DIR)/package.mk

TARGET_CFLAGS +=-Werror=implicit-function-declaration

# ---------------------------------------------------------------------------
# kernel module: kmod-oaf
# ---------------------------------------------------------------------------
define KernelPackage/oaf
  SUBMENU:=Netfilter Extensions
  TITLE:=OAF netfilter module
  FILES:=$(PKG_BUILD_DIR)/kernel/oaf.ko
  DEPENDS:=+kmod-nf-conntrack +kmod-nf-reject
  AUTOLOAD:=$(call AutoLoad,0,oaf)
  KCONFIG:=CONFIG_NETFILTER
endef

define KernelPackage/oaf/description
  OAF netfilter module
endef

OAF_KCFLAGS := -Wno-error=missing-prototypes \
               -Wno-error=missing-declarations \
               -Wno-error=unused-variable \
               -Wno-error=declaration-after-statement \
               -Wno-error=implicit-fallthrough \
               -Wno-error=missing-braces \
               -Wno-error=parentheses \
               -Wno-error=format \
               -Wno-frame-larger-than

OAF_MAKE_OPTS := \
	$(KERNEL_MAKE_FLAGS) \
	M="$(PKG_BUILD_DIR)/kernel" \
	KCFLAGS="$(OAF_KCFLAGS)" \
	$(EXTRA_KCONFIG)

# ---------------------------------------------------------------------------
# userspace daemon: appfilter
# ---------------------------------------------------------------------------
define Package/appfilter
  SECTION:=Base system
  CATEGORY:=Base system
  DEPENDS:=+libubox +libubus +libuci +libpthread +libjson-c +libblobmsg-json +libuci-lua +libsqlite3 +libcurl +kmod-oaf
  TITLE:=OAF service
endef

define Package/appfilter/description
  OAF service
endef

define Package/appfilter/conffiles
/etc/fwxd/custom_feature.cfg
/etc/config/fwx
endef

# ---------------------------------------------------------------------------
# build
# ---------------------------------------------------------------------------
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)/kernel
	$(CP) ./src/* $(PKG_BUILD_DIR)/
	$(CP) ./oaf-src/* $(PKG_BUILD_DIR)/kernel/
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR)/ \
	CC="$(TARGET_CROSS)gcc" \
	CFLAGS="$(TARGET_CFLAGS)" \
	LIBS="$(TARGET_LDFLAGS) -lm -lpthread -lubox -luci -lubus -ljson-c -lblobmsg_json -lsqlite3 -lcurl" \
	all

	$(MAKE) -C "$(LINUX_DIR)" \
		$(OAF_MAKE_OPTS) \
		modules
endef

define Build/Compile/Default

endef

define Package/appfilter/install
	$(INSTALL_DIR) $(1)/usr/bin $(1)/etc/init.d
	$(INSTALL_DIR) $(1)/etc/fwxd
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_DIR) $(1)/usr/share/rpcd/acl.d/
	$(INSTALL_DATA) ./files/feature.bin $(1)/etc/fwxd/feature.bin
	$(INSTALL_DATA) ./files/custom_feature.cfg $(1)/etc/fwxd/custom_feature.cfg
	$(INSTALL_BIN) ./files/appfilter.init $(1)/etc/init.d/appfilter
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/oafd $(1)/usr/bin
	$(INSTALL_BIN) ./files/rule_manager.lua $(1)/usr/bin/rule_manager
	$(INSTALL_BIN) ./files/hnat.sh $(1)/usr/bin/hnat.sh
	$(INSTALL_DATA) ./files/oaf_version $(1)/etc/oaf_version
	$(INSTALL_DATA) ./files/luci-app-oaf.json $(1)/usr/share/rpcd/acl.d/
endef

$(eval $(call BuildPackage,appfilter))
$(eval $(call KernelPackage,oaf))
