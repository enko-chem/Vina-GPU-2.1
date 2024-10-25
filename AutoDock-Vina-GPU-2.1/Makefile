# Define paths (WORK_DIR defaults to the current directory if not specified during make invocation)
WORK_DIR ?= $(shell pwd)
BOOST_LIB_PATH ?= /usr/lib/x86_64-linux-gnu
OPENCL_LIB_PATH ?= /opt/nvidia/hpc_sdk/Linux_x86_64/24.9/cuda/12.6/targets/x86_64-linux

# Compilation and platform settings
OPENCL_VERSION := -DOPENCL_2_0
GPU_PLATFORM := -DNVIDIA_PLATFORM
DOCKING_BOX_SIZE := -DSMALL_BOX

# Compiler, flags, and linker settings (use environment variables for flexibility)
CC := $(CC)       # Defaulting to nvc if set in the environment
CXX := $(CXX)     # Defaulting to nvc++ if set in the environment
CFLAGS := -O3 -DNDEBUG $(OPENCL_VERSION) $(GPU_PLATFORM) $(DOCKING_BOX_SIZE) -DBOOST_TIMER_ENABLE_DEPRECATED
LDFLAGS := -L$(BOOST_LIB_PATH) -L$(OPENCL_LIB_PATH)/lib

# Supress warning about unused variables
CFLAGS += --diag_suppress=declared_but_not_referenced
CFLAGS += --diag_suppress=virtual_function_decl_hidden
CFLAGS += --diag_suppress=deprecated_entity

# Include paths and libraries
INCLUDE_PATHS := -I$(BOOST_LIB_PATH) -I$(OPENCL_LIB_PATH)/include -I$(WORK_DIR)/lib -I$(WORK_DIR)/OpenCL/inc
LIBS := -lboost_program_options -lboost_system -lboost_filesystem -lOpenCL -lstdc++ -lstdc++fs -lm -lpthread
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
