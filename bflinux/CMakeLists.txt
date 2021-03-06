#
# Copyright (C) 2019 Assured Information Security, Inc.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.13)
project(bflinux C CXX)

init_project(bflinux INTERFACE)

target_link_libraries(bflinux INTERFACE userspace::bfintrinsics)

add_executable(init init.cpp)
target_link_options(init PRIVATE -static)
target_link_libraries(init PRIVATE bflinux)
add_custom_command(
    TARGET init POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PREFIXES_DIR}/initrd/
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PREFIXES_DIR}/initrd/dev
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PREFIXES_DIR}/initrd/sbin
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PREFIXES_DIR}/initrd/proc
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PREFIXES_DIR}/initrd/etc
    COMMAND ${CMAKE_COMMAND} -E copy /etc/localtime ${PREFIXES_DIR}/initrd/etc/localtime
    COMMAND ${CMAKE_COMMAND} -E copy init ${PREFIXES_DIR}/initrd/sbin/
    COMMAND ${CMAKE_COMMAND} -E chdir ${PREFIXES_DIR}/initrd/ bash ${BOXY_SOURCE_ROOT_DIR}/bflinux/build.sh
    VERBATIM
)

install(TARGETS init DESTINATION bin EXPORT bflinux-userspace-targets)

fini_project()
