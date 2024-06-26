#=============================================================================
#   CMake build system files
#
#   Copyright (c) 2021-22 chipStar developers
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"), to deal
#   in the Software without restriction, including without limitation the rights
#   to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#   copies of the Software, and to permit persons to whom the Software is
#   furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included in
#   all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#   AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#   OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#   THE SOFTWARE.
#
#=============================================================================

# For telling cuspvc where to look for clang.
get_filename_component(CLANG_BIN_DIR ${CMAKE_CXX_COMPILER} DIRECTORY)

# Read includes, prepend each one with -I.
set(prop "$<TARGET_PROPERTY:CHIP,INCLUDE_DIRECTORIES>")
set(CHIP_INCLUDES "$<$<BOOL:${prop}>:-I$<JOIN:${prop}, -I>>")

# Generate cuspvc tool that is usable within build directory.
# TODO: Deprecate cuspvc
configure_file(cuspvc-build.in cuspvc-build.in @ONLY)
file(
  GENERATE
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/cuspvc
  INPUT ${CMAKE_CURRENT_BINARY_DIR}/cuspvc-build.in)

# Generate cuspv tool that will be installed.
# TODO: Deprecate cuspvc
configure_file(cuspvc-install.in cuspvc-install @ONLY)
install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/cuspvc-install
  DESTINATION bin RENAME cuspvc)

# Generate cucc tool that is usable within build directory.
set(CUCC_HIP_PATH ${CMAKE_BINARY_DIR})
configure_file(cucc.in cucc @ONLY)

# This cucc variant is for the install destination.
set(CUCC_HIP_PATH ${CMAKE_INSTALL_PREFIX})
configure_file(cucc.in cucc-install @ONLY)
install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/cucc-install
  DESTINATION bin RENAME cucc)

# Run static type check on cucc if there is a tool available for it.
find_program(MYPY NAMES mypy)
if(MYPY)
  add_test(NAME cucc-static-type-check
    COMMAND ${MYPY} ${CMAKE_CURRENT_SOURCE_DIR}/cucc.in)
endif()
