#Copyright © 2023 A-OS-Project https://github.com/A-OS-Project/
#
#Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#
#The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
#THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
# File created on 8th January 2023 by Pyogenics under A-OS-Project
# Last modified on 19th January 2023
#

cmake_minimum_required(VERSION 2.8.12)
project(img4tool LANGUAGES CXX C)

# Find required libraries
find_package(OpenSSL 0.9.8 REQUIRED)
# These aren't cmake modules (yet)
# XXX: I'm unsure about this .so stuff, shouldn't this be OS independent?
find_library(LIBGENERAL libgeneral libgeneral.so
	PATHS ${ADDITIONAL_SEARCH_PATHS}
	REQUIRED
)
find_library(LIBPLIST libplist libplist-2.0.so
	PATHS ${ADDITIONAL_SEARCH_PATHS}
	REQUIRED
)
find_library(LIBLZFSE liblzfse liblzfse.a	# Static library?
	PATHS ${ADDITIONAL_SEARCH_PATHS}
	REQUIRED
)

# Source files, XXX: Don't put headers here! If they are in existing folder they should be picked up, otherwise add the new folder to INCLUDE_DIRS.
set(SOURCES
	img4tool/main.cpp
	img4tool/img4tool.cpp
	img4tool/lzssdec.c
	img4tool/ASN1DERElement.cpp
)

# Include directories
set(INCLUDE_DIRS
	img4tool/
	include/
)

add_executable(${CMAKE_PROJECT_NAME} ${SOURCES})
install(TARGETS ${CMAKE_PROJECT_NAME} DESTINATION bin) # FIXME: Seems to be broken? It can't find libgeneral.so.0 when installed but works fine in the build folder.

# Definitions
execute_process(
  COMMAND git rev-list --count HEAD
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE VERSION_COMMIT_COUNT
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
  COMMAND git rev-parse HEAD
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE VERSION_COMMIT_SHA
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
add_compile_definitions(HAVE_LIBLZFSE HAVE_OPENSSL HAVE_CRYPTO HAVE_PLIST PACKAGE_NAME="${CMAKE_PROJECT_NAME}" VERSION_COMMIT_COUNT="${VERSION_COMMIT_COUNT}" VERSION_COMMIT_SHA="${VERSION_COMMIT_SHA}") # TODO: Make optional dependencies work

target_include_directories(${CMAKE_PROJECT_NAME}
	PRIVATE ${INCLUDE_DIRS}
)
target_link_libraries(${CMAKE_PROJECT_NAME}
	PRIVATE OpenSSL::Crypto
	PRIVATE OpenSSL::SSL
	PRIVATE ${LIBGENERAL}
	PRIVATE ${LIBPLIST}
	PRIVATE ${LIBLZFSE}
)
