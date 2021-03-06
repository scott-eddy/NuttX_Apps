############################################################################
# apps/graphics/traveler/Makefile
#
#   Copyright (C) 2014, 2016 Gregory Nutt. All rights reserved.
#   Author: Gregory Nutt <gnutt@nuttx.org>
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in
#    the documentation and/or other materials provided with the
#    distribution.
# 3. Neither the name NuttX nor the names of its contributors may be
#    used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
# INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
# BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS
# OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED
# AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
# ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#
############################################################################

-include $(TOPDIR)/.config
-include $(TOPDIR)/Make.defs
include $(APPDIR)/Make.defs

# Directories

TRAVELER_DIR   = $(APPDIR)/graphics/traveler
TRAVELER_SRC   = $(TRAVELER_DIR)/src
TRAVELER_INC   = $(TRAVELER_DIR)/include
TRAVELER_DEMO  = $(TRAVELER_DIR)/world
TRAVELER_TOOLS = $(TRAVELER_DIR)/tools

# Traveler built-in application info

APPNAME   = traveler
PRIORITY  = SCHED_PRIORITY_DEFAULT
STACKSIZE = 2048

# Traveler files

ASRCS   =
CSRCS   = trv_bitmapfile.c trv_bitmaps.c trv_color.c trv_doors.c
CSRCS  += trv_fsutils.c trv_graphicfile.c trv_graphics.c trv_input.c
CSRCS  += trv_mem.c trv_paltbl.c trv_pcx.c trv_planefiles.c trv_planelists.c
CSRCS  += trv_pov.c trv_rayavoid.c trv_raycast.c trv_raycntl.c
CSRCS  += trv_rayprune.c trv_rayrend.c trv_texturefile.c trv_trigtbl.c
CSRCS  += trv_world.c

ifeq ($(CONFIG_GRAPHICS_TRAVELER_ROMFSDEMO),y)
CSRCS += trv_romfs.c
endif

MAINSRC = trv_main.c

ifeq ($(CONFIG_NX),y)
CSRCS  += trv_nxbkgd.c trv_nxlistener.c
endif

AOBJS = $(ASRCS:.S=$(OBJEXT))
COBJS = $(CSRCS:.c=$(OBJEXT))
MAINOBJ = $(MAINSRC:.c=$(OBJEXT))

SRCS = $(ASRCS) $(CSRCS) $(MAINSRC)
OBJS = $(AOBJS) $(COBJS)

ifneq ($(CONFIG_BUILD_KERNEL),y)
  OBJS += $(MAINOBJ)
endif

ifeq ($(CONFIG_GRAPHICS_TRAVELER_ROMFSDEMO),y)
ROMFS_IMG = trv_romfs.img
ROMFS_HDR = trv_romfs.h
endif

ifeq ($(CONFIG_WINDOWS_NATIVE),y)
  BIN = ..\..\libapps$(LIBEXT)
else
ifeq ($(WINTOOL),y)
  BIN = ..\\..\\libapps$(LIBEXT)
else
  BIN = ../../libapps$(LIBEXT)
endif
endif

ifeq ($(WINTOOL),y)
  INSTALL_DIR = "${shell cygpath -w $(BIN_DIR)}"
else
  INSTALL_DIR = $(BIN_DIR)
endif

CONFIG_GRAPHICS_TRAVELER_PROGNAME ?= hello$(EXEEXT)
PROGNAME = $(CONFIG_GRAPHICS_TRAVELER_PROGNAME)

CFLAGS += ${shell $(INCDIR) $(INCDIROPT) "$(CC)" $(TRAVELER_INC)}

DEPPATH = --dep-path . --dep-path src
VPATH = .:src

# Common build

all: .built tools
.PHONY: tools clean depend distclean preconfig
.PRECIOUS: ../../libapps$(LIBEXT)

$(AOBJS): %$(OBJEXT): %.S
	$(call ASSEMBLE, $<, $@)

$(COBJS) $(MAINOBJ): %$(OBJEXT): %.c
	$(call COMPILE, $<, $@)

ifeq ($(CONFIG_GRAPHICS_TRAVELER_ROMFSDEMO),y)
$(ROMFS_IMG):
	$(Q) genromfs -f $@ -d $(TRAVELER_DEMO) -V "DemoWorld"

$(ROMFS_HDR): $(ROMFS_IMG)
	$(Q) (xxd -i $< | sed -e "s/^unsigned/static const unsigned/g" >$@)
endif

.built: $(ROMFS_HDR) $(OBJS)
	$(call ARCHIVE, $(BIN), $(OBJS))
	@touch .built

tools:
	$(Q) $(MAKE) -C  $(TRAVELER_TOOLS) -f Makefile.host TOPDIR="$(TOPDIR)" APPDIR="$(APPDIR)"

ifeq ($(CONFIG_BUILD_KERNEL),y)
$(BIN_DIR)$(DELIM)$(PROGNAME): $(OBJS) $(MAINOBJ)
	@echo "LD: $(PROGNAME)"
	$(Q) $(LD) $(LDELFFLAGS) $(LDLIBPATH) -o $(INSTALL_DIR)$(DELIM)$(PROGNAME) $(ARCHCRT0OBJ) $(MAINOBJ) $(LDLIBS)
	$(Q) $(NM) -u  $(INSTALL_DIR)$(DELIM)$(PROGNAME)

install: $(BIN_DIR)$(DELIM)$(PROGNAME)

else
install:

endif

ifeq ($(CONFIG_NSH_BUILTIN_APPS),y)
$(BUILTIN_REGISTRY)$(DELIM)$(APPNAME)_main.bdat: $(DEPCONFIG) Makefile
	$(call REGISTER,$(APPNAME),$(PRIORITY),$(STACKSIZE),$(APPNAME)_main)

context: $(BUILTIN_REGISTRY)$(DELIM)$(APPNAME)_main.bdat
else
context:
endif

.depend: Makefile $(SRCS)
	@$(MKDEP) $(DEPPATH) "$(CC)" -- $(CFLAGS) -- $(SRCS) >Make.dep
	@touch $@

depend: .depend

clean:
	$(call DELFILE, .built)
	$(call DELFILE, $(ROMFS_IMG))
	$(call DELFILE, $(ROMFS_HDR))
	$(call CLEAN)

distclean: clean
	$(call DELFILE, Make.dep)
	$(call DELFILE, .depend)

preconfig:

-include Make.dep
