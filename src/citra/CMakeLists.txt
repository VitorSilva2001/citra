set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/CMakeModules)

add_executable(citra
    citra.cpp
    citra.rc
    config.cpp
    config.h
    default_ini.h
    emu_window/emu_window_sdl2.cpp
    emu_window/emu_window_sdl2.h
    emu_window/emu_window_sdl2_gl.cpp
    emu_window/emu_window_sdl2_gl.h
    emu_window/emu_window_sdl2_sw.cpp
    emu_window/emu_window_sdl2_sw.h
    precompiled_headers.h
    resource.h
)

create_target_directory_groups(citra)

target_link_libraries(citra PRIVATE citra_common citra_core input_common network)
target_link_libraries(citra PRIVATE inih glad)
if (MSVC)
    target_link_libraries(citra PRIVATE getopt)
endif()
target_link_libraries(citra PRIVATE ${PLATFORM_LIBRARIES} SDL2::SDL2 Threads::Threads)

if(UNIX AND NOT APPLE)
    install(TARGETS citra RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/bin")
endif()

if (MSVC AND ENABLE_WEB_SERVICE AND OPENSSL_DLL_DIR)
    include(CopyCitraOpensslDeps)
    copy_citra_openssl_deps(citra)
endif()

if (CITRA_USE_PRECOMPILED_HEADERS)
    target_precompile_headers(citra PRIVATE precompiled_headers.h)
endif()

if (CITRA_BUNDLE_LIBRARIES)
    add_custom_command(TARGET citra
        POST_BUILD
        COMMAND ${CMAKE_COMMAND}
        -DCMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}
        -DTYPE=standalone
        -DEXECUTABLE_PATH=$<TARGET_FILE:citra>
        -P ${CMAKE_SOURCE_DIR}/CMakeModules/BundleLibraries.cmake
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
endif()
