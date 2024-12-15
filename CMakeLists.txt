# i can't be assed to write and/or maintain a proper cmake build script for luajit.

cmake_minimum_required(VERSION 3.26)
project(LuaJIT)

set(archdetect_c_code "
#if defined(__aarch64__) || defined(_M_ARM64) 
    #error cmake_ARCH aarch64
#elif defined(__x86_64) || defined(__x86_64__) || defined(__amd64) || defined(_M_X64)
    #error cmake_ARCH x86_64
#endif

#error cmake_ARCH unsupported
")

file(WRITE "${CMAKE_BINARY_DIR}/arch.c" "${archdetect_c_code}")

enable_language(C)

try_run(
    run_result_unused
    compile_result_unused
    "${CMAKE_BINARY_DIR}"
    "${CMAKE_BINARY_DIR}/arch.c"
    COMPILE_OUTPUT_VARIABLE ARCH
)

string(REGEX MATCH "cmake_ARCH ([a-zA-Z0-9_]+)" ARCH "${ARCH}")

string(REPLACE "cmake_ARCH " "" ARCH "${ARCH}")

if (NOT ARCH)
    set(ARCH unsupported)
endif()

set(LUAJIT_TARGET_ARCH "${ARCH}")


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
        COMMAND ${CMAKE_CURRENT_LIST_DIR}/src/msvcbuild.bat
        VERBATIM
    )
elseif(APPLE)
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND export MACOSX_DEPLOYMENT_TARGET=10.15
        COMMAND make $<$<BOOL:${CMAKE_CROSSCOMPILING}>:TARGET_CFLAGS="-arch=${LUAJIT_TARGET_ARCH}">
        VERBATIM
    )
else()
    add_custom_target(build_luajit
        BYPRODUCTS
            ${LUAJIT_SHARED}
            ${LUAJIT_STATIC}
        WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
        COMMAND make $<$<BOOL:${CMAKE_CROSSCOMPILING}>:TARGET_CFLAGS="--target=${CMAKE_C_COMPILER_TARGET}">
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

