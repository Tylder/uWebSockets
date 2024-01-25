cmake_minimum_required(VERSION 3.21)
set(CMAKE_C_STANDARD 11)

file(GLOB_RECURSE HEADERS src/*.h)

MESSAGE(WARNING "HEADERS: " ${HEADERS})
MESSAGE(WARNING "CMAKE_SOURCE_DIR: " ${CMAKE_SOURCE_DIR})
MESSAGE(WARNING "CMAKE_BINARY_DIR: " ${CMAKE_BINARY_DIR})
MESSAGE(WARNING "CMAKE_CURRENT_SOURCE_DIR: " ${CMAKE_CURRENT_SOURCE_DIR})
MESSAGE(WARNING "CMAKE_CURRENT_LIST_DIR: " ${CMAKE_CURRENT_LIST_DIR})
MESSAGE(WARNING "CMAKE_INSTALL_INCLUDEDIR: " ${CMAKE_INSTALL_INCLUDEDIR})

add_library(uWebSockets ${HEADERS})
add_library(uWebSockets::uWebSockets ALIAS uWebSockets)
target_compile_definitions(uWebSockets PRIVATE LIBUS_NO_SSL)

find_package(ZLIB REQUIRED)

#target_include_directories(uWebSockets PUBLIC ${uWebSockets_content_SOURCE_DIR}/src)
set(TARGETS_EXPORT_NAME "uWebSocketsTargets")
set(CONFIG_EXPORT_NAME "uWebSocketsConfig")

target_link_libraries(uWebSockets PUBLIC
    uSockets
    ${ZLIB_LIBRARIES}
)

source_group( # IDEs should put the headers in a nice place
        TREE "${CMAKE_CURRENT_SOURCE_DIR}/src"
        PREFIX "Header Files"
        FILES ${HEADERS})

option(BUILD_SHARED_LIBS "Build shared library" ON)

MESSAGE(WARNING "CMAKE_INSTALL_INCLUDEDIR:  ${CMAKE_SOURCE_DIR}/${CMAKE_INSTALL_INCLUDEDIR}")
MESSAGE(WARNING "CMAKE_INSTALL_SOURCEDIR: ${CMAKE_SOURCE_DIR}/${CMAKE_INSTALL_SOURCEDIR}")
MESSAGE(WARNING "src: ${CMAKE_CURRENT_SOURCE_DIR}/src")
MESSAGE(WARNING "src2: ${CMAKE_SOURCE_DIR}/src")

target_include_directories(uWebSockets
        PUBLIC
#        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/${CMAKE_INSTALL_INCLUDEDIR}>
#        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>

#        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
#        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/extern> ## makes it #include "extern/uWebSockets/src/..."
        $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/extern>

#        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/${CMAKE_INSTALL_INCLUDEDIR}>
#        $<BUILD_INTERFACE:"${CMAKE_SOURCE_DIR}/extern/">
#        $<BUILD_INTERFACE:src>
        $<INSTALL_INTERFACE:src> # <prefix>/include/mylib
        PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}"  #/src
)

set_target_properties(uWebSockets PROPERTIES
        LINKER_LANGUAGE CXX
        ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
        LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
        PUBLIC_HEADER "${HEADERS}"
)

install(DIRECTORY
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

include(GNUInstallDirs) # make cache variables for install destinations

install(TARGETS uWebSockets uSockets
        EXPORT ${TARGETS_EXPORT_NAME}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/uWebSockets")

######
include(CMakePackageConfigHelpers)

install(EXPORT ${TARGETS_EXPORT_NAME}
        FILE "${TARGETS_EXPORT_NAME}.cmake"
        NAMESPACE uWebSockets::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/uWebSockets
)

export(EXPORT ${TARGETS_EXPORT_NAME}
        FILE "${CMAKE_BINARY_DIR}/cmake/${TARGETS_EXPORT_NAME}.cmake"
        NAMESPACE uWebSockets::
)

install(FILES
        "${CMAKE_BINARY_DIR}/cmake/${CONFIG_EXPORT_NAME}.cmake"
        "${CMAKE_BINARY_DIR}/cmake/${CONFIG_EXPORT_NAME}-version.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/uWebSockets
)