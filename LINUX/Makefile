# To build external modules, you must have a prebuilt kernel available
# that contains the configuration and header files used in the build.
# go in the kernel directory and do a
#       make oldconfig; make scripts; make prepare
#
# The list of targets is derived from obj-m
# and then the corresponding foo-objs

CONFIG_NETMAP:=m
CONFIG_R8169:=m
CONFIG_FORCEDETH:=m
CONFIG_E1000:=m
CONFIG_E1000E:=m
CONFIG_IXGBE:=m
CONFIG_IGB:=m

netmap_lin-objs := netmap.o

obj-$(CONFIG_NETMAP) = netmap_lin.o
obj-m += $(DRIVERS)
# obj-$(CONFIG_R8169) += r8169.o
# obj-$(CONFIG_FORCEDETH) += forcedeth.o
# obj-$(CONFIG_E1000) += e1000/
# obj-$(CONFIG_E1000E) += e1000e/
# obj-$(CONFIG_IXGBE) += ixgbe/
# obj-$(CONFIG_IGB) += igb/

# names of the driver sources. In old linuxes are under
# KSRC/drivers/net, but depending on where you build they
# can be in $(KSRC)/source/drivers/net/ethernet/$(manufacturer)
DRIVER_SRCS = r8169.c forcedeth.c e1000/ e1000e/ ixgbe/ igb/
DRIVER_SUBDIRS= nvidia realtek intel

# The following commands are needed to build the modules as out-of-tree, in
# fact the kernel sources path must be specified.

# Additional compile flags (e.g. header location)
M:=$(PWD)
EXTRA_CFLAGS := -I$(M) -I$(M)/../sys -I$(M)/../sys/dev -DCONFIG_NETMAP

# We use KSRC for the kernel configuration and sources.
# If the sources are elsewhere, then use SRC to point to them.
KSRC ?= /lib/modules/$(shell uname -r)/build
SRC ?= $(KSRC)

# extract version number and filter with the available patches.
LIN_VER = $(shell grep LINUX_VERSION_CODE $(KSRC)/include/linux/version.h | \
	awk '{printf "%03x%02d", $$3/256, $$3%256} ')

PATCHES := $(shell \
	    cd $(PWD)/patches; ls diff--* | awk -v v=$(LIN_VER) -F -- '{ \
	    if ((!$$3 || $$3 <= v) && (!$$4 || v < $$4)) print $0; }')

DRIVERS := $(shell \
	    cd $(PWD)/patches; ls diff--* | awk -v v=$(LIN_VER) -F -- '{ \
	    if ((!$$3 || $$3 <= v) && (!$$4 || v < $$4)) { ; \
	    if (match($$2, ".c")) print $$2 ; else print $$2 "/" } }' )

all:	get-drivers build

build:
	make -C $(KSRC) M=$(PWD) CONFIG_NETMAP=m	\
		CONFIG_E1000=m CONFIG_E1000E=m \
		CONFIG_IXGBE=m CONFIG_IGB=m EXTRA_CFLAGS='$(EXTRA_CFLAGS)' \
		DRIVERS="$(DRIVERS:%.c=%.o)" modules
	@ls -l `find . -name \*.ko`

test:
	@echo "version $(LIN_VER)"
	@echo "patches $(PATCHES)"
	@echo "drivers $(DRIVERS)"

clean:
	-@ make -C $(KSRC) M=$(PWD) clean
	-@ (rm -rf $(DRIVER_SRCS) *.orig *.rej *.ko *.o get-drivers )

# the source is not here so we need to specify a dependency
$(obj)/netmap.o: $(M)/../sys/dev/netmap/netmap.c
	$(call cmd,cc_o_c)
	$(call cmd,modversions)
#	$(call quiet_cmd,cc_o_c)

#-- copy and patch initial files
# The location changes depending on the OS version, so ...
get-drivers:
	@echo "LIN_VER $(LIN_VER) subdirs $(DRIVER_SUBDIRS) files $(DRIVERS)"
	-@(								\
	cd $(SRC); [ -d source ] && cd source;			\
	    cd drivers/net; s=. ;					\
	    [ -d ethernet ] && cd ethernet && s="$(DRIVER_SUBDIRS)" ;	\
	    for i in $$s; do (cd $$i ;					\
	    	cp -Rp $(DRIVERS) $(PWD) 2>/dev/null ); done )
	-@(for i in $(PATCHES) ; do echo "** use patches/$$i";		\
		patch --posix --quiet --force -p1 < patches/$$i; done )
	@touch get-drivers
	@echo "Building the following drivers: $(obj-m)"

# copy and patch files from the source tree
#$(obj)/r8169.c: $(srctree)/drivers/net/r8169.c 
#	# net-r8169.diff
#	$(call cmd,shipped)
#	(cd $(obj); patch < net-r8169.diff )
#

apps:
	(cd ../examples; make)
