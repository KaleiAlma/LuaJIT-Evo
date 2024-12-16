# i can't be assed to write and/or maintain a proper cmake build script for luajit.

cmake_minimum_required(VERSION 3.26)
project(LuaJIT)

set(LUAJIT_STATIC ${CMAKE_CURRENT_LIST_DIR}/src/libluajit.a)
set(LUAJIT_SHARED ${CMAKE_CURRENT_LIST_DIR}/src/libluajit.so)
if(WIN32)
    set(LUAJIT_STATIC ${CMAKE_CURRENT_LIST_DIR}/src/lua51.lib)
    set(LUAJIT_SHARED ${CMAKE_CURRENT_LIST_DIR}/src/lua51.dll)
endif()

if(MSVC)
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND ${CMAKE_CURRENT_LIST_DIR}/src/msvcbuild.bat static "" "${LSTG_EXTERNAL_COMPILE_OPTS_WIN}"
        VERBATIM
    )
elseif(APPLE)
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND export MACOSX_DEPLOYMENT_TARGET=10.15
        COMMAND make $<$<BOOL:${CMAKE_CROSSCOMPILING}>:TARGET_FLAGS=--target=${CMAKE_C_COMPILER_TARGET}> TARGET_CFLAGS=${LSTG_EXTERNAL_COMPILE_OPTS}
        VERBATIM
    )
else()
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND make "$<$<BOOL:${CMAKE_CROSSCOMPILING}>:'TARGET_FLAGS=--target=${CMAKE_C_COMPILER_TARGET}';'CROSS=${CMAKE_C_COMPILER_TARGET}-';STATIC_CC=clang;'DYNAMIC_CC=clang -fPIC'>" TARGET_CFLAGS=${LSTG_EXTERNAL_COMPILE_OPTS}
        COMMAND_EXPAND_LISTS
        VERBATIM
    )
endif()

add_library(luajit SHARED IMPORTED GLOBAL)
set_target_properties(luajit PROPERTIES
    IMPORTED_LOCATION ${LUAJIT_SHARED}
)
target_include_directories(luajit INTERFACE ${CMAKE_CURRENT_LIST_DIR}/src/)
add_dependencies(luajit build_luajit)

add_library(luajit-static STATIC IMPORTED GLOBAL)
set_target_properties(luajit-static PROPERTIES
    IMPORTED_LOCATION ${LUAJIT_STATIC}
)
target_include_directories(luajit-static INTERFACE ${CMAKE_CURRENT_LIST_DIR}/src/)
add_dependencies(luajit-static build_luajit)

