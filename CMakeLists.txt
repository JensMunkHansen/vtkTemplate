option(STANDALONE "Stand alone" ON)

if (STANDALONE)
  cmake_minimum_required(VERSION 3.8...3.12 FATAL_ERROR)
  
  # Extra policies (not covering all introduced after 3.12)
  foreach(policy
      CMP0083 # CMake 3.14
      CMP0127
      # Add more here
      )
    if(POLICY ${policy})
      cmake_policy(SET ${policy} NEW)
    endif()
  endforeach()
  
  include(CMakeDependentOption)
  
  project(Template
    LANGUAGES
    CXX)
  
  
  find_package(VTK 9.3 REQUIRED)
  
  option(BUILD_SHARED_LIBS "Build shared libraries." ON)
  
  include(GNUInstallDirs)
  
  option(TEMPLATE_RELOCATABLE_INSTALL "Do not embed hard-coded paths into the install" ON)
  mark_as_advanced(TEMPLATE_RELOCATABLE_INSTALL)

  option(BUILD_TESTING "Build the tests" ON)
  
  set(TEMPLATE_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})
  set(CMAKE_CXX_VISIBILITY_PRESET "hidden")
  set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)
  set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
  
  # Set up our directory structure for output libraries and binaries
  # (Note: these are the build locations, not the install locations)
  if(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${Template_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
  endif()
  
  if(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
    if(UNIX)
      set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${Template_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
    else()
      set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${Template_BINARY_DIR}${CMAKE_INSTALL_BINDIR}")
    endif()
  endif()
  
  if(NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${Template_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
  endif()
  
  set(template_requested_modules
    "VTK::Template"
  )
  
  option(Template_BUILD_ALL_MODULES "Build all modules by default" OFF)
  mark_as_advanced(Template_BUILD_ALL_MODULES)
  
  set(template_source_directories
    "${CMAKE_CURRENT_SOURCE_DIR}/Template"
  )
  
  vtk_module_find_modules(template_module_files ${template_source_directories})
  
  vtk_module_find_kits(template_kit_files ${template_source_directories})
  
  vtk_module_scan(
    MODULE_FILES        ${template_module_files}
    KIT_FILES           ${template_kit_files} # empty right now
    REQUEST_MODULES     ${template_requested_modules}
    REJECT_MODULES      ${template_rejected_modules}
    PROVIDES_MODULES    template_modules
    PROVIDES_KITS       template_kits
    WANT_BY_DEFAULT     "${Template_BUILD_ALL_MODULES}"
    ENABLE_TESTS        "${BUILD_TESTING}"
  )
  
  vtk_module_build(
    MODULES               ${template_modules}
    KITS                  ${template_kits}
    INSTALL_EXPORT        "${template_install_export}"
    ARCHIVE_DESTINATION   "lib"
    HEADERS_DESTINATION   "${CMAKE_INSTALL_INCLUDEDIR}/template${template_version_suffix}"
    CMAKE_DESTINATION     "${template_cmake_destination}"
    LICENSE_DESTINATION   "${CMAKE_INSTALL_LIBDIR}/template"
    HIERARCHY_DESTINATION "${CMAKE_INSTALL_LIBDIR}/vtk/hierarchy/${CMAKE_PROJECT_NAME}"
    LIBRARY_NAME_SUFFIX   "${Template_CUSTOM_LIBRARY_SUFFIX}"
    VERSION               "${Template_VERSION}"
    SOVERSION             "1" # If the library is small and should work as a remote module to VTK
    TEST_DATA_TARGET      TemplateData
    INSTALL_HEADERS       "${Template_INSTALL_SDK}"
    USE_EXTERNAL          "${Template_USE_EXTERNAL}"
    TEST_INPUT_DATA_DIRECTORY   "${CMAKE_CURRENT_SOURCE_DIR}/Testing"
    TEST_OUTPUT_DATA_DIRECTORY  "${CMAKE_CURRENT_BINARY_DIR}/ExternalData/Testing")
else()
  add_subdirectory(Template)
endif()
