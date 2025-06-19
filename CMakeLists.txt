cmake_minimum_required(VERSION 3.24)

add_library(freemodbus STATIC)

if(NOT TARGET freemodbus_port)
    message(FATAL_ERROR " freemodbus_port target not specified.  Please specify a cmake target that defines the include port.h:\n"
        "add_library(freemodbus_port\n"
        "    STATIC\n"
        "    ${CMAKE_SOURCE_DIR}/libs/port/portevent.c\n"
        "    ${CMAKE_SOURCE_DIR}/libs/port/portother.c\n"
        "    ${CMAKE_SOURCE_DIR}/libs/port/porttcp.c\n"
        ")\n"
        "\n"
        "target_include_directories(freemodbus_port SYSTEM\n"
        "    PUBLIC\n"
        "    ${CMAKE_SOURCE_DIR}/libs/port\n"
        ")\n"
        "\n"
        "target_link_libraries(freemodbus_port PUBLIC lwip)\n"
        "\n"
        "set(FREEMB_MB_ASCII_ENABLED ON)\n"
        "set(FREEMB_MB_RTU_ENABLED ON)\n"
        "set(FREEMB_MB_TCP_ENABLED ON)\n"
    )
endif()

target_link_libraries(freemodbus PUBLIC freemodbus_port)
target_link_libraries(freemodbus_port PUBLIC freemodbus)

if(FREEMB_MB_ASCII_ENABLED)
    target_compile_definitions(freemodbus PUBLIC -DMB_ASCII_ENABLED=1)
    set(MBASCII_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/modbus/ascii/mbascii.c)
    set(MBASCII_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/modbus/ascii)
endif()
if(FREEMB_MB_RTU_ENABLED)
    target_compile_definitions(freemodbus PUBLIC -DMB_RTU_ENABLED=1)
    set(MBRTU_SOURCE
        ${CMAKE_CURRENT_SOURCE_DIR}/modbus/rtu/mbcrc.c
        ${CMAKE_CURRENT_SOURCE_DIR}/modbus/rtu/mbrtu.c
    )
    set(MBRTU_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/modbus/rtu)
endif()
if(FREEMB_MB_TCP_ENABLED)
    target_compile_definitions(freemodbus PUBLIC -DMB_TCP_ENABLED=1)
    set(MBTCP_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/modbus/tcp/mbtcp.c)
    set(MBTCP_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/modbus/tcp)
endif()

target_include_directories(freemodbus PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/include
    ${MBASCII_HEADER}
    ${MBRTU_HEADER}
    ${MBTCP_HEADER}
)

target_sources(freemodbus PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/mb.c
    # Functions
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbfunccoils.c
    #${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbfuncdiag.c
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbfuncdisc.c
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbfuncholding.c
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbfuncinput.c
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbfuncother.c
    ${CMAKE_CURRENT_SOURCE_DIR}/modbus/functions/mbutils.c
    ${MBASCII_SOURCE}
    ${MBRTU_SOURCE}
    ${MBTCP_SOURCE}
)
