# Makefile for metasepi system.
ARCH       = amd64
KERNCONF   = GENERIC
CURDIR     = $(shell pwd)
TOOLDIR    = obj/tooldir
RELEASEDIR = obj/releasedir
SETSDIR    = ${RELEASEDIR}/${ARCH}/binary/sets
DESTDIR    = obj/destdir.${ARCH}
BOOTCDDIR  = obj/bootcd
NUMCPU     = $(shell cat /proc/cpuinfo | grep -c "^processor")
# See share/man/man5/mk.conf.5 to know following vars
MKVARS     = -V MKPCC=no -V MKEXTSRC=no -V MKMAN=no -V MKINFO=no -V MKATF=no -V MKCATPAGES=no -V MKCVS=no -V MKDOC=no -V MKHTML=no -V MKIPFILTER=no -V MKLVM=no -V MKNLS=no -V MKPF=no -V MKPOSTFIX=no -V MKRUMP=no -V MKX11FONTS==no -V MKX11=no -V MKYP=no -V MKZFS=no -V MKSKEY=no -V MKHESIOD=no -V MKLDAP=no -V MKMDNS=no -V MKGCCCMDS=no
BUILDSH    = sh build.sh -U -u -N 1 -j ${NUMCPU} ${MKVARS}
NBMAKE     = ${CURDIR}/${TOOLDIR}/bin/nbmake-${ARCH} -j ${NUMCPU}
NBMAKEFS   = ${CURDIR}/${TOOLDIR}/bin/nbmakefs
MINIIMGDIR = ${CURDIR}/distrib/${ARCH}/liveimage/miniimage
QEMU       = qemu-system-x86_64
QEMUOPTS   = -m 1024 -soundhw ac97 -no-reboot -cdrom ${BOOTCDDIR}/cd.iso
MAKEFSOPTS = -t cd9660 -o 'bootimage=i386;bootxx_cd9660,no-emul-boot'
#LOGFILTER  = -e "===>" -e "^nbgmake" -B 5 -e "Error code"
LOGFILTER  = ""

HSBUILD = metasepi/sys/hsbuild
HSSRC   = metasepi/sys/hssrc
HSCODE  = $(wildcard $(HSSRC)/*.hs $(HSSRC)/*/*.hs $(HSSRC)/*/*/*.hs $(HSSRC)/*/*/*/*.hs)

### Build kernel
all: obj/build_tools.stamp
	${BUILDSH} -T ${TOOLDIR} -m ${ARCH} kernel=${KERNCONF}

### Setup NetBSD environment
obj/build_tools.stamp:
	env MKCROSSGDB=yes ${BUILDSH} -T ${TOOLDIR} -m ${ARCH} tools | grep ${LOGFILTER}
	touch obj/build_tools.stamp

obj/build_dist.stamp: obj/build_tools.stamp
	${BUILDSH} -T ${TOOLDIR} -m ${ARCH} distribution | grep ${LOGFILTER}
	touch obj/build_dist.stamp

#obj/build_sets.stamp: obj/build_dist.stamp
#	${BUILDSH} -T ${TOOLDIR} -m ${ARCH} sets | grep ${LOGFILTER}
#	touch obj/build_sets.stamp

### Build QEMU image
bootcd: ${BOOTCDDIR}/bootxx_cd9660 ${BOOTCDDIR}/cd/boot \
	  ${BOOTCDDIR}/cd/boot.cfg ${BOOTCDDIR}/cd/miniroot.kmod all
	gzip -c sys/arch/${ARCH}/compile/obj/${KERNCONF}/netbsd > ${BOOTCDDIR}/cd/netbsd
	cd ${BOOTCDDIR} && ${NBMAKEFS} ${MAKEFSOPTS} cd.iso cd

${BOOTCDDIR}/bootxx_cd9660: obj/build_dist.stamp
	mkdir -p ${BOOTCDDIR}
	cp ${DESTDIR}/usr/mdec/bootxx_cd9660 $@

${BOOTCDDIR}/cd/boot: obj/build_dist.stamp
	mkdir -p ${BOOTCDDIR}
	cp ${DESTDIR}/usr/mdec/boot $@

${BOOTCDDIR}/cd/boot.cfg:
	mkdir -p ${BOOTCDDIR}/cd
	echo "timeout=0\nload=/miniroot.kmod" > $@

${BOOTCDDIR}/cd/miniroot.kmod: obj/build_dist.stamp obj/audioplay distrib/i386/ramdisks/ramdisk-audioplay/Makefile distrib/i386/ramdisks/ramdisk-audioplay/list
	mkdir -p ${BOOTCDDIR}/cd
	${NBMAKE} -C distrib/i386/ramdisks/ramdisk-audioplay
	${NBMAKE} -C distrib/i386/kmod-audioplay
	cp distrib/i386/kmod-audioplay/miniroot.kmod $@

obj/audioplay:
	${NBMAKE} -C usr.bin/audio clean
	${NBMAKE} -C usr.bin/audio LDSTATIC=-static
	cp usr.bin/audio/play/obj/audioplay $@

### Run QEMU image
qemu: bootcd
	env QEMU_AUDIO_DRV=alsa ${QEMU} ${QEMUOPTS}

qemucurses: bootcd
	env QEMU_AUDIO_DRV=alsa ${QEMU} ${QEMUOPTS} -curses

qemugnometerm: bootcd
	gnome-terminal -e "env QEMU_AUDIO_DRV=alsa ${QEMU} ${QEMUOPTS} -curses"

qemuvnc: bootcd
	@echo '####################################'
	@echo '### Run "gvncviewer localhost:0" ###'
	@echo '####################################'
	env QEMU_AUDIO_DRV=alsa ${QEMU} ${QEMUOPTS} -vnc :0

clean:
	rm -rf sys/arch/${ARCH}/compile/obj/${KERNCONF} ${HSBUILD} ${BOOTCDDIR} *~

distclean: clean
	rm -f obj/build_dist.stamp ${DESTDIR} *~
	env MKCROSSGDB=yes ${BUILDSH} -T ${TOOLDIR} -m ${ARCH} cleandir
	${NBMAKE} -C distrib/i386/kmod-audioplay clean
	${NBMAKE} -C distrib/i386/ramdisks/ramdisk-audioplay clean

.PHONY: setup bootcd clean distclean qemu qemucurses
