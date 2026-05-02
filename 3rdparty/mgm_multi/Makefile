CFLAGS   ?= -O3 -DNDEBUG -march=native
CXXFLAGS := $(CFLAGS)

CPPFLAGS := -Iiio
LDLIBS   := -lfftw3 -lpng -ltiff -ljpeg -lm

# macOS / Homebrew: Apple clang ships without OpenMP, so prefer Homebrew gcc
# when available (gives a real -fopenmp build) and pull in Homebrew's include
# and lib roots so libpng/libtiff/libjpeg/fftw3 headers are found. Activated
# only when the user hasn't pinned CC/CXX explicitly.
UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
  ifeq ($(origin CC),default)
    HB_GCC := $(shell ls /opt/homebrew/bin/gcc-* 2>/dev/null | grep -E 'gcc-[0-9]+$$' | sort -V | tail -1)
    HB_GXX := $(shell ls /opt/homebrew/bin/g++-* 2>/dev/null | grep -E 'g\+\+-[0-9]+$$' | sort -V | tail -1)
    ifneq ($(HB_GCC),)
      CC  := $(HB_GCC)
      CXX := $(HB_GXX)
    endif
  endif
  ifneq ($(wildcard /opt/homebrew/include),)
    CPPFLAGS := $(CPPFLAGS) -I/opt/homebrew/include
    LDFLAGS  := $(LDFLAGS) -L/opt/homebrew/lib
  endif
endif


BIN       = mgm_multi  mgm
OBJ       = mgm_core.o  mgm_costvolume.o  mgm_multiscale.o  census_tools.o \
            stereo_utils.o  point.o  shear.o  img.o  iio.o

all       : $(BIN)
iio.o     : iio/iio.c       ; $(CC) $(CFLAGS) $(CPPFLAGS) -c $^ -o $@
%         : main_%.o $(OBJ) ; $(CXX) $(LDFLAGS)              $^ -o $@ $(LDLIBS)
clean     :                 ; $(RM) $(BIN) $(OBJ)


test: all
	MEDIAN=1 CENSUS_NCC_WIN=3 USE_TRUNCATED_LINEAR_POTENTIALS=1  TSGM=3 ./mgm -P2 20000 -P1 2 -r -120 -R 30 -t census -s vfit -O 8 data/fountain23-im?.png /tmp/{disp,cost}.tif
	MEDIAN=1 USE_TRUNCATED_LINEAR_POTENTIALS=1  TSGM=3 ./mgm -P2 20000 -P1 4 -r -120 -R 30 -p sobel_x -truncDist 63 -s vfit -O 8 data/fountain23-im?.png /tmp/{disp,cost}.tif


# hack to add -fopenmp only when the CXX compiler supports it
OMPVER := $(shell $(CXX) -dM -E -fopenmp - 2>/dev/null </dev/null |grep _OPENMP)
ifneq ($(OMPVER),)
LDLIBS := $(LDLIBS) -fopenmp
CXXFLAGS := $(CXXFLAGS) -fopenmp
endif

# hack to add -std=gnu99 to very old compilers
CVERSION := $(shell $(CC) -dM -E - < /dev/null | grep __STDC_VERSION__)
ifeq ($(CVERSION),)
CFLAGS := $(CFLAGS) -std=gnu99
endif

