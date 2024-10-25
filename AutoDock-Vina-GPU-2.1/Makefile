# Define paths (WORK_DIR defaults to the current directory if not specified during make invocation)
WORK_DIR ?= $(shell pwd)
BOOST_LIB_PATH ?= /usr/lib/x86_64-linux-gnu
OPENCL_LIB_PATH ?= /opt/nvidia/hpc_sdk/Linux_x86_64/24.9/cuda/12.6/targets/x86_64-linux

# Installation paths
INSTALL_DIR ?= /opt/AutoDock-Vina-GPU
KERNEL_DIR ?= $(INSTALL_DIR)/kernels

# Compilation and platform settings
BOX_SIZE_FLAG ?= -DSMALL_BOX
OPENCL_VERSION ?= 2
EXTRA_OUTPUT ?= 0

# Set the OPENCL_VERSION flag based on the command-line argument (either 2 or 3)
ifeq ($(OPENCL_VERSION), 2)
    OPENCL_FLAG := -DOPENCL_2_0
else ifeq ($(OPENCL_VERSION), 3)
    OPENCL_FLAG := -DOPENCL_3_0
else
    $(error Invalid OPENCL_VERSION specified. Use 2 or 3.)
endif

# Add EXTRA_OUTPUT flags if specified
ifeq ($(EXTRA_OUTPUT), 1)
    EXTRA_FLAGS := -DTIME_ANALYSIS -DDISPLAY_ADDITION_INFO
else
    EXTRA_FLAGS :=
endif

# Compiler, flags, and linker settings
CC := $(CC)
CXX := $(CXX)
CFLAGS := -O3 -DNDEBUG $(BOX_SIZE_FLAG) $(OPENCL_FLAG) $(EXTRA_FLAGS) -DBOOST_TIMER_ENABLE_DEPRECATED
LDFLAGS := -L$(BOOST_LIB_PATH) -L$(OPENCL_LIB_PATH)/lib

# Supress warning about unused variables
CFLAGS += --diag_suppress=declared_but_not_referenced
CFLAGS += --diag_suppress=virtual_function_decl_hidden
CFLAGS += --diag_suppress=deprecated_entity

# Include paths and libraries
INCLUDE_PATHS := -I$(BOOST_LIB_PATH) -I$(OPENCL_LIB_PATH)/include -I$(WORK_DIR)/lib -I$(WORK_DIR)/OpenCL/inc
LIBS := -lboost_program_options -lboost_system -lboost_filesystem -lboost_thread -lOpenCL -lstdc++ -lstdc++fs -lm -lpthread
LIBS += -L$(BOOST_LIB_PATH) -L$(OPENCL_LIB_PATH)/lib

# Source files
SRC := ./lib/*.cpp ./OpenCL/src/wrapcl.cpp

# Targets
TARGET := AutoDock-Vina-GPU-2-1

# Default target
all: $(TARGET)

# Build main target
$(TARGET): ./main/main.cpp $(SRC)
	$(CXX) -o $(TARGET) $(INCLUDE_PATHS) $(CFLAGS) ./main/main.cpp $(SRC) $(LIBS) $(LDFLAGS)

# Build option for kernel source
source: ./main/main.cpp
	$(CXX) -o $(TARGET) $(INCLUDE_PATHS) $(CFLAGS) ./main/main.cpp $(SRC) $(LIBS) $(LDFLAGS) -DBUILD_KERNEL_FROM_SOURCE

# Debug build option
debug: ./main/main.cpp
	$(CXX) -o $(TARGET) $(INCLUDE_PATHS) -g ./main/main.cpp $(SRC) $(LIBS) $(LDFLAGS) -DBUILD_KERNEL_FROM_SOURCE

# Clean target
clean:
	rm -f $(TARGET)

# Install target using 'install' command
install: $(TARGET)
	mkdir -p $(INSTALL_DIR)
	mkdir -p $(KERNEL_DIR)
	install -m 755 $(TARGET) $(INSTALL_DIR)/
	install -m 644 ./kernels/*.bin $(KERNEL_DIR)/
