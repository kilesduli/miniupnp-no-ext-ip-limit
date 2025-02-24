#
# Copyright (C) 2006-2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=miniupnpd-no-ext-ip-reserved-limit
PKG_VERSION:=2.3.7
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/kilesduli/miniupnp
PKG_SOURCE_VERSION:=fd4c50358ab6e67b16be736621ea42478808d9ee
PKG_MIRROR_HASH:=90d308c16f52793424d45852d289d327d46ca01273d4b674e6c60ec4c00eae1f

PKG_MAINTAINER:=
PKG_LICENSE:=BSD-3-Clause
PKG_LICENSE_FILES:=LICENSE
PKG_CPE_ID:=cpe:/a:miniupnp_project:miniupnpd

PKG_INSTALL:=1
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/version.mk

define Package/miniupnpd-no-ext-ip-reserved-limit/Default
  SECTION:=net
  CATEGORY:=Network
  DEPENDS:= \
	+libcap-ng \
	+libmnl \
	+libuuid
  PROVIDES:=miniupnpd-no-ext-ip-reserved-limit
  CONFLICTS:=miniupnpd
  TITLE:=Lightweight UPnP IGD & PCP/NAT-PMP daemon
  SUBMENU:=Firewall
  URL:=https://github.com/kilesduli/miniupnp
endef

define Package/miniupnpd-no-ext-ip-reserved-limit-iptables
  $(call Package/miniupnpd-no-ext-ip-reserved-limit/Default)
  DEPENDS+= \
	+IPV6:ip6tables \
	+IPV6:libip6tc \
	+iptables \
	+libip4tc \
	+libnetfilter-conntrack
  TITLE+= (iptables)
  CONFLICTS:=miniupnpd-iptables
  VARIANT:=iptables
endef

define Package/miniupnpd-no-ext-ip-reserved-limit-nftables
  $(call Package/miniupnpd-no-ext-ip-reserved-limit/Default)
  DEPENDS+= \
	+libnftnl
  TITLE+= (nftables)
  VARIANT:=nftables
  DEFAULT_VARIANT:=1
  CONFLICTS:=miniupnpd-no-ext-ip-reserved-limit-iptables miniupnpd-nftables
endef

define Package/miniupnpd-no-ext-ip-reserved-limit/conffiles/Default
/etc/config/upnpd
endef

Package/miniupnpd-no-ext-ip-reserved-limit-iptables/conffiles = $(Package/miniupnpd-no-ext-ip-reserved-limit/conffiles/Default)
Package/miniupnpd-no-ext-ip-reserved-limit-nftables/conffiles = $(Package/miniupnpd-no-ext-ip-reserved-limit/conffiles/Default)

define Build/Prepare
	$(call Build/Prepare/Default)
	echo "$(VERSION_NUMBER)" | tr '() ' '_' >$(PKG_BUILD_DIR)/os.openwrt
endef

CONFIGURE_ARGS = \
	$(if $(CONFIG_IPV6),--ipv6) \
	--igd2 \
	--leasefile \
	--portinuse \
	--firewall=$(BUILD_VARIANT) \
	--disable-fork \
	--regex

TARGET_CFLAGS += $(FPIC)
TARGET_LDFLAGS += -Wl,--gc-sections,--as-needed

ifeq ($(BUILD_VARIANT),iptables)
	TARGET_CFLAGS += -flto
endif

define Package/miniupnpd-no-ext-ip-reserved-limit/install/Default
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_DIR) $(1)/etc/hotplug.d/iface
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/sbin/miniupnpd $(1)/usr/sbin/miniupnpd
	$(INSTALL_BIN) ./files/miniupnpd.init $(1)/etc/init.d/miniupnpd
	$(INSTALL_CONF) ./files/upnpd.config $(1)/etc/config/upnpd
	$(INSTALL_DATA) ./files/miniupnpd.hotplug $(1)/etc/hotplug.d/iface/50-miniupnpd
endef

define Package/miniupnpd-no-ext-ip-reserved-limit-iptables/install
	$(call Package/miniupnpd-no-ext-ip-reserved-limit/install/Default,$1)
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_DIR) $(1)/usr/share/miniupnpd
	$(INSTALL_BIN) ./files/miniupnpd.defaults.iptables $(1)/etc/uci-defaults/99-miniupnpd
	$(INSTALL_DATA) ./files/firewall3.include $(1)/usr/share/miniupnpd/firewall.include
endef

define Package/miniupnpd-no-ext-ip-reserved-limit-nftables/install
	$(call Package/miniupnpd-no-ext-ip-reserved-limit/install/Default,$1)
	$(INSTALL_DIR) $(1)/usr/share/nftables.d
	$(CP) ./files/nftables.d/* $(1)/usr/share/nftables.d/
endef

$(eval $(call BuildPackage,miniupnpd-no-ext-ip-reserved-limit-iptables))
$(eval $(call BuildPackage,miniupnpd-no-ext-ip-reserved-limit-nftables))
