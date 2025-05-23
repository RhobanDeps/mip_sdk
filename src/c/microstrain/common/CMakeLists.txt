set(MICROSTRAIN_COMMON_LIBRARY "microstrain_common" CACHE INTERNAL "Name of the common MicroStrain library")
mark_as_advanced(MICROSTRAIN_COMMON_LIBRARY)

#option(MICROSTRAIN_ENABLE_LOGGING "Build with logging functions enabled" ON)
set(MICROSTRAIN_LOGGING_MAX_LEVEL "MICROSTRAIN_LOG_LEVEL_WARN" CACHE STRING "Max log level the SDK is allowed to log. If this is defined, any call to logging functions with a higher level than this will be compiled out.")
set(MICROSTRAIN_TIMESTAMP_TYPE "uint64_t" CACHE STRING "Override the type used for received data timestamps and timeouts (must be unsigned or at least 64 bits).")

add_library(${MICROSTRAIN_COMMON_LIBRARY}
    "${CMAKE_CURRENT_LIST_DIR}/logging.h"
    "${CMAKE_CURRENT_LIST_DIR}/platform.h"
    "${CMAKE_CURRENT_LIST_DIR}/serialization.c"
    "${CMAKE_CURRENT_LIST_DIR}/serialization.h"
)

target_compile_features(${MICROSTRAIN_COMMON_LIBRARY} PUBLIC c_std_11)

target_compile_definitions(${MICROSTRAIN_COMMON_LIBRARY}
    PUBLIC "MICROSTRAIN_TIMESTAMP_TYPE=${MICROSTRAIN_TIMESTAMP_TYPE}"
    PRIVATE ${MICROSTRAIN_PRIVATE_COMPILE_DEFINITIONS}
)

target_compile_options(${MICROSTRAIN_COMMON_LIBRARY} PRIVATE ${MICROSTRAIN_PRIVATE_COMPILE_OPTIONS})

# Logging is a little weird since we need to install the header no matter what
if(MICROSTRAIN_ENABLE_LOGGING)
    if(MICROSTRAIN_LOGGING_MAX_LEVEL STREQUAL "")
        message(FATAL_ERROR "MICROSTRAIN_LOGGING_MAX_LEVEL must be MICROSTRAIN_LOG_LEVEL_* or a number")
    endif()
    target_sources(${MICROSTRAIN_COMMON_LIBRARY} PRIVATE "${CMAKE_CURRENT_LIST_DIR}/logging.c")
    message("MicroStrain logging is enabled. Max level = '${MICROSTRAIN_LOGGING_MAX_LEVEL}'")
    target_compile_definitions(${MICROSTRAIN_COMMON_LIBRARY} PUBLIC "MICROSTRAIN_ENABLE_LOGGING" "MICROSTRAIN_LOGGING_MAX_LEVEL=${MICROSTRAIN_LOGGING_MAX_LEVEL}")
endif()

target_include_directories(${MICROSTRAIN_COMMON_LIBRARY} INTERFACE ${MICROSTRAIN_SRC_C_DIR})

set(MICROSTRAIN_LIBRARIES_TMP ${MICROSTRAIN_LIBRARIES_TMP} ${MICROSTRAIN_COMMON_LIBRARY} PARENT_SCOPE)

#
# Installation
#

include(microstrain_utilities)
microstrain_setup_library_install(${MICROSTRAIN_COMMON_LIBRARY} ${MICROSTRAIN_SRC_DIR})
