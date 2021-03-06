include ../../Makefile.include
include FFMPEG-VERSION
DEPS= ../../Makefile.include FFMPEG-VERSION Makefile \
  pfcd_hevc_optimisations.patch \
  0001-ffmpeg-Call-get_format-to-fix-an-issue-with-MMAL-ren.patch \
  0001-mpeg4video-Signal-unsupported-GMC-with-more-than-one.patch \
  added_upstream_mvc_patches.patch \

# set to "yes" to enable patching
# we don't apply patches until we move to a vanilla ffmpeg tarball
APPLY_PATCHES=no

# configuration settings
ffmpg_config = --prefix=$(PREFIX) --extra-version="kodi-$(VERSION)"
ffmpg_config += --cc="$(CC)" --cxx="$(CXX)" --ar=$(AR) --ranlib=$(RANLIB) --strip=$(STRIP)
ffmpg_config += --disable-devices --disable-doc
ffmpg_config += --disable-ffplay --disable-ffmpeg
ffmpg_config += --disable-ffprobe
ffmpg_config += --disable-sdl2
ffmpg_config += --enable-gpl --enable-runtime-cpudetect
ffmpg_config += --enable-postproc --enable-pthreads
ffmpg_config += --enable-muxer=spdif --enable-muxer=adts
ffmpg_config += --enable-muxer=asf --enable-muxer=ipod
ffmpg_config += --enable-encoder=ac3 --enable-encoder=aac
ffmpg_config += --enable-encoder=wmav2 --enable-protocol=http
ffmpg_config += --enable-gnutls
ffmpg_config += --enable-encoder=png --enable-encoder=mjpeg
ffmpg_config += $(FFMPEG_CONFIGURE_OPTIONS)

ARCH = $(CPU)
ifeq ($(CPU), cortex-a7)
  ARCH = arm
endif

ifeq ($(CROSS_COMPILING), yes)
  ffmpg_config += --arch=$(ARCH) --enable-cross-compile
endif
ifeq ($(OS), linux)
  ffmpg_config += --target-os=$(OS)
  ffmpg_config += --enable-pic
endif
ifeq ($(OS), android)
  ifeq ($(findstring arm64, $(CPU)), arm64)
    ffmpg_config += --arch=aarch64 --cpu=cortex-a53
  else
    ifeq ($(findstring arm, $(CPU)), arm)
      ffmpg_config += --cpu=cortex-a9
    else
      ffmpg_config += --cpu=i686 --disable-mmx
      ffmpg_config += --extra-cflags=-no-integrated-as --extra-cflags=-mno-stackrealign
    endif
  endif
  ffmpg_config += --target-os=linux --extra-libs=-liconv --disable-linux-perf
endif
ifeq ($(OS), darwin_embedded)
  ifneq ($(CPU), arm64)
    ffmpg_config += --cpu=cortex-a8
    ffmpg_config += --as="$(NATIVEPREFIX)/bin/gas-preprocessor.pl -- $(CC)"
  else
    ffmpg_config += --as="$(NATIVEPREFIX)/bin/gas-preprocessor.pl -arch aarch64 -- $(CC)"
  endif
  ffmpg_config += --yasmexe=$(NATIVEPREFIX)/bin/yasm
  ffmpg_config += --disable-decoder=mpeg_xvmc --disable-crystalhd --enable-videotoolbox
  ffmpg_config += --target-os=darwin
endif
ifeq ($(OS), osx)
  ffmpg_config += --disable-decoder=mpeg_xvmc --disable-crystalhd --enable-videotoolbox
  ffmpg_config += --target-os=darwin
  ffmpg_config += --disable-securetransport
endif
ifeq ($(findstring arm, $(CPU)), arm)
  ffmpg_config += --enable-pic --disable-armv5te --disable-armv6t2
endif
ifeq ($(findstring mips, $(CPU)), mips)
  ffmpg_config += --disable-mips32r2 --disable-mipsdsp --disable-mipsdspr2
endif
ifeq ($(Configuration), Release)
  ffmpg_config += --disable-debug
endif

ffmpg_config += --enable-rpi --extra-cflags=-mfpu=neon

all: .installed-$(PLATFORM)

$(TARBALLS_LOCATION)/$(ARCHIVE):
	cd $(TARBALLS_LOCATION); $(RETRIEVE_TOOL) -Ls --create-dirs -f -o $(TARBALLS_LOCATION)/$(ARCHIVE) $(BASE_URL)/archive/$(VERSION).tar.gz

$(PLATFORM): $(TARBALLS_LOCATION)/$(ARCHIVE) $(DEPS)
	rm -rf $(PLATFORM); mkdir -p $(PLATFORM)
	cd $(PLATFORM); $(ARCHIVE_TOOL) $(ARCHIVE_TOOL_FLAGS) $(TARBALLS_LOCATION)/$(ARCHIVE)
	cd $(PLATFORM); sed -i".bak" -e "s%pkg_config_default=pkg-config%export PKG_CONFIG_LIBDIR=$(PREFIX)/lib/pkgconfig \&\& pkg_config_default=$(NATIVEPREFIX)/bin/pkg-config%" configure
	cd $(PLATFORM); patch -p1 < ../pfcd_hevc_optimisations.patch
	cd $(PLATFORM); patch -p1 < ../0001-ffmpeg-Call-get_format-to-fix-an-issue-with-MMAL-ren.patch
	cd $(PLATFORM); patch -p1 < ../0001-mpeg4video-Signal-unsupported-GMC-with-more-than-one.patch
	cd $(PLATFORM); patch -p1 < ../added_upstream_mvc_patches.patch
	cd $(PLATFORM);\
	CFLAGS="$(CFLAGS)" CXXFLAGS="$(CXXFLAGS)" CPPFLAGS="$(CPPFLAGS)" LDFLAGS="$(LDFLAGS)" \
	./configure $(ffmpg_config)
ifeq ($(OS), darwin_embedded)
	cd $(PLATFORM); sed -i -- 's/HAVE_CLOCK_GETTIME 1/HAVE_CLOCK_GETTIME 0/g' config.h
endif

.build-$(PLATFORM): $(PLATFORM)
	$(MAKE) -C $(PLATFORM)
	touch $@

.installed-$(PLATFORM): .build-$(PLATFORM)
	$(MAKE) -C $(PLATFORM) install
	touch $@

clean:
	$(MAKE) -C $(PLATFORM) clean
	rm -f .installed-$(PLATFORM)

distclean::
	rm -rf $(PLATFORM) .installed-$(PLATFORM)
