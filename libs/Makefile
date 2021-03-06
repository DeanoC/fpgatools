#
# Makefile - fpgatools
# Author: Wolfgang Spraul
#
# This is free and unencumbered software released into the public domain.
# For details see the UNLICENSE file at the root of the source tree.
#

# OS X linker doesn't support -soname, and use different extension
# see : https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/DynamicLibraryDesignGuidelines.html
ifeq ($(shell uname), Darwin)
	CC = clang
else
	CC = clang-3.6
endif

LIBS_VERSION_MAJOR = 0
LIBS_VERSION = $(LIBS_VERSION_MAJOR).0.0

LIBFPGA_BIT_OBJS       = bit_frames.o bit_regs.o
LIBFPGA_MODEL_OBJS     = model_main.o model_tiles.o model_devices.o \
	model_ports.o model_conns.o model_switches.o model_helper.o
LIBFPGA_FLOORPLAN_OBJS = floorplan.o
LIBFPGA_CONTROL_OBJS   = control.o parts.o helper.o

OBJS := $(LIBFPGA_BIT_OBJS) $(LIBFPGA_MODEL_OBJS) \
	$(LIBFPGA_FLOORPLAN_OBJS) $(LIBFPGA_CONTROL_OBJS)

DYNAMIC_LIBS = libfpga-model.so libfpga-bit.so libfpga-floorplan.so \
	libfpga-control.so #libfpga-cores.so

DYNAMIC_HEADS = bit.h control.h floorplan.h helper.h model.h parts.h

ifeq ($(shell uname), Darwin)
	ALL_LIBS = $(DYNAMIC_LIBS:.so=.a)
else
	SHARED_FLAGS = -shared -Wl,-soname,$@.$(LIBS_VERSION_MAJOR)
	ALL_LIBS = $(DYNAMIC_LIBS) $(DYNAMIC_LIBS:.so=.a)
endif

.PHONY:	all clean install uninstall FAKE

all: $(ALL_LIBS)
	@echo $(DYNAMIC_HEADS)
	@echo $(ALL_LIBS)

include ../Makefile.common

libfpga-bit.a: $(LIBFPGA_BIT_OBJS)

libfpga-model.a: $(LIBFPGA_MODEL_OBJS)

libfpga-floorplan.a: $(LIBFPGA_FLOORPLAN_OBJS)

libfpga-control.a: $(LIBFPGA_CONTROL_OBJS)

%.a:
	$(AR) $@ $^
	$(RANLIB) $@

libfpga-bit.so: $(LIBFPGA_BIT_OBJS)

libfpga-model.so: $(LIBFPGA_MODEL_OBJS)

libfpga-floorplan.so: $(LIBFPGA_FLOORPLAN_OBJS)

libfpga-control.so: $(LIBFPGA_CONTROL_OBJS)

%.so:
	$(CC) $(LDFLAGS) $(SHARED_FLAGS) -o $@.$(LIBS_VERSION) $^
	@ln -sf $@.$(LIBS_VERSION_MAJOR) $@
	@ln -sf $@.$(LIBS_VERSION) $@.$(LIBS_VERSION_MAJOR)

%.o:	%.c
	$(CC) $(CPPFLAGS) $(CFLAGS) -fPIC -o $@ -c $<
	$(MKDEP)

clean:
	rm -f $(OBJS) $(OBJS:.o=.d) $(DYNAMIC_LIBS)
	rm -f $(DYNAMIC_LIBS:.so=.a)
	rm -f $(DYNAMIC_LIBS:.so=.so.$(LIBS_VERSION_MAJOR))
	rm -f $(DYNAMIC_LIBS:.so=.so.$(LIBS_VERSION))

install: all
	mkdir -p $(DESTDIR)/$(PREFIX)/include/
	mkdir -p $(DESTDIR)/$(PREFIX)/lib/
	install -m 644 $(DYNAMIC_HEADS) $(DESTDIR)/$(PREFIX)/include/
	for f in $(DYNAMIC_LIBS); do \
	  chrpath -d $$f.$(LIBS_VERSION_MAJOR) && \
	  install -m 644 $$f.$(LIBS_VERSION) $(DESTDIR)/$(PREFIX)/lib/$$f.$(LIBS_VERSION) && \
	  (cd $(DESTDIR)/$(PREFIX)/lib/ && \
	   ln -sf $$f.$(LIBS_VERSION) $$f.$(LIBS_VERSION_MAJOR) && \
	   ln -sf $$f.$(LIBS_VERSION_MAJOR) $$f) \
	  || exit 1; done
	for f in $(DYNAMIC_LIBS:.so=.a); do \
	  install -m 644 $$f $(DESTDIR)/$(PREFIX)/lib/$$f \
	  || exit 1; done

uninstall:
	for f in $(DYNAMIC_HEADS); do rm -f $(DESTDIR)/$(PREFIX)/include/$$f || exit 1; done
	for f in $(DYNAMIC_LIBS) $(DYNAMIC_LIBS:.so=.a) \
	  $(DYNAMIC_LIBS:.so=.so.$(LIBS_VERSION_MAJOR)) \
	  $(DYNAMIC_LIBS:.so=.so.$(LIBS_VERSION)); do \
	  rm -f $(DESTDIR)/$(PREFIX)/lib/$$f || exit 1; done
