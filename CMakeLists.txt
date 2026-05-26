cmake_minimum_required(VERSION 3.0...3.10)
project(libldac
        LANGUAGES C
        VERSION 2.0.3.1)

set(CMAKE_C_STANDARD 99)


option(LDAC_SOFT_FLOAT "For device without FPU, turn on this option" OFF)
if(${LDAC_SOFT_FLOAT} STREQUAL ON)
    set(LDAC_SOFT_FLOAT_DEFINE "-D_32BIT_FIXED_POINT")
endif()

if("${INSTALL_LIBDIR}" STREQUAL "")
    set(INSTALL_LIBDIR ${CMAKE_INSTALL_PREFIX}/lib)
endif()

if("${INSTALL_INCLUDEDIR}" STREQUAL "")
    set(INSTALL_INCLUDEDIR ${CMAKE_INSTALL_PREFIX}/include)
endif()

if("${INSTALL_PKGCONFIGDIR}" STREQUAL "")
    set(INSTALL_PKGCONFIGDIR ${INSTALL_LIBDIR}/pkgconfig)
endif()

if("${INSTALL_LDAC_INCLUDEDIR}" STREQUAL "")
    set(INSTALL_LDAC_INCLUDEDIR ${INSTALL_INCLUDEDIR}/ldac)
endif()

option(APPLY_PATCHES "For tarballs releases just ignore it" FALSE)
if("${APPLY_PATCHES}" STREQUAL TRUE)
	set(LDACBT_APPLY_PATCHES "TRUE")
endif()

option(LDACBT_LEGACY_SYMLINKS "Generate Symlinks for legacy ldacBT_enc.so" OFF)
if("${LDACBT_LEGACY}" STREQUAL ON)
	set(LDACBT_SYMLINKS "TRUE")
endif()

option(LDAC_ENCODER_ONLY "Build enconder only LDACBT" OFF)
if("${LDACBT_ENCODER_ONLY}" STREQUAL ON)
	set(LDACBT_ENCODER_DEFINE "-D_ENCODER_ONLY")
endif()

add_subdirectory(pkgconfig)

include_directories(libldac/inc)
include_directories(libldac/abr/inc)

if("$LDACBT_APPLY_PATCHES" STREQUAL TRUE)
	execute_process(COMMAND git apply "${CMAKE_SOURCE_DIR}/0001-hack-Try-to-fix-broken-API.patch"
                        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
endif()

# libldacBT
add_library(ldacBT SHARED
        libldac/src/ldaclib.c
        libldac/src/ldacBT.c
        )

target_link_libraries(ldacBT m)
target_compile_definitions(ldacBT PUBLIC ${LDAC_ENCODER_DEFINE} ${LDAC_SOFT_FLOAT_DEFINE})
set_target_properties(ldacBT PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        PUBLIC_HEADER libldac/inc/ldacBT.h
        )

# libldacBT_abr
add_library(ldacBT_abr SHARED
        libldac/abr/src/ldacBT_abr.c
        )

target_link_libraries(ldacBT_abr ldacBT)
set_target_properties(ldacBT_abr PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        PUBLIC_HEADER libldac/abr/inc/ldacBT_abr.h
        )

if("$LDACBT_SYMLINKS" STREQUAL TRUE)
ADD_CUSTOM_TARGET(ldacBT_enc ALL
                  COMMAND ${CMAKE_COMMAND} -E create_symlink ${ldacBT} ${ldacBT_enc})
set_target_properties(ldacBT PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        PUBLIC_HEADER libldac/inc/ldacBT.h
        )
endif()

if("$LDACBT_SYMLINKS" STREQUAL TRUE and "$LDACBT_ENCODER_ONLY" STREQUAL "OFF")
ADD_CUSTOM_TARGET(ldacBT_dec ALL
                  COMMAND ${CMAKE_COMMAND} -E create_symlink ${target} ${ldacBT_dec})
set_target_properties(ldacBT PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR}
        PUBLIC_HEADER libldac/inc/ldacBT.h
        )
endif()

install(TARGETS
                ldacBT
				ldacBT_abr
        LIBRARY
                DESTINATION ${INSTALL_LIBDIR}
        PUBLIC_HEADER
                DESTINATION ${INSTALL_LDAC_INCLUDEDIR}
	)


install(FILES
	libldacBT_dec.h 
	DESTINATION ${INSTALL_LDAC_INCLUDEDIR}
)
	
