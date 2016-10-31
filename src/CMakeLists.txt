MESSAGE(STATUS "\n Checking ${CMAKE_CURRENT_SOURCE_DIR}")

# lwip configured with TCP+ipv4+sockets+TexasInstruments lwiplib
SET(COMPONENT_NAME "lwip")
string(TOLOWER ${COMPONENT_NAME} LIBRARY_NAME)

# For now same version as the whole project
SET(COMPONENT_VERSION 1.0.0 )

set(generated_dir "${CMAKE_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${COMPONENT_NAME}ConfigVersion.cmake")
set(component_config "${generated_dir}/${COMPONENT_NAME}Config.cmake")
set(targets_export_name "${COMPONENT_NAME}Targets")
set(namespace "${PROJECT_NAMESPACE}")


#------------------------------
# Dependencies
#------------------------------

# No dependencies so far
SET(MODULE_DEPS "")

find_package(TivaWare COMPONENTS driverlib CONFIG REQUIRED)
LIST(APPEND MODULE_DEPS TivaWare::driverlib )

find_package(FreeRTOS CONFIG REQUIRED)
LIST(APPEND MODULE_DEPS freertos)

#------------------------------
# normal sources
#------------------------------
set(SRCS
  # Api sources
  ${CMAKE_CURRENT_SOURCE_DIR}/api/sockets.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/err.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/api_lib.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/netifapi.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/api_msg.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/netbuf.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/netdb.c
  ${CMAKE_CURRENT_SOURCE_DIR}/api/tcpip.c
  # Core 
  ${CMAKE_CURRENT_SOURCE_DIR}/core/stats.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/mem.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/tcp_in.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/tcp.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/sys.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/dns.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/timers.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/def.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/init.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/udp.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/netif.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/raw.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/tcp_out.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/pbuf.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/memp.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/dhcp.c
  # IPv4
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/inet.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/autoip.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/ip_addr.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/icmp.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/ip_frag.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/ip.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/inet_chksum.c
  ${CMAKE_CURRENT_SOURCE_DIR}/core/ipv4/igmp.c
  # netif (only ARP)
  ${CMAKE_CURRENT_SOURCE_DIR}/netif/etharp.c
  # TivaWare port
  ${CMAKE_CURRENT_SOURCE_DIR}/tiva-tm4c129/perf.c
  ${CMAKE_CURRENT_SOURCE_DIR}/tiva-tm4c129/sys_arch.c
  ${CMAKE_CURRENT_SOURCE_DIR}/tiva-tm4c129/tiva-tm4c129.c
  # lwiplib
  ${CMAKE_CURRENT_SOURCE_DIR}/lwiplib/lwiplib.c
  )


#------------------------------
# Output a library
#------------------------------
ADD_LIBRARY(${LIBRARY_NAME} ${SRCS})


MESSAGE(STATUS ${LWIP_COMPILE_DEFINITIONS})

SET_TARGET_PROPERTIES(
  ${LIBRARY_NAME} PROPERTIES
  COMPILE_DEFINITIONS "${LWIP_COMPILE_DEFINITIONS}"
  COMPILE_FLAGS "-Wno-unused-parameter -Wno-empty-body"
  OUTPUT_NAME ${LIBRARY_NAME}
  VERSION ${LIBRARY_VERSION}
  SOVERSION ${LIBRARY_VERSION}
  )


target_include_directories(
  ${LIBRARY_NAME}
  PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}/>
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}/arch>
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}/ipv4>
  INTERFACE
  $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}/${namespace}/>
  $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}/${namespace}/ipv4>
  )

# Add dependencies to library to pass
# dependencies on to client executable
TARGET_LINK_LIBRARIES( ${LIBRARY_NAME}
    ${MODULE_DEPS}
    )

#-----

#-----
# Install Targets Library + Headers
#-----
MESSAGE(STATUS "** Generating Package Configurations **")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${version_config}
  VERSION ${LIBRARY_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Note: variable 'targets_export_name' used
configure_file("${CMAKE_SOURCE_DIR}/cmake/Config.cmake.in"
  "${component_config}" @ONLY)

INSTALL(TARGETS ${LIBRARY_NAME}
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
  LIBRARY DESTINATION ${INSTALL_LIB_DIR}
  RUNTIME DESTINATION ${INSTALL_BIN_DIR}
  # this will only add -I<prefix>/include to client compile flags
  # The users of the library have to #include "driverlib/foo.h"
  INCLUDES DESTINATION ${INSTALL_INCLUDE_DIR}/${namespace}
)


# Core headers
INSTALL(DIRECTORY
  ${PROJECT_INCLUDE_DIR}/
  COMPONENT DEVELOP
  DESTINATION ${INSTALL_INCLUDE_DIR}/${namespace}
  )

INSTALL(FILES
  ${PROJECT_SOURCE_DIR}/lwipopts.h
  COMPONENT DEVELOP
  DESTINATION ${INSTALL_INCLUDE_DIR}/${namespace}
  )

INSTALL(
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  NAMESPACE "${namespace}::"
  DESTINATION "${INSTALL_CMAKE_DIR}/${namespace}"
  )

INSTALL(
  FILES "${component_config}" "${version_config}"
  COMPONENT DEVELOP
  DESTINATION "${INSTALL_CMAKE_DIR}/${namespace}"
  )
