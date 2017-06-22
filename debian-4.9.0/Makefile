BUILDDEPS=debhelper cpio module-init-tools python xmlto quilt kernel-wedge kmod python python-six lzma patchutils bc gcc-4.8 xz-utils

REPOSITORY=stretch

# EXTRADCHARGS:= --distributor Untangle

# Upstream version numbers
KVER=4.9.30
KDEBVER=2

ORIG_TARBALL := linux_$(KVER).orig.tar.xz
ORIG_TARBALL_URL := http://download.untangle.com/kernels/$(ORIG_TARBALL)
DEBIAN_TARBALL := linux_$(KVER)-$(KDEBVER).debian.tar.xz
DEBIAN_TARBALL_URL := http://download.untangle.com/kernels/$(DEBIAN_TARBALL)

KUPVER=${KDEBVER}
PACKPFX=linux_${KVER}
KERNEL_DIR=linux-${KVER}
KDSC=${PACKPFX}-${KUPVER}.dsc
KOURDIFF=untangle-debian.diff
KOURPATCH=${KOURDIFF}.gz

UNTANGLE_PATCH_SERIES := patches/untangle/series
DEBIAN_PATCH_SERIES := patches/debian/series

MACHINE:=$(shell dpkg-architecture -qDEB_BUILD_ARCH)
NPROCEXP:=$(shell echo "$$((1+`grep -c processor /proc/cpuinfo`))")
TEMPDIR:=$(shell mktemp)
TEMPDIR2:=$(shell mktemp)

ifeq ($(MACHINE),armhf)
  NPROCEXP := 1
endif

DOPTIONS = DEBIAN_KERNEL_JOBS=${NPROCEXP} DEB_BUILD_OPTIONS="parallel=${NPROCEXP}" CONCURRENCY_LEVEL=${NPROCEXP} AUTOBUILD=1

all:	clean pkgs

$(ORIG_TARBALL):
	curl -O $(ORIG_TARBALL_URL)
$(DEBIAN_TARBALL):
	curl -O $(DEBIAN_TARBALL_URL)

extract: extract-stamp
extract-stamp: $(ORIG_TARBALL) $(DEBIAN_TARBALL)
	dpkg-source -x ${KDSC}
	touch $@

patch-debian: patch-debian-stamp
patch-debian-stamp: extract-stamp patches/debian/*
	while read patch ; do \
	  echo "Applying $$patch" ; \
	  patch -d $(KERNEL_DIR) -p1 < patches/debian/$$patch || exit 1 ; \
	done < $(DEBIAN_PATCH_SERIES)
	# FIXME: prolly already in basediff patch ?
	cat $(CONFIG_ORIG) $(CONFIG_UNTANGLE_ORIG) >| $(CONFIG_DEST_KERNEL)
	touch $@

patch-untangle: patch-untangle-stamp
patch-untangle-stamp: extract-stamp patches/untangle/*
	while read patch ; do \
	  echo "Applying $$patch" ; \
	  patch -d $(KERNEL_DIR) -p1 < patches/untangle/$$patch ; \
	done < $(UNTANGLE_PATCH_SERIES)
	touch $@

patch: patch-debian patch-untangle
	# FIXME: make this a debian patch
	perl -i -pe 's/fuzz=0/fuzz=3/' ${KERNEL_DIR}/debian/rules*

pkgs:	kpkg-arch kpkg-indep

deps:	force
	sudo apt-get install ${BUILDDEPS} || echo "unable to run sudo"

kpkg-arch: control-real force
	cd ${KERNEL_DIR}; ${DOPTIONS} fakeroot make -j ${NPROCEXP} -f debian/rules.gen binary-arch_${MACHINE}_untangle
kpkg-indep: control-real force
	cd ${KERNEL_DIR}; ${DOPTIONS} fakeroot make -j ${NPROCEXP} -f debian/rules binary-indep

kern:   ${KERNEL_DIR} kpkg-arch

src: 	${KERNEL_DIR} control-real force
	cd ${KERNEL_DIR}; ${DOPTIONS} fakeroot make -f debian/rules.gen source_untangle setup_${MACHINE}_untangle

control-real: ${KERNEL_DIR}/debian/control-real
${KERNEL_DIR}/debian/control-real: force patch
	cd ${KERNEL_DIR};make -f debian/rules debian/control-real || true

clean::
	rm -f ${KOURPATCH} ${KOURDIFF}
	rm -rf ${KERNEL_DIR}
	rm -f *.deb modules/*.deb
	rm -f *.udeb modules/*.udeb

force: