# Set location of base MicroPython directory, and this port's directory.
get_filename_component(MPY_DIR ${PROJECT_DIR}/../.. ABSOLUTE)
set(MPY_PORT_DIR ${PROJECT_DIR})
set(MPY_BOARD_DIR ${PROJECT_DIR}/boards/GENERIC)

# Include core source components.
include(${MPY_DIR}/py/py.cmake)
include(${MPY_DIR}/extmod/extmod.cmake)

set(SOURCE_EXTMOD_EXTRA
    ${MPY_DIR}/extmod/modonewire.c
)

set(SOURCE_LIB
    ${MPY_DIR}/lib/littlefs/lfs1.c
    ${MPY_DIR}/lib/littlefs/lfs1_util.c
    ${MPY_DIR}/lib/littlefs/lfs2.c
    ${MPY_DIR}/lib/littlefs/lfs2_util.c
    ${MPY_DIR}/lib/mbedtls_errors/mp_mbedtls_errors.c
    ${MPY_DIR}/lib/mp-readline/readline.c
    ${MPY_DIR}/lib/netutils/netutils.c
    ${MPY_DIR}/lib/oofatfs/ff.c
    ${MPY_DIR}/lib/oofatfs/ffunicode.c
    ${MPY_DIR}/lib/timeutils/timeutils.c
    ${MPY_DIR}/lib/utils/interrupt_char.c
    ${MPY_DIR}/lib/utils/stdout_helpers.c
    ${MPY_DIR}/lib/utils/sys_stdio_mphal.c
    ${MPY_DIR}/lib/utils/pyexec.c
)

set(SOURCE_DRIVERS
    ${MPY_DIR}/drivers/bus/softspi.c
    ${MPY_DIR}/drivers/dht/dht.c
)

set(SOURCE_PORT
    ${PROJECT_DIR}/main.c
    ${PROJECT_DIR}/uart.c
    ${PROJECT_DIR}/gccollect.c
    ${PROJECT_DIR}/mphalport.c
    ${PROJECT_DIR}/fatfs_port.c
    ${PROJECT_DIR}/help.c
    ${PROJECT_DIR}/modutime.c
    ${PROJECT_DIR}/moduos.c
    ${PROJECT_DIR}/machine_timer.c
    ${PROJECT_DIR}/machine_pin.c
    ${PROJECT_DIR}/machine_touchpad.c
    ${PROJECT_DIR}/machine_adc.c
    ${PROJECT_DIR}/machine_dac.c
    ${PROJECT_DIR}/machine_i2c.c
    ${PROJECT_DIR}/machine_pwm.c
    ${PROJECT_DIR}/machine_uart.c
    ${PROJECT_DIR}/modmachine.c
    ${PROJECT_DIR}/modnetwork.c
    ${PROJECT_DIR}/network_lan.c
    ${PROJECT_DIR}/network_ppp.c
    ${PROJECT_DIR}/mpnimbleport.c
    ${PROJECT_DIR}/modsocket.c
    ${PROJECT_DIR}/modesp.c
    ${PROJECT_DIR}/esp32_partition.c
    ${PROJECT_DIR}/esp32_rmt.c
    ${PROJECT_DIR}/esp32_ulp.c
    ${PROJECT_DIR}/modesp32.c
    ${PROJECT_DIR}/espneopixel.c
    ${PROJECT_DIR}/machine_hw_spi.c
    ${PROJECT_DIR}/machine_wdt.c
    ${PROJECT_DIR}/mpthreadport.c
    ${PROJECT_DIR}/machine_rtc.c
    ${PROJECT_DIR}/machine_sdcard.c
)

set(SOURCE_QSTR
    ${SOURCE_PY}
    ${SOURCE_EXTMOD}
    ${SOURCE_EXTMOD_EXTRA}
    ${SOURCE_LIB}
    ${SOURCE_PORT}
)

set(IDF_COMPONENTS
    app_update
    bootloader_support
    driver
    esp32
    esp_common
    esp_eth
    esp_event
    esp_netif
    esp_ringbuf
    esp_rom
    esp_wifi
    freertos
    heap
    log
    lwip
    mbedtls
    mdns
    newlib
    nvs_flash
    sdmmc
    soc
    spi_flash
    tcpip_adapter
    ulp
    vfs
    xtensa
)

# Register the main IDF component.
idf_component_register(
    SRCS
        ${SOURCE_PY}
        ${SOURCE_EXTMOD}
        ${SOURCE_EXTMOD_EXTRA}
        ${SOURCE_LIB}
        ${SOURCE_DRIVERS}
        ${SOURCE_PORT}
    INCLUDE_DIRS
        ${MPY_DIR}
        ${MPY_PORT_DIR}
        ${MPY_BOARD_DIR}
        ${CMAKE_BINARY_DIR}
    REQUIRES
        ${IDF_COMPONENTS}
)

# Set the MicroPython target as the current (main) IDF component target.
set(MICROPYTHON_TARGET ${COMPONENT_TARGET})

# Define mpy-cross flags and frozen manifest
set(MPY_CROSS_FLAGS -march=xtensawin)
set(FROZEN_MANIFEST ${PROJECT_DIR}/boards/manifest.py)

# Set compile options for this port.
target_compile_definitions(${MICROPYTHON_TARGET} PUBLIC
    MICROPY_ESP_IDF_4=1
    MICROPY_VFS_FAT=1
    MICROPY_VFS_LFS2=1
    FFCONF_H=\"${OOFATFS_DIR}/ffconf.h\"
    LFS1_NO_MALLOC LFS1_NO_DEBUG LFS1_NO_WARN LFS1_NO_ERROR LFS1_NO_ASSERT
    LFS2_NO_MALLOC LFS2_NO_DEBUG LFS2_NO_WARN LFS2_NO_ERROR LFS2_NO_ASSERT
)

# Disable some warnings to keep the build output clean.
target_compile_options(${MICROPYTHON_TARGET} PUBLIC
    -Wno-clobbered
    -Wno-deprecated-declarations
    -Wno-missing-field-initializers
)

# Collect all of the include directories and compile definitions for the IDF components.
foreach(comp ${IDF_COMPONENTS})
    get_target_property(type __idf_${comp} TYPE)
    set(_inc OFF)
    set(_def OFF)
    if(${type} STREQUAL STATIC_LIBRARY)
        get_target_property(_inc __idf_${comp} INCLUDE_DIRECTORIES)
        get_target_property(_def __idf_${comp} COMPILE_DEFINITIONS)
    elseif(${type} STREQUAL INTERFACE_LIBRARY)
        get_target_property(_inc __idf_${comp} INTERFACE_INCLUDE_DIRECTORIES)
        get_target_property(_def __idf_${comp} INTERFACE_COMPILE_DEFINITIONS)
    endif()
    if(_inc)
        list(APPEND MPY_CPP_INC_EXTRA ${_inc})
    endif()
    if(_def)
        list(APPEND MPY_CPP_DEF_EXTRA ${_def})
    endif()
endforeach()

# Include the main MicroPython cmake rules.
include(${MPY_DIR}/py/mkrules.cmake)
