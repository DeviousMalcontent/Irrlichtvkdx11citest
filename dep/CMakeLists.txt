if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/GLM/CMakeLists.txt")
    AddBuildInLibrary(GLM glm)
elseif(NOT TARGET glm AND NOT TARGET glm::glm)
    find_package(glm REQUIRED)
endif()

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/JPEGLIB/CMakeLists.txt")
    AddBuildInLibrary(JPEGLIB jpeglib)
elseif(NOT TARGET JPEG::JPEG)
    find_package(JPEG REQUIRED)
endif()

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/LIBPNG/CMakeLists.txt")
    AddBuildInLibrary(LIBPNG libpng)
elseif(NOT TARGET PNG::PNG)
    find_package(PNG REQUIRED)
endif()

add_subdirectory(threads)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/ZLIB/CMakeLists.txt")
    AddBuildInLibrary(ZLIB zlib)
elseif(NOT TARGET ZLIB::ZLIB)
    find_package(ZLIB REQUIRED)
endif()

IF (BUILD_IN_USE_GLSLANG)
    # Set Glslang settings and connections
    SET(SPIRV-Headers_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/SPIRV/SPIRV-Headers)
    add_subdirectory(SPIRV/SPIRV-Tools)
    add_subdirectory(SPIRV/Glslang)
ENDIF(BUILD_IN_USE_GLSLANG)
