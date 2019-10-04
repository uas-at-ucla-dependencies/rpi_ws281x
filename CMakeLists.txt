cmake_minimum_required(VERSION 3.10)

file(READ "version" VERSION_FROM_FILE)
string(STRIP "${VERSION_FROM_FILE}" VERSION_FROM_FILE)

project(rpi_ws281x LANGUAGES C VERSION ${VERSION_FROM_FILE})

configure_file(version.h.in version.h)
include_directories("${CMAKE_CURRENT_BINARY_DIR}")

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
set(cmake_package_name ${PROJECT_NAME})
set(targets_export_name ${cmake_package_name}Targets CACHE INTERNAL "")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated" CACHE INTERNAL "")
set(cmake_files_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${cmake_package_name}")
set(version_file "${generated_dir}/${cmake_package_name}ConfigVersion.cmake")

set(ws2811_SRCS
    mailbox.c
    ws2811.c
    pwm.c
    pcm.c
    dma.c
    rpihw.c
)
set(ws2811_HDRS
    mailbox.h
    ws2811.h
    pwm.h
    pcm.h
    dma.h
    rpihw.h
)

add_library(ws2811 ${ws2811_SRCS} ${ws2811_HDRS})
target_include_directories(ws2811 SYSTEM INTERFACE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOUCE_DIR}>"
    "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")

add_executable(ws2811_test main.c)
target_link_libraries(ws2811_test PUBLIC ws2811)

install(FILES ${ws2811_HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ws2811)
install(TARGETS ws2811
    EXPORT ${targets_export_name}
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")

write_basic_package_version_file(${version_file} VERSION ${rpi_ws281x_VERSION} COMPATIBILITY AnyNewerVersion)
install(EXPORT ${targets_export_name}
    NAMESPACE ${cmake_package_name}::
    DESTINATION ${cmake_files_install_dir})
set(config_file "${generated_dir}/${cmake_package_name}Config.cmake")
configure_package_config_file("Config.cmake.in"
    "${config_file}" INSTALL_DESTINATION ${cmake_files_install_dir})
install(FILES ${version_file} ${config_file}
    DESTINATION ${cmake_files_install_dir})
