set(TARGET_NAME gzip)


if(LOW_LATENCY)
    # Compile the low latency version of the design
    message(STATUS "Compiling the Low Latency variant of the design")
    set(SOURCE_FILE gzip_ll.cpp crc32.cpp WriteGzip.cpp CompareGzip.cpp)
    set(DEVICE_SOURCE_FILE gzipkernel_ll.cpp)
    set(DEVICE_HEADER_FILE gzipkernel_ll.hpp)
else()
    # Compile the high bandwidth version of the design
    message(STATUS "Compiling the High Bandwidth variation of the design")
    set(SOURCE_FILE gzip.cpp crc32.cpp WriteGzip.cpp CompareGzip.cpp)
    set(DEVICE_SOURCE_FILE gzipkernel.cpp)
    set(DEVICE_HEADER_FILE gzipkernel.hpp)
endif()

set(EMULATOR_TARGET ${TARGET_NAME}.fpga_emu)
set(FPGA_TARGET ${TARGET_NAME}.fpga)

# FPGA board selection
if(NOT DEFINED FPGA_BOARD)
    set(FPGA_BOARD "intel_a10gx_pac:pac_a10")
    message(STATUS "FPGA_BOARD was not specified.\
                    \nConfiguring the design to run on the default FPGA board ${FPGA_BOARD} (Intel(R) PAC with Intel Arria(R) 10 GX FPGA). \
                    \nPlease refer to the README for information on board selection.")
else()
    message(STATUS "Configuring the design to run on FPGA board ${FPGA_BOARD}")
endif()

# This is a Windows-specific flag that enables error handling in host code
if(WIN32)
    set(WIN_FLAG "/EHsc")
endif()

# Set design parameters according to the selected chip
if(FPGA_BOARD MATCHES ".*a10.*")
    # A10 parameters
    set(NUM_ENGINES 1)
    if(DEFINED LOW_LATENCY)
        set(SEED "-Xsseed=4")
        set(NUM_REORDER "")
    else()
        set(SEED "-Xsseed=4")
        set(NUM_REORDER "")
    endif()
elseif(FPGA_BOARD MATCHES ".*s10.*")
    # S10 parameters
    set(NUM_ENGINES 2)
    if(DEFINED LOW_LATENCY)
        set(SEED "-Xsseed=19")
        set(NUM_REORDER "")
    else()
        set(SEED "-Xsseed=18")
        # For the High Bandwidth variant, specify 6 reordering units to improve global memory read bandwidth across 4 channels of DDR.
        # For Low Latency variant this is not necessary since only one channel of global memory is used (host memory).
        set(NUM_REORDER "-Xsnum-reorder=6")
    endif()
elseif(FPGA_BOARD MATCHES ".*agilex.*")
    # Agilex™
    set(NUM_ENGINES 2)
    if(DEFINED LOW_LATENCY)
        set(SEED "-Xsseed=1")
        set(NUM_REORDER "")
    else()
        set(SEED "-Xsseed=8")
        # For the High Bandwidth variant, specify 6 reordering units to improve global memory read bandwidth across 4 channels of DDR.
        # For Low Latency variant this is not necessary since only one channel of global memory is used (host memory).
        set(NUM_REORDER "-Xsnum-reorder=6")
    endif()
else()
    set(NUM_ENGINES 1)
    set(SEED "-Xsseed=1")
    set(NUM_REORDER "")
endif()

if(IGNORE_DEFAULT_SEED)
    set(SEED "")
endif()


# Presence of USM host allocations (and whether to turn on enable the low-latency target) is detected automatically by
# looking at the name of the BSP, or manually by the user when running CMake.
# E.g., cmake .. -DUSER_ENABLE_USM=1
if(FPGA_BOARD MATCHES ".*usm.*" OR USER_ENABLE_USM)
    set(USM_ENABLED 1)
elseif(LOW_LATENCY)
    # Low latency design requires USM, so error out
    message(FATAL_ERROR "Error: The Low Latency variant of the design requires USM host allocations")
endif()

message(STATUS "NUM_ENGINES=${NUM_ENGINES}")
message(STATUS "SEED=${SEED}")
message(STATUS "NUM_REORDER=${NUM_REORDER}")

# A DPC++ ahead-of-time (AoT) compile processes the device code in two stages.
# 1. The "compile" stage compiles the device code to an intermediate representation (SPIR-V).
# 2. The "link" stage invokes the compiler's FPGA backend before linking.
#    For this reason, FPGA backend flags must be passed as link flags in CMake.
set(EMULATOR_COMPILE_FLAGS "-Wall ${WIN_FLAG} -fintelfpga -g0 -DNUM_ENGINES=${NUM_ENGINES} -DFPGA_EMULATOR")
set(EMULATOR_LINK_FLAGS "-fintelfpga -DNUM_ENGINES=${NUM_ENGINES}")
set(HARDWARE_COMPILE_FLAGS "-Wall ${WIN_FLAG} -fintelfpga -DNUM_ENGINES=${NUM_ENGINES}")
set(HARDWARE_LINK_FLAGS "-fintelfpga -Xshardware -Xsparallel=2 -Xsopt-arg=\"-nocaching\" -Xsboard=${FPGA_BOARD} -DNUM_ENGINES=${NUM_ENGINES} ${USER_HARDWARE_FLAGS}")
# use cmake -D USER_HARDWARE_FLAGS=<flags> to set extra flags for FPGA backend compilation

###############################################################################
### FPGA Emulator
###############################################################################
add_executable(${EMULATOR_TARGET} ${SOURCE_FILE} ${DEVICE_SOURCE_FILE})
target_include_directories(${EMULATOR_TARGET} PRIVATE ../../../include)
set_target_properties(${EMULATOR_TARGET} PROPERTIES COMPILE_FLAGS "${EMULATOR_COMPILE_FLAGS}")
set_target_properties(${EMULATOR_TARGET} PROPERTIES LINK_FLAGS "${EMULATOR_LINK_FLAGS}")
add_custom_target(fpga_emu DEPENDS ${EMULATOR_TARGET})

###############################################################################
### Generate Report
###############################################################################
set(FPGA_EARLY_IMAGE ${TARGET_NAME}_report.a)
# The compile output is not an executable, but an intermediate compilation result unique to DPC++.
add_executable(${FPGA_EARLY_IMAGE} ${SOURCE_FILE} ${DEVICE_SOURCE_FILE})
target_include_directories(${FPGA_EARLY_IMAGE} PRIVATE ../../../include)
add_custom_target(report DEPENDS ${FPGA_EARLY_IMAGE})
set_target_properties(${FPGA_EARLY_IMAGE} PROPERTIES COMPILE_FLAGS "${HARDWARE_COMPILE_FLAGS}")
set_target_properties(${FPGA_EARLY_IMAGE} PROPERTIES LINK_FLAGS "${HARDWARE_LINK_FLAGS} ${NUM_REORDER} ${SEED} -fsycl-link=early")
# fsycl-link=early stops the compiler after RTL generation, before invoking Quartus

###############################################################################
### FPGA Hardware
###############################################################################
add_executable(${FPGA_TARGET} EXCLUDE_FROM_ALL ${SOURCE_FILE} ${DEVICE_SOURCE_FILE})
target_include_directories(${FPGA_TARGET} PRIVATE ../../../include)
add_custom_target(fpga DEPENDS ${FPGA_TARGET})
set_target_properties(${FPGA_TARGET} PROPERTIES COMPILE_FLAGS "${HARDWARE_COMPILE_FLAGS}")
set_target_properties(${FPGA_TARGET} PROPERTIES LINK_FLAGS "${HARDWARE_LINK_FLAGS} ${NUM_REORDER} ${SEED} -reuse-exe=${CMAKE_BINARY_DIR}/${FPGA_TARGET}")
# The -reuse-exe flag enables rapid recompilation of host-only code changes.
# See DPC++FPGA/GettingStarted/fast_recompile for details.



