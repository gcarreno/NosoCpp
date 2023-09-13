# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)

# Set the project name
project(
  NosoCpp
  VERSION 0.2
  DESCRIPTION "NosoC++ is a C++ cryptographic library that provides various cryptographic functions to work with the Noso coin."
  HOMEPAGE_URL "https://github.com/pasichDev/NosoCpp"
)

set(SOURCE_FILES
    src/nCore.cpp
    src/nCripto.cpp
    src/NosoCpp.cpp
    src/nUtils.cpp
    src/WalletObject.cpp
)

set(INSTALL_HEADER_FILES
  include/Constants.h
  include/nCore.h
  include/nCripto.h
  include/nUtils.h
  include/Objects.h
  include/WalletObject.h
)

# Add the library source SHARED
if (NOSOCPP_MAKE_SHARED)
  add_library(NosoCpp SHARED
    ${SOURCE_FILES}
  )

  include (GenerateExportHeader)

  # generates the export header shared_EXPORTS.h automatically
  GENERATE_EXPORT_HEADER(NosoCpp
      BASE_NAME shared
      EXPORT_MACRO_NAME shared_EXPORTS
      EXPORT_FILE_NAME shared_EXPORTS.h
      STATIC_DEFINE SHARED_EXPORTS_BUILT_AS_STATIC
  )

  install(FILES
    ${CMAKE_BINARY_DIR}/shared_EXPORTS.h
    DESTINATION include
  )

  set(
    CPACK_PACKAGE_NAME "libnosocpp"
  )
endif()

# Add the library source STATIC
if (NOSOCPP_MAKE_STATIC)
  add_library(NosoCpp STATIC
    ${SOURCE_FILES}
  )
  set(
    CPACK_PACKAGE_NAME "libnosocpp-dev"
  )
endif()

# Set the lib version, public API header and Windows Export symbols
set_target_properties(NosoCpp PROPERTIES
  VERSION ${PROJECT_VERSION}
  PUBLIC_HEADER include/NosoCpp.h
  OUTPUT_NAME nosocpp
  CXX_STANDARD 20
)

## PKG Config export
configure_file(NosoCpp.pc.in nosocpp.pc @ONLY)

# Set source and include folders
target_include_directories(NosoCpp PRIVATE
  include
  src
)

# Add the dependency packets
## Find required packages
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

## Botan
find_package(Botan REQUIRED)
target_link_libraries(NosoCpp PRIVATE Botan::Botan)

# OS and CPack configuration
## Linux
if(LINUX OR APPLE)

message(STATUS
  "Configuring CPack for Linux or Apple"
)

# Install rules from GNU
include(GNUInstallDirs)


set_target_properties(NosoCpp PROPERTIES
  SOVERSION 1
)

install(TARGETS NosoCpp
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(FILES
    ${CMAKE_BINARY_DIR}/nosocpp.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
endif()

## Windows
if(WINDOWS)

message(STATUS
  "Configuring CPack for Windows"
)

install(TARGETS NosoCpp
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(FILES
    ${CMAKE_BINARY_DIR}/nosocpp.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

endif()

install(FILES
  ${INSTALL_HEADER_FILES}
  DESTINATION include
)

install(FILES
  README.md
  LICENSE.txt
  TYPE DOC
)

include(CPack)
