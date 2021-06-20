cmake_minimum_required(VERSION 3.9)

project(Dump1090 VERSION 1.10.1706.21)

find_package(PkgConfig)

# dump1090
set(HAVE_HACKRF_SUPPORT OFF CACHE BOOL "Enables support of HackRF One device")
set(HAVE_RTLSDR_SUPPORT OFF CACHE BOOL "Enables support of RTL-SDR device")

add_executable(dump1090 anet.h anet.c interactive.c mode_ac.c mode_s.c net_io.c winstubs.h dump1090.h dump1090.c)
set_target_properties(dump1090 PROPERTIES  C_STANDARD_REQUIRED 11)

target_include_directories(dump1090 PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# Set platform specific compiler options
if(WIN32)
    set(HAVE_STRUCT_TIMESPEC ON CACHE BOOL "Specifies if 'timespec' structure is defined in standard headers")

    if(${HAVE_STRUCT_TIMESPEC})
        target_compile_definitions(dump1090 PRIVATE HAVE_STRUCT_TIMESPEC)
    endif()
endif()

add_subdirectory(dependencies)

# Find pthread library
if(WIN32)
    add_dependencies(dump1090 pthread)

    add_custom_command(TARGET dump1090
        POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:pthread> $<TARGET_FILE_DIR:dump1090>)
    
    set(PTHREADS_INCLUDEDIR_DEFAULT $<TARGET_PROPERTY:pthread,INTERFACE_INCLUDE_DIRECTORIES>)
    set(PTHREADS_LIBRARIES_DEFAULT $<TARGET_LINKER_FILE:pthread>)
else()
    set(THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads)
    
    set(PTHREADS_INCLUDEDIR_DEFAULT ${THREADS_PTHREADS_INCLUDE_DIR})
    set(PTHREADS_LIBRARIES_DEFAULT ${CMAKE_THREAD_LIBS_INIT})
endif()

set(PTHREADS_INCLUDEDIR ${PTHREADS_INCLUDEDIR_DEFAULT} CACHE PATH "Path to header files for pthread library")
set(PTHREADS_LIBRARIES ${PTHREADS_LIBRARIES_DEFAULT} CACHE FILEPATH "Path to .lib file for pthread library")

target_include_directories(dump1090 PRIVATE ${PTHREADS_INCLUDEDIR})
target_link_libraries(dump1090 PRIVATE ${PTHREADS_LIBRARIES})

# HacRF support
if(${HAVE_HACKRF_SUPPORT})
    if(PkgConfig_FOUND)
        pkg_check_modules(LIBHACKRF libhackrf)
    elseif(WIN32)
        add_dependencies(dump1090 libhackrf)

        add_custom_command(TARGET dump1090
            POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:libhackrf> $<TARGET_FILE_DIR:dump1090>)
        add_custom_command(TARGET dump1090
            POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:libusb> $<TARGET_FILE_DIR:dump1090>)
        add_custom_command(TARGET dump1090
            POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:pthread> $<TARGET_FILE_DIR:dump1090>)
        
        set(LIBHACKRF_INCLUDEDIR_DEFAULT $<TARGET_PROPERTY:libhackrf,INTERFACE_INCLUDE_DIRECTORIES>)
        set(LIBHACKRF_LIBRARIES_DEFAULT $<TARGET_LINKER_FILE:libhackrf>)
    endif()
    
    set(LIBHACKRF_INCLUDEDIR ${LIBHACKRF_INCLUDEDIR_DEFAULT} CACHE PATH "Path to header files for libhackrf library")
    set(LIBHACKRF_LIBRARIES ${LIBHACKRF_LIBRARIES_DEFAULT} CACHE FILEPATH "Path to .lib file for libhackrf library")
    
    target_compile_definitions(dump1090 PRIVATE HAVE_HACKRF_SUPPORT)
    target_include_directories(dump1090 PRIVATE ${LIBHACKRF_INCLUDEDIR})
    target_link_libraries(dump1090 PRIVATE ${LIBHACKRF_LIBRARIES})
endif()

# RTL-SDR support
if(${HAVE_RTLSDR_SUPPORT})
    if(PkgConfig_FOUND)
        pkg_check_modules(LIBRTLSDR librtlsdr)
    elseif(WIN32)
        add_dependencies(dump1090 librtlsdr)

        add_custom_command(TARGET dump1090
            POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:librtlsdr> $<TARGET_FILE_DIR:dump1090>)
        add_custom_command(TARGET dump1090
            POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:libusb> $<TARGET_FILE_DIR:dump1090>)
        add_custom_command(TARGET dump1090
            POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:pthread> $<TARGET_FILE_DIR:dump1090>)

        set(LIBRTLSDR_INCLUDEDIR_DEFAULT $<TARGET_PROPERTY:librtlsdr,INTERFACE_INCLUDE_DIRECTORIES>)
        set(LIBRTLSDR_LIBRARIES_DEFAULT $<TARGET_LINKER_FILE:librtlsdr>)
    endif()
    
    set(LIBRTLSDR_INCLUDEDIR ${LIBRTLSDR_INCLUDEDIR_DEFAULT} CACHE PATH "Path to header files for librtlsdr library")
    set(LIBRTLSDR_LIBRARIES ${LIBRTLSDR_LIBRARIES_DEFAULT} CACHE FILEPATH "Path to .lib file for librtlsdr library")
    
    target_compile_definitions(dump1090 PRIVATE HAVE_RTLSDR_SUPPORT)
    target_include_directories(dump1090 PRIVATE ${LIBRTLSDR_INCLUDEDIR})
    target_link_libraries(dump1090 PRIVATE ${LIBRTLSDR_LIBRARIES})
endif()

# Set platform specific linker options
if(WIN32)
    target_link_libraries(dump1090 PRIVATE ws2_32)
else()
    target_link_libraries(dump1090 PRIVATE m)
endif()

# view1090
add_executable(view1090 anet.h anet.c interactive.c mode_ac.c mode_s.c net_io.c winstubs.h dump1090.h view1090.h view1090.c)
target_include_directories(view1090 PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

if(WIN32)
    if(${HAVE_STRUCT_TIMESPEC})
        target_compile_definitions(view1090 PRIVATE HAVE_STRUCT_TIMESPEC)
    endif()
endif()

target_include_directories(view1090 PRIVATE ${PTHREADS_INCLUDEDIR})
target_link_libraries(view1090 PRIVATE ${PTHREADS_LIBRARIES})

if(WIN32)
    target_link_libraries(view1090 PRIVATE ws2_32)
else()
    target_link_libraries(view1090 PRIVATE m)
endif()
