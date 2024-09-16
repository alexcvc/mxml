# Minimum CMake version
cmake_minimum_required(VERSION 3.10)

# Project name and version
project(libmxml VERSION 4.0.4 LANGUAGES C)

set(MXML_VERSION ${PROJECT_VERSION})

# name library
set(TargetName mxml)
set(MXML_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR} CACHE PATH "mxml include directories")

# Specify the C standard
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

set(MXML_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(MXML_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(MXML_VERSION_MICRO ${PROJECT_VERSION_PATCH})

# Source files for the library
set(SOURCES
   mxml-attr.c
   mxml-file.c
   mxml-get.c
   mxml-index.c
   mxml-node.c
   mxml-options.c
   mxml-private.c
   mxml-search.c
   mxml-set.c
)

# Header files to install
set(HEADERS
   mxml-private.h
   config.h
)

# Create the library
add_library(${TargetName} STATIC ${SOURCES} ${HEADERS})
add_library(${TargetName}::${TargetName} ALIAS ${TargetName})

# Add version info to the library
set_target_properties(${TargetName} PROPERTIES
   VERSION ${PROJECT_VERSION}
   SOVERSION 1
)

target_include_directories(${TargetName} PUBLIC
   $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
   $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Enable secure build settings
target_compile_definitions(${TargetName} PRIVATE -D_FORTIFY_SOURCE=2 -D_GNU_SOURCE)
target_compile_options(${TargetName} PRIVATE -Wall -Werror -fstack-protector-strong -Wformat -Wformat-security -O2)
target_link_options(${TargetName} PRIVATE -Wl,-z,relro -Wl,-z,now)

# Remove -Werror=unused-parameter for this specific target
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_compile_options(${TargetName} PRIVATE -Wno-unused-parameter)
endif ()

# If you are using Clang, you might need similar condition:
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  target_compile_options(${TargetName} PRIVATE -Wno-unused-parameter)
endif ()

# Check if the compiler supports the -Wimplicit-fallthrough option
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-Wimplicit-fallthrough" HAS_IMPLICIT_FALLTHROUGH)

if (HAS_IMPLICIT_FALLTHROUGH)
  # Remove -Werror=implicit-fallthrough= if it exists
  target_compile_options(${TargetName} PRIVATE
     $<$<COMPILE_LANGUAGE:CXX>:$<$<COMPILE_FLAG:-Werror=implicit-fallthrough=>: -Wno-error=implicit-fallthrough=>>)
endif ()

# Link libraries based on the OS
if (WIN32)
  target_link_libraries(${TargetName} wsock32)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "QNX")
  target_link_libraries(${TargetName} socket)
endif ()

# Install library and headers
install(TARGETS ${TargetName}
   LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
   ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(FILES ${HEADERS}
   DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/mxml
)


