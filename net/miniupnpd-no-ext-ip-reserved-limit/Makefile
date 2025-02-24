#
# Copyright (C) 2006-2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=miniupnpd-no-ext-ip-reserved-limit
PKG_VERSION:=2.3.7
PKG_RELEASE:=99

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/kilesduli/miniupnp
PKG_SOURCE_VERSION:=d7fdb5679e856ab7acd54f0af9b9c69a4096ce87
PKG_MIRROR_HASH:=5e9b1bb99ecd5d1d3cc8f3e2c7a67d6f9934a77736a3180de27540d606da587a

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
  DEPENDS:= \
	+libcap-ng \
	+libmnl \
	+libuuid
  PROVIDES:=miniupnpd
  TITLE:=Lightweight UPnP IGD & PCP/NAT-PMP daemon
  URL:=https://github.com/kilesduli/miniupnp
endef

define Package/miniupnpd-iptables
  $(call Package/miniupnpd-no-ext-ip-reserved-limit/Default)
  DEPENDS+= \
	+IPV6:ip6tables \
	+IPV6:libip6tc \
	+iptables \
	+libip4tc \
	+libnetfilter-conntrack
  TITLE+= (iptables)
  VARIANT:=iptables
endef

define Package/miniupnpd-nftables
  $(call Package/miniupnpd-no-ext-ip-reserved-limit/Default)
  DEPENDS+= \
	+libnftnl
  TITLE+= (nftables)
  VARIANT:=nftables
  DEFAULT_VARIANT:=1
  CONFLICTS:=miniupnpd-iptables
endef

define Package/miniupnpd-no-ext-ip-reserved-limit/conffiles/Default
/etc/config/upnpd
endef

Package/miniupnpd-iptables/conffiles = $(Package/miniupnpd-no-ext-ip-reserved-limit/conffiles/Default)
Package/miniupnpd-nftables/conffiles = $(Package/miniupnpd-no-ext-ip-reserved-limit/conffiles/Default)

define Build/Prepare
	$(call Build/Prepare/Default)
	$(FIND) $(PKG_BUILD_DIR) -mindepth 1 -maxdepth 1 ! -name "miniupnpd" -exec $(RM) -rf {} +
	$(CP) $(PKG_BUILD_DIR)/miniupnpd/* $(PKG_BUILD_DIR)
	$(RM) -r $(PKG_BUILD_DIR)/miniupnpd
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

define Package/miniupnpd-iptables/install
	$(call Package/miniupnpd-no-ext-ip-reserved-limit/install/Default,$1)
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_DIR) $(1)/usr/share/miniupnpd
	$(INSTALL_BIN) ./files/miniupnpd.defaults.iptables $(1)/etc/uci-defaults/99-miniupnpd
	$(INSTALL_DATA) ./files/firewall3.include $(1)/usr/share/miniupnpd/firewall.include
endef

define Package/miniupnpd-nftables/install
	$(call Package/miniupnpd-no-ext-ip-reserved-limit/install/Default,$1)
	$(INSTALL_DIR) $(1)/usr/share/nftables.d
	$(CP) ./files/nftables.d/* $(1)/usr/share/nftables.d/
endef

$(eval $(call BuildPackage,miniupnpd-iptables))
$(eval $(call BuildPackage,miniupnpd-nftables))
