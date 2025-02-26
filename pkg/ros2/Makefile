# Makefile - for ros2
#
# Copyright (c) 2019 Wind River Systems, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# modification history
# --------------------
# 15nov19,rcw  created
# 12mar18,akh  created
#

# SYNOPSIS        automate the build of ROS 2
include $(WIND_USR_MK)/defs.common.mk
include $(WIND_USR_MK)/defs.packages.mk
include $(WIND_USR_MK)/defs.crossbuild.mk

PACKAGES += ros2

PKG_NAME = ros2
PKG_VER = $(ROS_DISTRO)

# use the same branch for the patches
PKG_URL = https://github.com/Wind-River/vxworks7-layer-for-ros2.git
PKG_TYPE = git

# a layer branch is always aligned with the current build branch
PKG_COMMIT_ID = $(shell git rev-parse --abbrev-ref HEAD)

ifeq ($(PKG_COMMIT_ID),master)
PKG_REPO_COMMIT_ID = $(ROS_DISTRO)-release
else
# extract ROS 2 release part, <ros2 release>-<vxworks release>-<add-on>,
# where <ros-release> - humble-release, or rolling
# <vxworks release> - 23.03
# <add-on> - next
PKG_REPO_COMMIT_ID = $(shell git rev-parse --abbrev-ref HEAD | grep -o '^[^[:digit:]]*' | head -c-2)
endif
PKG_REPO_URL = https://raw.githubusercontent.com/ros2/ros2/${PKG_REPO_COMMIT_ID}/${PKG_NAME}.repos

PKG_LICENSE = BSD

PKG_BUILD_DIR = ros2_ws
PKG_SRC_DIR = patches

PKG_BUILD_TYPE := Debug

include $(PACKAGE_DIR)/ros2/files/$(ROS_DISTRO)/packages.mk

define ros2_patch
        for pn in $(1); do \
	        N=$$(basename $$pn); \
                if ls $(BUILD_DIR)/$(PKG_NAME)/$(PKG_SRC_DIR)/$$N/usr_src/0001-* 1> /dev/null 2>&1; then \
                        PATCHES="$(BUILD_DIR)/$(PKG_NAME)/$(PKG_SRC_DIR)/$$N/usr_src/*.patch"; \
                        for PATCH in $$PATCHES; do \
                                cd $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/src/$$pn ; \
                                $(PATCH) -p1 < $$PATCH ; \
                        done; \
                fi \
	done;
endef

ros2.unpack: ros2.download
	@$(call echo_action,Unpacking,$*)
	$(call pkg_unpack,$(PKG_NAME))
	mkdir -p $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR); \
	cd $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR); \
	curl -o $(PKG_NAME).repos $(PKG_REPO_URL); \
	mkdir src; \
	vcs import src < $(PKG_NAME).repos
	@$(MAKE_STAMP)

ros2.patch: ros2.unpack
	@$(call echo_action,Patching,$*)
	$(call ros2_patch,$(ROS2_PATCH_DIRS))
	@$(MAKE_STAMP)

ros2.configure: ros2.patch
	@$(call echo_action,Configuring,$*)
	for i in $(ROS_IGNORE_DIRS); do \
		touch $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/src/$$i/COLCON_IGNORE; \
	done;
	pip3 install numpy;
	pip3 install lark-parser;
	mkdir -p $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/.colcon/mixin/default;
	cp files/$(ROS_DISTRO)/*.mixin $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/.colcon/mixin/default/.;
	sed -i "s/-3./-3$(TGT_PYTHON_MINOR)/g" $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/.colcon/mixin/default/vxworks.mixin;
	sed -i "s+/work+$(TOP_BUILDDIR)+g" $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/.colcon/mixin/default/vxworks.mixin;
	sed -i "s+/wrsdk/vxsdk/host+$(WIND_SDK_HOST_TOOLS)+g" $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/.colcon/mixin/default/vxworks.mixin;
	@$(MAKE_STAMP)

ros2.build: ros2.configure
	@$(call echo_action,Building,$*)
	cd $(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR); \
	export COLCON_HOME=$(BUILD_DIR)/$(PKG_NAME)/$(PKG_BUILD_DIR)/.colcon; \
	colcon build --mixin vxworks --packages-up-to-regex $(PKG_PKGS_UP_TO)
	@$(MAKE_STAMP)

ros2.install : ros2.build
	@$(call echo_action,Installing runtime dependencies,$*)
	pip3 install --ignore-installed --prefix=$(DEPLOY_DIR) setuptools
	pip3 install --ignore-installed --prefix=$(DEPLOY_DIR) catkin_pkg
	pip3 install --ignore-installed --prefix=$(DEPLOY_DIR) pyyaml
	pip3 install --ignore-installed --prefix=$(DEPLOY_DIR) empy
	pip3 install --ignore-installed --prefix=$(DEPLOY_DIR) pyelftools
	pip3 install --ignore-installed --prefix=$(DEPLOY_DIR) packaging
	@$(call echo_action,Installing build artifacts,$*)
	cp -r $(ROOT_DIR)/bin/* $(DEPLOY_DIR)/bin/.
	cp -r $(ROOT_DIR)/lib/* $(DEPLOY_DIR)/lib/.
	cp -r $(ROOT_DIR)/share $(DEPLOY_DIR)/.
	cp files/$(ROS_DISTRO)/vxscript $(DEPLOY_DIR)/.
	@$(MAKE_STAMP)
	
include $(WIND_USR_MK)/rules.packages.mk

