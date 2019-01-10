#
# Copyright (C) 2016 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Build one shared lib. When we build, we set the bin to $user/tmp, and
# remove it after build complete, so only the distribution place could be
# used for application linking.

cmake_minimum_required(VERSION 3.4.1)

set(CMAKE_VERBOSE_MAKEFILE on)

add_library(acrosync SHARED rsync/rsync_client.cpp
        rsync/rsync_entry.cpp
        rsync/rsync_file.cpp
        rsync/rsync_io.cpp
        rsync/rsync_log.cpp
        rsync/rsync_pathutil.cpp
        rsync/rsync_socketutil.cpp
        rsync/rsync_sshio.cpp
        rsync/rsync_stream.cpp
        rsync/rsync_timeutil.cpp
        rsync/rsync_util.cpp)

set(dependencies_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../dependencies)
set(distribution_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../../../../distribution)

include_directories(${CMAKE_CURRENT_SOURCE_DIR} "${dependencies_DIR}/openssl/include" "${dependencies_DIR}/libssh2/include")
link_directories("${dependencies_DIR}/openssl/libs/${ANDROID_ABI}" "${dependencies_DIR}/libssh2/libs/${ANDROID_ABI}")
#link_directories("${dependencies_DIR}/libssh2/libs/${ANDROID_ABI}")

add_library(lib_libssh2 STATIC IMPORTED)
set_target_properties(lib_libssh2 PROPERTIES IMPORTED_LOCATION
        ${dependencies_DIR}/libssh2/libs/${ANDROID_ABI}/libssh2.a)

add_library(lib_openssl STATIC IMPORTED)
set_target_properties(lib_openssl PROPERTIES IMPORTED_LOCATION
        ${dependencies_DIR}/openssl/libs/${ANDROID_ABI}/libcrypto.a)

target_link_libraries(acrosync
        z
        lib_libssh2
        lib_openssl)

# copy out the lib binary and remove generated files
set_target_properties(acrosync
                      PROPERTIES
                      LIBRARY_OUTPUT_DIRECTORY
                      "${distribution_DIR}/acrosync/lib/${ANDROID_ABI}")

add_custom_command(TARGET acrosync POST_BUILD
                   COMMAND "${CMAKE_COMMAND}" -E
                   copy "${CMAKE_CURRENT_SOURCE_DIR}/rsync/rsync_client.h"
                   "${distribution_DIR}/acrosync/include/rsync_client.h"
        #                   **** the following 2 lines are for potential future debug purpose ****
        #                   COMMAND "${CMAKE_COMMAND}" -E
        #                   remove_directory "${CMAKE_CURRENT_BINARY_DIR}"
                   COMMENT "Copying rsync to output directory")

