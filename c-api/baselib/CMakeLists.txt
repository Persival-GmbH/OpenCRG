cmake_minimum_required(VERSION 3.25)

project(OpenCRG VERSION 2.0.1 LANGUAGES C)

if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build Type")
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release RelWithDebInfo MinSizeRel)
endif()

# Embed debug information when using MSVC according to build type
if (MSVC)
    set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<$<CONFIG:Debug,RelWithDebInfo>:Embedded>")
endif()

# Source files
set(OPENCRG_SOURCES
    src/crgContactPoint.c
    src/crgEvalpk.c
    src/crgEvaluv2xy.c
    src/crgEvalxy2uv.c
    src/crgEvalz.c
    src/crgLoader.c
    src/crgMgr.c
    src/crgMsg.c
    src/crgOptionMgmt.c
    src/crgPortability.c
    src/crgStatistics.c
)

# Create static library
add_library(${PROJECT_NAME} STATIC ${OPENCRG_SOURCES})

# Include header directory
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/inc)

# Set C11 Standard for target
set_target_properties(${PROJECT_NAME} PROPERTIES
    C_STANDARD 11
    C_STANDARD_REQUIRED ON
    C_EXTENSIONS OFF
)

# Set warning compiler flags
target_compile_options(${PROJECT_NAME} PRIVATE
    $<$<C_COMPILER_ID:MSVC>:/W4>
    $<$<NOT:$<C_COMPILER_ID:MSVC>>:-Wall -Wextra>
)

# Link to libm if unix-like
if(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE m)
endif()

# Install library to libdir
include(GNUInstallDirs)

install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/${CMAKE_INSTALL_LIBDIR}
	CONFIGURATIONS Debug Release RelWithDebInfo MinSizeRel
)

install(CODE "
    file(INSTALL
        DESTINATION \"${CMAKE_CURRENT_SOURCE_DIR}/${CMAKE_INSTALL_LIBDIR}\"
        TYPE FILE
        FILES \"\$<TARGET_FILE:${PROJECT_NAME}>\"
        RENAME \"\$<TARGET_FILE_PREFIX:${PROJECT_NAME}>\$<TARGET_FILE_BASE_NAME:${PROJECT_NAME}>.${PROJECT_VERSION}\$<TARGET_FILE_SUFFIX:${PROJECT_NAME}>\"
    )"
)