############################################################################
#  lib/CMakeLists.txt
#
#  CMake file for libstxxl.a/so/lib
#
#  Part of the STXXL. See http://stxxl.sourceforge.net
#
#  Copyright (C) 2013 Timo Bingmann <tb@panthema.net>
#
#  Distributed under the Boost Software License, Version 1.0.
#  (See accompanying file LICENSE_1_0.txt or copy at
#  http://www.boost.org/LICENSE_1_0.txt)
############################################################################

set(LIBSTXXL_SOURCES

  common/cmdline.cpp
  common/exithandler.cpp
  common/log.cpp
  common/rand.cpp
  common/seed.cpp
  common/utils.cpp
  common/verbose.cpp
  common/version.cpp

  io/boostfd_file.cpp
  io/create_file.cpp
  io/disk_queued_file.cpp
  io/file.cpp
  io/fileperblock_file.cpp
  io/iostats.cpp
  io/mem_file.cpp
  io/request.cpp
  io/request_queue_impl_1q.cpp
  io/request_queue_impl_qwqr.cpp
  io/request_queue_impl_worker.cpp
  io/request_with_state.cpp
  io/request_with_waiters.cpp
  io/serving_request.cpp
  io/syscall_file.cpp
  io/ufs_file_base.cpp
  io/wbtl_file.cpp
  io/wfs_file_base.cpp
  io/wincall_file.cpp

  mng/block_manager.cpp
  mng/config.cpp
  mng/disk_allocator.cpp

  algo/async_schedule.cpp

  )

if(NOT MSVC)
  # additional sources for non Visual Studio builds
  set(LIBSTXXL_SOURCES ${LIBSTXXL_SOURCES}
    io/mmap_file.cpp
    io/simdisk_file.cpp
    )
endif(NOT MSVC)

if(STXXL_HAVE_LINUXAIO_FILE)
  # additional sources fo LinuxAIO fileio access method
  set(LIBSTXXL_SOURCES ${LIBSTXXL_SOURCES}
    io/linuxaio_file.cpp
    io/linuxaio_queue.cpp
    io/linuxaio_request.cpp
    )
endif()

if(USE_MALLOC_COUNT)
  # enable light-weight heap profiling tool malloc_count
  set(LIBSTXXL_SOURCES ${LIBSTXXL_SOURCES}
    utils/malloc_count.cpp
    )

  set_source_files_properties(utils/malloc_count.cpp
    PROPERTIES COMPILE_FLAGS "-Wno-pedantic")

endif(USE_MALLOC_COUNT)

# tell top-level cmakelists which library we build
set(STXXL_EXPORTED_LIBS stxxl)

# we name debug library builds "stxxl_debug" and release builds "stxxl"
string(TOLOWER "stxxl_${CMAKE_BUILD_TYPE}" STXXL_LIBNAME)
if(STXXL_LIBNAME STREQUAL "stxxl_release" OR STXXL_LIBNAME STREQUAL "stxxl_")
  set(STXXL_LIBNAME "stxxl")
endif()

if(NOT BUILD_STATIC_LIBS)
  message(FATAL_ERROR "You must build either a static or shared STXXL library")
endif()

# build stxxl STATIC library and link all programs again it
add_library(stxxl STATIC ${LIBSTXXL_SOURCES})
if (MSVC)
  target_compile_definitions(stxxl PUBLIC NOMINMAX WIN32_LEAN_AND_MEAN)
endif()
set_target_properties(stxxl PROPERTIES OUTPUT_NAME "${STXXL_LIBNAME}")
set_target_properties(stxxl PROPERTIES VERSION "${STXXL_VERSION_STRING}")
target_include_directories(stxxl SYSTEM PUBLIC ${STXXL_INCLUDE_DIRS})
target_compile_options(stxxl PRIVATE -Wno-deprecated-declarations)
# install(TARGETS stxxl
#   EXPORT stxxl-targets
#   ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
#   LIBRARY DESTINATION ${INSTALL_LIB_DIR})

# export variables to top-level cmakelists
set(STXXL_LIBNAME ${STXXL_LIBNAME} PARENT_SCOPE)
set(STXXL_EXPORTED_LIBS ${STXXL_EXPORTED_LIBS} PARENT_SCOPE)
