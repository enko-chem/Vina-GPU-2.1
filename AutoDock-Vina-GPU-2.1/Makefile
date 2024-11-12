# Define paths (WORK_DIR defaults to the current directory if not specified during make invocation)
WORK_DIR ?= $(shell pwd)
BOOST_LIB_PATH ?= /usr/lib/x86_64-linux-gnu

# Installation paths
INSTALL_DIR ?= /opt/AutoDock-Vina-GPU
KERNEL_DIR ?= $(INSTALL_DIR)/kernels

# NVIDIA HPC SDK paths inferred from NVHPC_ROOT
CUDA_INC_PATH := $(NVHPC_ROOT)/cuda/include
CUDA_LIB_PATH := $(NVHPC_ROOT)/cuda/lib64
COMPILER_LIB_PATH := $(NVHPC_ROOT)/compilers/lib
MATH_LIB_PATH := $(NVHPC_ROOT)/math_libs/lib64
NCCL_LIB_PATH := $(NVHPC_ROOT)/comm_libs/nccl/lib
NVSHMEM_LIB_PATH := $(NVHPC_ROOT)/comm_libs/nvshmem/lib

# Compilation and platform settings
BOX_SIZE ?= small
OPENCL_VERSION ?= 2
EXTRA_OUTPUT ?= 0

# Set the BOX_SIZE_FLAG based on the command-line argument (either small or large)
ifeq ($(BOX_SIZE), small)
	BOX_SIZE_FLAG := -DSMALL_BOX
else ifeq ($(BOX_SIZE), large)
	BOX_SIZE_FLAG := -DLARGE_BOX
else
    $(error Invalid BOX_SIZE specified. Use 'small' or 'large'.)
endif

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
    EXTRA_FLAGS := -DTIME_ANALYSIS -DDISPLAY_ADDITION_INFO -DDISPLAY_SUCCESS
else
    EXTRA_FLAGS :=
endif

# Compiler, flags, and linker settings
CC := $(CC)
CXX := $(CXX)
CFLAGS := -fast -DNDEBUG -DNVIDIA_PLATFORM $(BOX_SIZE_FLAG) $(OPENCL_FLAG) $(EXTRA_FLAGS) -DBOOST_TIMER_ENABLE_DEPRECATED

# Add -fPIE if the compiler is nvc or nvc++
CC_NAME := $(notdir $(CC))
CXX_NAME := $(notdir $(CXX))

# Conditionally add -fPIE if the compiler is nvc or nvc++
ifeq ($(CC_NAME),nvc)
    CC += -fPIC -Xptxas -v #-gpu=rdc -gpu=ccall
endif
ifeq ($(CXX_NAME),nvc++)
    CXX += -fPIC -Xptxas -v #-gpu=rdc -gpu=ccall
endif

# Supress warning about unused variables
CFLAGS += --diag_suppress=declared_but_not_referenced
CFLAGS += --diag_suppress=virtual_function_decl_hidden
CFLAGS += --diag_suppress=deprecated_entity

LDFLAGS := -L$(BOOST_LIB_PATH) -L$(CUDA_LIB_PATH) -L$(COMPILER_LIB_PATH) -L$(MATH_LIB_PATH) -L$(NCCL_LIB_PATH) -L$(NVSHMEM_LIB_PATH)

# Include paths and libraries
INCLUDE_PATHS := -I$(BOOST_LIB_PATH) -I$(CUDA_INC_PATH) -I$(WORK_DIR)/lib -I$(WORK_DIR)/OpenCL/inc
LIBS := -lboost_program_options -lboost_system -lboost_filesystem -lboost_thread -lOpenCL -lstdc++ -lstdc++fs -lm -lpthread

# Source files and object files
SRC := ./lib/*.cpp ./OpenCL/src/wrapcl.cpp

# Targets
TARGET := AutoDock-Vina-GPU-2-1

# Default target
all: $(TARGET)

$(TARGET): ./main/main.cpp
	$(CXX) -o $(TARGET) $(INCLUDE_PATHS) $(CFLAGS) ./main/main.cpp $(SRC) $(LIBS) $(LDFLAGS)

# Build option for kernel source
source: ./main/main.cpp
	@echo "The value of NVHPC_ROOT is: $(NVHPC_ROOT)"
	$(CXX) -o $(TARGET) $(INCLUDE_PATHS) -DBUILD_KERNEL_FROM_SOURCE $(CFLAGS) ./main/main.cpp $(SRC) $(LIBS) $(LDFLAGS)

# Debug build option
debug: ./main/main.cpp
	$(CXX) -o $(TARGET) $(INCLUDE_PATHS) -g -DBUILD_KERNEL_FROM_SOURCE $(CFLAGS) ./main/main.cpp $(SRC) $(LIBS) $(LDFLAGS)

# Clean target
clean:
	rm -f $(TARGET)
	rm -f *.o

# Create a wrapper script to run the executable with the --opencl_binary_path argument
runscript: $(TARGET)
	echo '#!/bin/bash' > vina-gpu
	echo '$(INSTALL_DIR)/$(TARGET) --opencl_binary_path=$(KERNEL_DIR) "$$@"' >> vina-gpu

# Install target using 'install' command
install: $(TARGET) runscript
	install -d $(INSTALL_DIR) $(KERNEL_DIR)
	install -m 755 $(TARGET) $(INSTALL_DIR)/
	install -m 664 Kernel?_Opt.bin $(KERNEL_DIR)/
	install -m 755 vina-gpu /usr/local/bin/

uninstall:
	rm -rf $(KERNEL_DIR)
	rm -rf $(INSTALL_DIR)
	rm -rf /usr/loca/bin/vina-gpu
