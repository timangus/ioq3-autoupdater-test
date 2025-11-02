cmake_minimum_required(VERSION 3.10)
project(autoupdater C)

if(WIN32)
    message(FATAL_ERROR "Windows support left as an exercise for the reader")
endif()

set(CMAKE_C_STANDARD 99)

include(ExternalProject)

set(RSA_TOOLS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/code/autoupdater/rsa_tools)
set(TFMVER "0.13.1")
set(LTCVER "1.17")
set(TFM_DIR "${RSA_TOOLS_DIR}/tomsfastmath-${TFMVER}")
set(LTC_DIR "${RSA_TOOLS_DIR}/libtomcrypt-${LTCVER}")

ExternalProject_Add(libtom_build
    SOURCE_DIR ${RSA_TOOLS_DIR}
    CONFIGURE_COMMAND ""
    BUILD_COMMAND bash ${RSA_TOOLS_DIR}/build-libtom-unix.sh
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND ""
    BUILD_BYPRODUCTS
        ${TFM_DIR}/libtfm.a
        ${LTC_DIR}/libtomcrypt.a
)

add_library(tommath STATIC IMPORTED)
set_target_properties(tommath PROPERTIES
    IMPORTED_LOCATION ${TFM_DIR}/libtfm.a
)

add_library(tomcrypt STATIC IMPORTED)
set_target_properties(tomcrypt PROPERTIES
    IMPORTED_LOCATION ${LTC_DIR}/libtomcrypt.a
)

set(TOMMATH_INCLUDE_DIR ${TFM_DIR}/src/headers)
set(TOMCRYPT_INCLUDE_DIR ${LTC_DIR}/src/headers)

add_executable(autoupdater code/autoupdater/autoupdater.c)

add_dependencies(autoupdater libtom_build)

target_include_directories(autoupdater PRIVATE
    ${TOMCRYPT_INCLUDE_DIR}
    ${TOMMATH_INCLUDE_DIR}
)

target_link_libraries(autoupdater
    dl
    tomcrypt
    tommath
)

target_include_directories(autoupdater PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/code/autoupdater/rsa_tools
)

add_executable(rsa_make_keys
    code/autoupdater/rsa_tools/rsa_make_keys.c
    code/autoupdater/rsa_tools/rsa_common.c
)

add_executable(rsa_sign
    code/autoupdater/rsa_tools/rsa_sign.c
    code/autoupdater/rsa_tools/rsa_common.c
)

add_executable(rsa_verify
    code/autoupdater/rsa_tools/rsa_verify.c
    code/autoupdater/rsa_tools/rsa_common.c
)

add_dependencies(rsa_make_keys libtom_build)
add_dependencies(rsa_sign libtom_build)
add_dependencies(rsa_verify libtom_build)

target_include_directories(rsa_make_keys PRIVATE
    ${TOMCRYPT_INCLUDE_DIR}
    ${TOMMATH_INCLUDE_DIR}
)
target_include_directories(rsa_sign PRIVATE
    ${TOMCRYPT_INCLUDE_DIR}
    ${TOMMATH_INCLUDE_DIR}
)
target_include_directories(rsa_verify PRIVATE
    ${TOMCRYPT_INCLUDE_DIR}
    ${TOMMATH_INCLUDE_DIR}
)

target_link_libraries(rsa_make_keys tomcrypt tommath)
target_link_libraries(rsa_sign tomcrypt tommath)
target_link_libraries(rsa_verify tomcrypt tommath)
