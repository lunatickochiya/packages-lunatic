# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=node

ifeq ($(CONFIG_NODEJS_12),y)
PKG_VERSION:=v12.14.0
PKG_RELEASE:=1
PKG_HASH:=088a217ba2af641b8cc15be29f6e2956b8a33e6badb85596bbc2cdea9df9be71
PATCH_DIR:=./patches/v12.x
else
ifeq ($(CONFIG_NODEJS_10),y)
PKG_VERSION:=v10.18.0
PKG_RELEASE:=1
PKG_HASH:=0b2662ab849713dea708d3f74fe5e63fe2d0e869d1dc06b9e9f178ed1c66fede
PATCH_DIR:=./patches/v10.x
else
PKG_VERSION:=v8.17.0
PKG_RELEASE:=1
PKG_HASH:=5b0d96db482b273f0324c299ead86ecfbc5d033516e5fc37c92cfccb933ef6ff
PATCH_DIR:=./patches/v8.x
endif
endif

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_SOURCE_URL:=https://nodejs.org/dist/$(PKG_VERSION)

PKG_MAINTAINER:=Hirokazu MORIKAWA <morikw2@gmail.com>
PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE
PKG_CPE_ID:=cpe:/a:nodejs:node.js

HOST_BUILD_DEPENDS:=!NODEJS_12:python/host NODEJS_12:python3/host
HOST_BUILD_PARALLEL:=1

PKG_BUILD_DEPENDS:=!NODEJS_12:python/host NODEJS_12:python3/host NODEJS_12:node/host
PKG_BUILD_PARALLEL:=1
PKG_INSTALL:=1
PKG_USE_MIPS16:=0

OW_VER:=$(shell git --git-dir=$(TOPDIR)/.git describe --abbrev=0 --always --tag)

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/node
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=Node.js is a platform built on Chrome's JavaScript runtime
  URL:=https://nodejs.org/
  DEPENDS:=@!arc @!armeb @!powerpc \
           +libstdcpp +USE_UCLIBC:libpthread +USE_UCLIBC:librt +zlib \
           $(if $(filter reboot v19.07%, $(OW_VER)),+libopenssl,+NODEJS_8:libopenssl) \
           +NODEJS_ICU_SYSTEM:icu +NODEJS_ICU_SYSTEM:icu-full-data \
           +!NODEJS_8:libatomic
endef

define Package/node/description
  Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses
  an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js'
   package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
endef

define Package/node-npm
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=NPM stands for Node Package Manager
  URL:=https://www.npmjs.com/
  DEPENDS:=+node
endef

define Package/node-npm/description
 NPM is the package manager for Node.js
endef

define Package/node/config
	menu "Configuration"
		depends on PACKAGE_node

	choice
		prompt "Version Selection"
		default NODEJS_8
		help
		 Select node.js version.

		config NODEJS_8
			bool "8.x Maintenance LTS"

		config NODEJS_10
			bool "10.x Active LTS"

		config NODEJS_12
			bool "12.x Active LTS"
	endchoice

	choice
		prompt "ICU Selection"
		default NODEJS_ICU_NONE
		help
		 Select i18n features

		config NODEJS_ICU_NONE
			bool "Disable"

		config NODEJS_ICU_SMALL
			depends on !BIG_ENDIAN
			bool "small-icu"

		config NODEJS_ICU_SYSTEM
			depends on ARCH_64BIT&&!BIG_ENDIAN
			bool "system-icu"
	endchoice

	endmenu
		config NODEJS_UPX
		depends on PACKAGE_node
		bool "Compress executable files with UPX"
endef

NODEJS_CPU:=$(subst powerpc,ppc,$(subst aarch64,arm64,$(subst x86_64,x64,$(subst i386,ia32,$(ARCH)))))

ifeq ($(CONFIG_NODEJS_10),y)
TARGET_LDFLAGS+=-latomic
endif

ifeq ($(CONFIG_NODEJS_12),y)
TARGET_LDFLAGS+=-latomic
endif

ifneq ($(CONFIG_ARCH_64BIT),y)
FORCE_32BIT:=-m32
endif

MAKE_VARS+= \
	DESTCPU=$(NODEJS_CPU) \
	LD_LIBRARY_PATH=$(STAGING_DIR_HOSTPKG)/share/icu/65.1/lib

CONFIGURE_VARS:= \
	CC="$(TARGET_CC) $(TARGET_OPTIMIZATION)" \
	CXX="$(TARGET_CXX) $(TARGET_OPTIMIZATION)" \
	CC_host="$(HOSTCC) $(FORCE_32BIT)" \
	CXX_host="$(HOSTCXX) $(FORCE_32BIT)"

CONFIGURE_ARGS:= \
	--dest-cpu=$(NODEJS_CPU) \
	--dest-os=linux \
	--cross-compiling \
	$(if $(CONFIG_NODEJS_ICU_NONE),--without-snapshot) \
	$(if $(CONFIG_BIG_ENDIAN),--without-snapshot) \
	--shared-zlib \
	$(if $(filter reboot v19.07%, $(OW_VER)),--shared-openssl, \
		$(if $(CONFIG_NODEJS_8),--shared-openssl)) \
	$(if $(CONFIG_NODEJS_ICU_SMALL),, \
		--with-intl=$(if $(CONFIG_NODEJS_ICU_SYSTEM),system-icu,none)) \
	$(if $(findstring mips,$(NODEJS_CPU)), \
		$(if $(CONFIG_SOFT_FLOAT),--with-mips-float-abi=soft)) \
	$(if $(findstring +neon,$(CONFIG_CPU_TYPE)),--with-arm-fpu=neon) \
	$(if $(findstring +vfp",$(CONFIG_CPU_TYPE)),--with-arm-fpu=vfp) \
	$(if $(findstring +vfpv3",$(CONFIG_CPU_TYPE)),--with-arm-fpu=vfpv3-d16) \
	$(if $(findstring +vfpv4",$(CONFIG_CPU_TYPE)),--with-arm-fpu=vfpv3) \
	--prefix=/usr

HOST_CONFIGURE_VARS:=
HOST_CONFIGURE_ARGS:= \
	--dest-os=$(if $(findstring Darwin,$(HOST_OS)),mac,linux) \
	--without-snapshot \
	--with-intl=none \
	--prefix=$(STAGING_DIR_HOSTPKG)

HOST_CONFIGURE_CMD:=python2 ./configure

ifeq ($(CONFIG_NODEJS_12),y)

HOST_CONFIGURE_VARS:= \
	CC="$(HOSTCC) $(FORCE_32BIT)" \
	CXX="$(HOSTCXX) $(FORCE_32BIT)"

ifeq ($(CONFIG_NODEJS_ICU_NONE),y)

HOST_EXE=$(HOST_BUILD_DIR)/out/Release/
FILE_V8_GYP:=$(PKG_BUILD_DIR)/tools/v8_gypfiles/v8.gyp
FILE_NODE_GYP:=$(PKG_BUILD_DIR)/node.gyp

define Build/Prepare
	$(Build/Prepare/Default)
	$(SED) "s#<(PRODUCT_DIR)/<(EXECUTABLE_PREFIX)#$(HOST_EXE)#g" $(FILE_V8_GYP)
	$(SED) "s#<(PRODUCT_DIR)/<(EXECUTABLE_PREFIX)#$(HOST_EXE)#g" $(FILE_NODE_GYP)
endef
endif
endif

define Build/Install
$(call Build/Install/Default)
ifeq ($(CONFIG_NODEJS_UPX),y)
	upx --lzma --best $(PKG_INSTALL_DIR)/usr/bin/node
endif
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/* $(1)/usr/include/
endef

define Package/node/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/node $(1)/usr/bin/
endef

define Package/node-npm/install
	$(INSTALL_DIR) $(1)/usr/lib/node_modules
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/node_modules/* $(1)/usr/lib/node_modules/
	$(INSTALL_DIR) $(1)/usr/bin
	$(LN) ../lib/node_modules/npm/bin/npm-cli.js $(1)/usr/bin/npm
	$(LN) ../lib/node_modules/npm/bin/npx-cli.js $(1)/usr/bin/npx
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,node))
$(eval $(call BuildPackage,node-npm))
