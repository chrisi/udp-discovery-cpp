cmake_minimum_required(VERSION 3.16)

project(udp-discovery VERSION 1.0.0)

option(BUILD_EXAMPLE "Build example application." OFF)
option(BUILD_TOOL "Build udp-discovery-tool application." OFF)
option(BUILD_TEST "Build test." ON)

set(LIB_SOURCES
        udp_discovery_ip_port.cpp
        udp_discovery_peer.cpp
        udp_discovery_protocol.cpp)
set(LIB_HEADERS
        udp_discovery_ip_port.hpp
        udp_discovery_peer.hpp
        udp_discovery_peer_parameters.hpp
        udp_discovery_protocol.hpp
        udp_discovery_protocol_version.hpp)

add_library(${PROJECT_NAME} STATIC ${LIB_SOURCES} ${LIB_HEADERS})
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 98)

if (UNIX)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pthread")
endif ()

if (BUILD_EXAMPLE)
    set(DISCOVERY_EXAMPLE_SOURCES discovery_example.cpp)
    set(DISCOVERY_EXAMPLE_LIBS ${PROJECT_NAME})

    if (APPLE)
    elseif (UNIX)
        set(DISCOVERY_EXAMPLE_LIBS ${DISCOVERY_EXAMPLE_LIBS} rt)
    endif ()

    if (WIN32)
        set(DISCOVERY_EXAMPLE_LIBS ${DISCOVERY_EXAMPLE_LIBS} Ws2_32)
    endif (WIN32)

    add_executable(udp-discovery-example ${DISCOVERY_EXAMPLE_SOURCES})
    target_link_libraries(udp-discovery-example ${DISCOVERY_EXAMPLE_LIBS})
    set_property(TARGET udp-discovery-example PROPERTY CXX_STANDARD 98)
endif ()

if (BUILD_TOOL)
    set(DISCOVERY_TOOL_SOURCES discovery_tool.cpp)
    set(DISCOVERY_TOOL_LIBS ${PROJECT_NAME})

    if (APPLE)
    elseif (UNIX)
        set(DISCOVERY_TOOL_LIBS ${DISCOVERY_TOOL_LIBS} rt)
    endif ()

    add_executable(udp-discovery-tool ${DISCOVERY_TOOL_SOURCES})
    target_link_libraries(udp-discovery-tool ${DISCOVERY_TOOL_LIBS})
    set_property(TARGET udp-discovery-example PROPERTY CXX_STANDARD 98)
endif ()

if (BUILD_TEST)
    enable_testing()

    add_executable(udp-discovery-protocol-test udp_discovery_protocol.cpp udp_discovery_protocol_test.cpp)
    set_property(TARGET udp-discovery-protocol-test PROPERTY CXX_STANDARD 98)
    add_test(udp-discovery-protocol-test udp-discovery-protocol-test)

    add_executable(udp-discovery-peer-e2e-test udp_discovery_protocol.cpp udp_discovery_peer.cpp udp_discovery_peer_e2e_test.cpp)
    set_property(TARGET udp-discovery-peer-e2e-test PROPERTY CXX_STANDARD 98)
    add_test(udp-discovery-peer-e2e-test udp-discovery-peer-e2e-test)
endif ()

set(PUB_HEADERS
        udp_discovery_peer.hpp
        udp_discovery_ip_port.hpp
        udp_discovery_protocol.hpp
        udp_discovery_peer_parameters.hpp
        udp_discovery_protocol_version.hpp
        udp_discovery_discovered_peer.hpp
)

set_target_properties(${PROJECT_NAME}
        PROPERTIES PUBLIC_HEADER "${PUB_HEADERS}"
)

install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}Targets
        INCLUDES DESTINATION "include/udp-discovery"
        PUBLIC_HEADER DESTINATION "include/udp-discovery"
)

install(EXPORT ${PROJECT_NAME}Targets
        FILE ${PROJECT_NAME}Targets.cmake
        NAMESPACE gtidev::
        DESTINATION lib/cmake/${PROJECT_NAME}
)

# Create a version file for the library
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
)

# Install the package configuration files
install(FILES
        ${PROJECT_NAME}Config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION lib/cmake/${PROJECT_NAME}
)

set(CPACK_GENERATOR "DEB")
set(CPACK_PACKAGE_NAME ${CMAKE_PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "GTIDEV")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Christian Gebauer")

set(CPACK_PACKAGE_FILE_NAME "udp-discovery") #provide a static name for curl upload in Jenkinsfile

include(CPack)
