cmake_minimum_required(VERSION 3.13)
set(WOLFBOOT_PATH ${CMAKE_CURRENT_LIST_DIR}/../lib/wolfBoot)
set(CMAKE_CXX_COMPILER arm-none-eabi-gcc)
set(LIB_PICO_RUNTIME_INIT=0)

#include(${PICO_SDK_PATH}/pico_sdk_init.cmake)
include("$ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake")

#set(PICOTOOL_FETCH_FROM_GIT_PATH ../wolfboot/build/picotool)
set(BOOT_STAGE2_FILE ${CMAKE_CURRENT_LIST_DIR}/boot2_empty.S)
set(PICO_NO_RUNTIME 1)

project(blink)

# initialize the Raspberry Pi Pico SDK
pico_sdk_init()

add_library(wolfboot STATIC)
target_sources(wolfboot PUBLIC
    ${WOLFBOOT_PATH}/src/libwolfboot.c
    ${WOLFBOOT_PATH}/hal/rp2350.c
)
target_include_directories(wolfboot PUBLIC 
   ${WOLFBOOT_PATH}/include
)
target_link_libraries(wolfboot 
	pico_stdlib
	hardware_flash
	)
target_compile_options(wolfboot PRIVATE
    -DPICO_RUNTIME_NO_INIT_BOOTROM_RESET=1
    -DPICO_RUNTIME_NO_INIT_CLOCKS=1
    -DPICO_TIME_DEFAULT_ALARM_POOL_DISABLED=1
)


add_executable(blink
    blink.c
    runtime.c
)

target_compile_options(blink PRIVATE
    -DPICO_RUNTIME_NO_INIT_BOOTROM_RESET=1
    -DPICO_RUNTIME_NO_INIT_CLOCKS=1
    -DPICO_TIME_DEFAULT_ALARM_POOL_DISABLED=1
)
target_compile_definitions(blink PRIVATE PICO_NO_RUNTIME=1)

pico_set_linker_script(blink ${WOLFBOOT_PATH}/hal/rp2350-app.ld)
target_link_libraries(blink 
	pico_stdlib
	wolfboot)

# create map/bin/hex/uf2 file etc.
pico_add_extra_outputs(blink)
pico_enable_stdio_usb(blink 0)
pico_enable_stdio_uart(blink 1)
target_compile_definitions(blink PRIVATE
    PICO_DEFAULT_UART_RX_PIN=17
    PICO_DEFAULT_UART_TX_PIN=16
)

