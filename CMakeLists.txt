# CMake minimum version
cmake_minimum_required (VERSION 3.1)

# Project Infomation
project( FreeRTOS_Template )
enable_language(ASM)

# Reset output path
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)

# STDLIB
set (CMAKE_SHARED_LIBRARY_LINK_C_FLAGS)

# Set include path
include_directories (FreeRTOS/Include )
include_directories (Drivers/CMSIS/Device/Include )
include_directories (Drivers/CMSIS/Include )
include_directories (Drivers/STM32F10x_StdPeriph_Driver/inc)
include_directories (User/Include)

# The need build source path and build all files
aux_source_directory (User/Source/ DIR_USER_SRCS)
aux_source_directory (Drivers/CMSIS/Device/Source DIR_CMSIS_SRCS)

# CC AR LD AS
set (CMAKE_C_COMPILER "arm-none-eabi-gcc")
set (CMAKE_C_AR "arm-none-eabi-ar")
set (CMAKE_ASM_COMPILER "arm-none-eabi-gcc")

# CFLAGS
set(CMAKE_C_FLAGS "-g -mthumb -fno-builtin -mcpu=cortex-m3 -Wall -std=gnu99 -ffunction-sections -fdata-sections -fomit-frame-pointer -mabi=aapcs -fno-unroll-loops -ffast-math -ftree-vectorize" CACHE INTERNAL "c compiler flags")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D STM32F10X_HD -D USE_STDPERIPH_DRIVER")

# CXXFLAGS
set (CMAKE_CXX_FLAGS "-g -mthumb -fno-builtin -mcpu=cortex-m3 -Wall -std=c++11 -ffunction-sections -fdata-sections -fomit-frame-pointer -mabi=aapcs -fno-unroll-loops -ffast-math -ftree-vectorize" CACHE INTERNAL "cxx compiler flags")

# ASMFLAGS
set (CMAKE_ASM_FLAGS "-g -mthumb -mcpu=cortex-m3 -x assembler-with-cpp" CACHE INTERNAL "asm compiler flags")

# LDFLAGS
set (CMAKE_EXE_LINKER_FLAGS "-g -Wl,--gc-sections -mthumb -mcpu=cortex-m3 -mabi=aapcs" CACHE INTERNAL "executable linker flags")
set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${CMAKE_SOURCE_DIR}/ldscripts/stm32f103vet6.ld")

# Load the the extern STM32F10x_StdPeriph_Driver library
add_library (STM32F10x_StdPeriph_Driver STATIC IMPORTED)
set_property (TARGET STM32F10x_StdPeriph_Driver PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/Drivers/Build/lib/libSTM32F10x_StdPeriph_Driver.a)

# Load the the extern STM32F10x_RTOS library
add_library (STM32F10x_FreeRTOS STATIC IMPORTED)
set_property (TARGET STM32F10x_FreeRTOS PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/FreeRTOS/Build/lib/libSTM32F10x_FreeRTOS.a)

# Generate the target
add_executable (${CMAKE_PROJECT_NAME}.elf ${DIR_USER_SRCS} ${DIR_CMSIS_SRCS} ${CMAKE_SOURCE_DIR}/Drivers/CMSIS/Device/Source/startup/startup_stm32f10x_hd.s)

# Link the library to the target
target_link_libraries (${CMAKE_PROJECT_NAME}.elf STM32F10x_StdPeriph_Driver STM32F10x_FreeRTOS)

# Generate the binary file
add_custom_target (${CMAKE_PROJECT_NAME}.bin ALL arm-none-eabi-objcopy -Obinary "${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.elf" "${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.bin" DEPENDS ${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.elf)

# Generate the hex file
add_custom_target (${CMAKE_PROJECT_NAME}.hex ALL arm-none-eabi-objcopy -Oihex "${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.elf" "${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.hex" DEPENDS ${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.elf)

# Echo the size Infomation
add_custom_target (size ALL arm-none-eabi-size "${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.elf" DEPENDS ${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.elf)

# Make flash to the board by st-link
add_custom_target (flash COMMAND st-flash write ${EXECUTABLE_OUTPUT_PATH}/${CMAKE_PROJECT_NAME}.bin 0x8000000)

# Make clean-all
add_custom_target (clean-all COMMAND rm -rf ${CMAKE_BINARY_DIR}/*)
