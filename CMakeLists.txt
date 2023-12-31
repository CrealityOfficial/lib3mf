include(GNUInstallDirs)

# Define Version
set(LIB3MF_VERSION_MAJOR 2)        # increase on every backward-compatibility breaking change of the API
set(LIB3MF_VERSION_MINOR 1)        # increase on every backward compatible change of the API
set(LIB3MF_VERSION_MICRO 1)        # increase on on every change that does not alter the API
set(LIB3MF_VERSION_PRERELEASE "")  # denotes pre-release information of a version of lib3mf

set(LIB3MF_NAME "lib3MF")
option(USE_INCLUDED_ZLIB "Use included zlib" ON)
option(USE_INCLUDED_LIBZIP "Use included libzip" ON)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  # using GCC
  add_definitions(-DBUILD_DLL)
  add_compile_options(-Wall)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2")
elseif ("${CMAKE_SYSTEM_NAME}" MATCHES "Darwin")
  # using GCC
  add_definitions(-DBUILD_DLL)
  add_compile_options(-Wall) 
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2")
  set(CMAKE_MACOSX_RPATH ON)
endif()

if (${MSVC})
  # using Visual Studio C++

  # this ensures that the min/max macros of minwindef.h are not used
  add_definitions(-DNOMINMAX)
  
  # add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE -D_CRT_SECURE_NO_WARNINGS)
  # set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
  # set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
endif()

### The API generation target
if(CMAKE_HOST_UNIX)
  if(APPLE)
    set(ACT_COMMANDENDING darwin)
  else()
    set(ACT_COMMANDENDING linux)
  endif()
elseif(CMAKE_HOST_WIN32)
  set(ACT_COMMANDENDING exe)
endif()

set(CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED ${CMAKE_CURRENT_BINARY_DIR}/Autogenerated)
set(CMAKE_TEMP_HEADER_FOLDER "${CMAKE_CURRENT_SOURCE_DIR}/AutomaticComponentToolkit/Lib3MF_component")
#set(ACT_GENERATED_SOURCE
#  ${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Source/Implementation/lib3mf_interfaceexception.cpp
#  ${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Source/Implementation/lib3mf_interfacewrapper.cpp
#  ${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Source/Implementation/lib3mf_interfacejournal.cpp 
#)
#add_custom_command(
#  BYPRODUCTS
#    ${ACT_GENERATED_SOURCE}
#  OUTPUT
#    ${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/LIB3MFACTGENERATIONSTAMP
#  WORKING_DIRECTORY
#    ${CMAKE_CURRENT_SOURCE_DIR}/AutomaticComponentToolkit
#  COMMAND
#    cmake -E make_directory "${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}"
#  COMMAND
#    cmake -E touch "${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/LIB3MFACTGENERATIONSTAMP"
#  COMMAND
#    ${CMAKE_CURRENT_SOURCE_DIR}/AutomaticComponentToolkit/bin/act.${ACT_COMMANDENDING} ${CMAKE_CURRENT_SOURCE_DIR}/AutomaticComponentToolkit/lib3mf.xml
#  COMMAND
#    cmake -E remove_directory "${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Source"
#  COMMAND
#    cmake -E remove_directory "${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Bindings"
#  COMMAND
#    cmake -E copy_directory "${CMAKE_TEMP_HEADER_FOLDER}/Bindings" ${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Bindings
#  COMMAND
#    cmake -E copy_directory "${CMAKE_TEMP_HEADER_FOLDER}/Implementations/Cpp/Interfaces" "${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/Source/Implementation"
#  COMMAND
#    cmake -E remove_directory "${CMAKE_TEMP_HEADER_FOLDER}"
#  COMMENT
#    "Generating hourglass API bindings and implementation stub"
#  DEPENDS
#    ${CMAKE_CURRENT_SOURCE_DIR}/AutomaticComponentToolkit/lib3mf.xml
#  VERBATIM
#)
#
#add_custom_target(lib3mfACT
#  DEPENDS
#    ${CMAKE_CURRENT_BINARY_DIR_AUTOGENERATED}/LIB3MFACTGENERATIONSTAMP
#  SOURCES
#    ${CMAKE_CURRENT_SOURCE_DIR}/AutomaticComponentToolkit/lib3mf.xml
#)

include(Source/CMakeLists.txt)
#########################################################
# Set version info
if(DEFINED ENV{BUILD_NUMBER})
  set(BUILD_NUMBER $ENV{BUILD_NUMBER})
else()
  set(BUILD_NUMBER 0)
endif()

if (WIN32)
  message("BUILD_NUMBER ... " ${BUILD_NUMBER})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/Include/VersionInfo)

  include(generate_product_version)
    generate_product_version(
    VERSION_FILES_OUTPUTLOCATION
    NAME "lib3mf"
    FILE_DESCRIPTION "lib3mf is an implementation of the 3D Manufacturing Format file standard"
    BUNDLE "3MF Consortium"
    VERSION_MAJOR ${LIB3MF_VERSION_MAJOR}
    VERSION_MINOR ${LIB3MF_VERSION_MINOR}
    VERSION_PATCH ${LIB3MF_VERSION_MICRO}
    VERSION_REVISION ${BUILD_NUMBER}
    PRODUCT_VERSION_MAJOR ${LIB3MF_VERSION_MAJOR}
    PRODUCT_VERSION_MINOR ${LIB3MF_VERSION_MINOR}
    PRODUCT_VERSION_PATCH ${LIB3MF_VERSION_MICRO}
    PRODUCT_VERSION_REVISION ${LIB3MF_VERSION_PRERELEASE}
    COMPANY_NAME "3MF Consortium"
  )
  message("VERSION_FILES_OUTPUTLOCATION ... " ${VERSION_FILES_OUTPUTLOCATION})
else()
  set(VERSION_FILES_OUTPUTLOCATION "")
endif()


#########################################################
# Shared library

set(LIBS)
if (USE_INCLUDED_LIBZIP)
    list(APPEND LIBS zip)
endif()
if (USE_INCLUDED_ZLIB)
    list(APPEND LIBS zlib)
endif()

__add_real_target(${LIB3MF_NAME} lib SOURCE ${SRCS_COMMON} ${HDRS_COMMON}
  ${ACT_GENERATED_SOURCE}
  LIB ${LIBS}
  DEF Z_PREFIX
  INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/Bindings/Cpp/
)
SOURCE_GROUP("Source Files\\Autogenerated" FILES ${ACT_GENERATED_SOURCE})

#add_dependencies(${LIB3MF_NAME} lib3mfACT)
target_include_directories(${LIB3MF_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Source/Implementation)
target_include_directories(${LIB3MF_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Include/API)
target_include_directories(${LIB3MF_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Include)

if (USE_INCLUDED_LIBZIP)
    target_link_libraries(${LIB3MF_NAME} PRIVATE zip)
endif()
if (USE_INCLUDED_ZLIB)
    target_link_libraries(${LIB3MF_NAME} PRIVATE zlib)
endif()

# This makes sure symbols are exported
target_compile_options(${LIB3MF_NAME} PRIVATE "-D__LIB3MF_EXPORTS")
# The following two properties are crucial to reduce the number of undesirably exported symbols
set_target_properties(${LIB3MF_NAME} PROPERTIES C_VISIBILITY_PRESET hidden)
set_target_properties(${LIB3MF_NAME} PROPERTIES CXX_VISIBILITY_PRESET hidden)
set_target_properties(${LIB3MF_NAME} PROPERTIES VISIBILITY_INLINES_HIDDEN ON)

set(STARTUPPROJECT ${LIB3MF_NAME})

#########################################################
if (UNIX OR MINGW)
  # Uncomment the following to put the version info into the .so-file.
  set_target_properties(${LIB3MF_NAME} PROPERTIES VERSION "${LIB3MF_VERSION_MAJOR}.${LIB3MF_VERSION_MINOR}.${LIB3MF_VERSION_MICRO}.${BUILD_NUMBER}")
  set_target_properties(${LIB3MF_NAME} PROPERTIES SOVERSION "${LIB3MF_VERSION_MAJOR}")
  if (NOT APPLE)
    set_target_properties(${LIB3MF_NAME} PROPERTIES LINK_FLAGS -s)
  endif()
endif()

if(CC_INSTALL_3MF)
	__install_directory_specif(${CMAKE_CURRENT_SOURCE_DIR}/Bindings/Cpp/ 3mf)
endif()
