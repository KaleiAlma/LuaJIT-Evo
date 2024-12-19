# i can't be assed to write and/or maintain a proper cmake build script for luajit.

cmake_minimum_required(VERSION 3.26)
project(LuaJIT)

set(LUAJIT_STATIC ${CMAKE_CURRENT_BINARY_DIR}/luajit/lib/libluajit.a)
set(LUAJIT_SHARED ${CMAKE_CURRENT_BINARY_DIR}/luajit/lib/libluajit.so)
set(LUAJIT_LIB
    ${CMAKE_CURRENT_LIST_DIR}/src/libluajit.a
    ${CMAKE_CURRENT_LIST_DIR}/src/libluajit.so
)
if(WIN32)
    set(LUAJIT_STATIC ${CMAKE_CURRENT_LIST_DIR}/luajit/lib/lua51.lib)
    set(LUAJIT_SHARED ${CMAKE_CURRENT_LIST_DIR}/luajit/lib/lua51.dll)
    set(LUAJIT_LIB
        ${CMAKE_CURRENT_LIST_DIR}/src/lua51.lib
        ${CMAKE_CURRENT_LIST_DIR}/src/lua51.dll
    )
endif()

set(LUAJIT_HEADERS
    lua.h
    lualib.h
    lauxlib.h
    luaconf.h
    lua.hpp
    luajit.h
)

list(TRANSFORM LUAJIT_HEADERS PREPEND ${CMAKE_CURRENT_LIST_DIR}/src/ OUTPUT_VARIABLE LUAJIT_INCLUDE_SRC)
list(TRANSFORM LUAJIT_HEADERS PREPEND ${CMAKE_CURRENT_BINARY_DIR}/luajit/include/luajit-2.1/ OUTPUT_VARIABLE LUAJIT_PUBLIC_HEADERS)

if(MSVC)
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND ${CMAKE_CURRENT_LIST_DIR}/src/msvcbuild.bat amalg "" "${LSTG_EXTERNAL_COMPILE_OPTS_WIN}"
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/luajit/include/luajit-2.1
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/luajit/lib
        COMMAND ${CMAKE_COMMAND} -E copy ${LUAJIT_INCLUDE_SRC} ${CMAKE_CURRENT_BINARY_DIR}/luajit/include/luajit-2.1/
        COMMAND ${CMAKE_COMMAND} -E copy ${LUAJIT_LIB} ${CMAKE_CURRENT_BINARY_DIR}/luajit/lib/
        COMMAND_EXPAND_LISTS
        VERBATIM
    )
elseif(APPLE)
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND export MACOSX_DEPLOYMENT_TARGET=10.15
        COMMAND make amalg $<$<BOOL:${CMAKE_CROSSCOMPILING}>:TARGET_FLAGS=--target=${CMAKE_C_COMPILER_TARGET}> TARGET_CFLAGS=${LSTG_EXTERNAL_COMPILE_OPTS}
        COMMAND make install DESTDIR=${CMAKE_CURRENT_BINARY_DIR} PREFIX=/luajit/
        VERBATIM
    )
else()
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND make amalg
            "$<$<BOOL:${CMAKE_CROSSCOMPILING}>:TARGET_FLAGS=--target=${CMAKE_C_COMPILER_TARGET};CROSS=${CMAKE_C_COMPILER_TARGET}-;STATIC_CC=clang;DYNAMIC_CC=clang -fPIC;TARGET_LD=clang>"
            TARGET_CFLAGS=${LSTG_EXTERNAL_COMPILE_OPTS}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
        COMMAND make install DESTDIR=${CMAKE_CURRENT_BINARY_DIR} PREFIX=/luajit/
        COMMAND_EXPAND_LISTS
        VERBATIM
    )
endif()

set(LUAJIT_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/luajit/include/)

add_library(luajit SHARED IMPORTED GLOBAL)
set_target_properties(luajit PROPERTIES
    IMPORTED_LOCATION ${LUAJIT_SHARED}
    PUBLIC_HEADER "${LUAJIT_PUBLIC_HEADERS}"
)
target_include_directories(luajit INTERFACE $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/luajit>)
add_dependencies(luajit build_luajit)

add_library(luajit-static STATIC IMPORTED GLOBAL)
set_target_properties(luajit-static PROPERTIES
    IMPORTED_LOCATION ${LUAJIT_STATIC}
    PUBLIC_HEADER "${LUAJIT_PUBLIC_HEADERS}"
)
target_include_directories(luajit-static INTERFACE $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/luajit>)
add_dependencies(luajit-static build_luajit)

include(GNUInstallDirs)

install(
    FILES ${LUAJIT_PUBLIC_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/luajit
)
install(
    FILES ${LUAJIT_STATIC} ${LUAJIT_SHARED} DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
