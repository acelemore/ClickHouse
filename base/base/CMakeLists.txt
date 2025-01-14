set (SRCS
    argsToConfig.cpp
    coverage.cpp
    demangle.cpp
    getFQDNOrHostName.cpp
    getMemoryAmount.cpp
    getPageSize.cpp
    getThreadId.cpp
    JSON.cpp
    LineReader.cpp
    mremap.cpp
    phdr_cache.cpp
    preciseExp10.cpp
    setTerminalEcho.cpp
    shift10.cpp
    sleep.cpp
    terminalColors.cpp
    errnoToString.cpp
    StringRef.cpp
)

if (ENABLE_REPLXX)
    list (APPEND SRCS ReplxxLineReader.cpp)
endif ()

if (USE_DEBUG_HELPERS)
    get_target_property(MAGIC_ENUM_INCLUDE_DIR magic_enum INTERFACE_INCLUDE_DIRECTORIES)
    # CMake generator expression will do insane quoting when it encounters special character like quotes, spaces, etc.
    # Prefixing "SHELL:" will force it to use the original text.
    set (INCLUDE_DEBUG_HELPERS "SHELL:-I\"${MAGIC_ENUM_INCLUDE_DIR}\" -include \"${ClickHouse_SOURCE_DIR}/base/base/iostream_debug_helpers.h\"")
    # Use generator expression as we don't want to pollute CMAKE_CXX_FLAGS, which will interfere with CMake check system.
    add_compile_options($<$<COMPILE_LANGUAGE:CXX>:${INCLUDE_DEBUG_HELPERS}>)
endif ()

add_library (common ${SRCS})

if (WITH_COVERAGE)
    target_compile_definitions(common PUBLIC WITH_COVERAGE=1)
else ()
    target_compile_definitions(common PUBLIC WITH_COVERAGE=0)
endif ()

if (USE_INTERNAL_CCTZ)
    set_source_files_properties(DateLUTImpl.cpp PROPERTIES COMPILE_DEFINITIONS USE_INTERNAL_CCTZ)
endif()

target_include_directories(common PUBLIC .. "${CMAKE_CURRENT_BINARY_DIR}/..")

if (OS_DARWIN AND NOT MAKE_STATIC_LIBRARIES)
    target_link_libraries(common PUBLIC -Wl,-U,_inside_main)
endif()

target_link_libraries (common
    PUBLIC
        ${CITYHASH_LIBRARIES}
        boost::headers_only
        boost::system
        Poco::Net
        Poco::Net::SSL
        Poco::Util
        Poco::Foundation
        replxx
        cctz
        fmt
        magic_enum
)

if (ENABLE_TESTS)
    add_subdirectory (tests)
endif ()
