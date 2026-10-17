set(_IRR_DYNAMIC_ false)

if(_IRR_DYNAMIC_)
  set(LIBTYPE SHARED)
else(_IRR_DYNAMIC_)
  set(LIBTYPE STATIC)
endif(_IRR_DYNAMIC_)

# Prefer target-based include handling; don't use a global include_directories here.
# We still add the source dir as a target include below after the target exists.

file(GLOB_RECURSE CVulkan_Source RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp *.c *.h *.hpp *.vert *.frag)
file(GLOB_RECURSE RESOURCE_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.vert *.frag)

LIST(APPEND CVulkan_Source ${RESOURCE_GENERATED_FILES})

# Preserve existing CreateLibrary macro usage (assumes it defines a target named CVulkan)
CreateLibrary("CVulkan" "${CVulkan_Source}" "${LIBTYPE}" "" "")

# --- Find Vulkan loader (vulkan-1) and provide target Vulkan::Vulkan (standard CMake module or package config) ---
# Prefer CMake's find_package for Vulkan which typically provides Vulkan::Vulkan
find_package(Vulkan REQUIRED)

# --- Find vulkan-hpp (vulkan/vulkan.hpp) in vcpkg, local C:/vulkan-hpp, or VK_SDK_PATH ---
# Try common vcpkg locations and the manual folder; create an INTERFACE target VulkanHeaders when found.
if (NOT TARGET VulkanHeaders)
  find_path(VULKAN_HPP_INCLUDE_DIR
    NAMES vulkan/vulkan.hpp
    PATHS
      "$ENV{VCPKG_ROOT}/installed/x64-windows/include"
      "$ENV{VCPKG_ROOT}/installed/x86-windows/include"
      "$ENV{VCPKG_ROOT}/installed/x64-windows/include/vulkan"
      "$ENV{VCPKG_ROOT}/installed/x86-windows/include/vulkan"
      "C:/vulkan-hpp"
      "$ENV{VK_SDK_PATH}/Include"
      ${CMAKE_INSTALL_PREFIX}/include
      /usr/include
      /usr/local/include
  )

  if (VULKAN_HPP_INCLUDE_DIR)
    add_library(VulkanHeaders INTERFACE)
    target_include_directories(VulkanHeaders INTERFACE "${VULKAN_HPP_INCLUDE_DIR}")
    message(STATUS "Found vulkan-hpp headers: ${VULKAN_HPP_INCLUDE_DIR} -> created VulkanHeaders INTERFACE target")
  else()
    message(WARNING "vulkan.hpp (vulkan-hpp) not found. To compile, install vulkan-hpp (e.g., via vcpkg) or place headers in C:/vulkan-hpp or set VK_SDK_PATH.")
  endif()
endif()

# Ensure CVulkan depends on ZLIB and Glslang before linking
add_dependencies(CVulkan ZLIB Glslang)

# Add source-dir include for this module (private to the target)
target_include_directories(CVulkan PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# Link core libraries. Vulkan::Vulkan is required (find_package(Vulkan REQUIRED) above).
# Link VulkanHeaders only if the INTERFACE target was created.
target_link_libraries(CVulkan
  PRIVATE
    Vulkan::Vulkan
    ZLIB
    Glslang
    glm
    CFramework
)

if (TARGET VulkanHeaders)
  target_link_libraries(CVulkan PRIVATE VulkanHeaders)
endif()

# Resource build section (unchanged)
resource_build(
    TARGET "CVulkan"
    GENERATED_DIR "${CMAKE_BINARY_DIR}"
    SOURCE_ROOT "${CMAKE_CURRENT_LIST_DIR}"
    SOURCE_FILE "${CMAKE_CURRENT_BINARY_DIR}/.Resource/Resource_CVulkan.dsc"
    MODULE_SOURCE_FILE CVulkan
    RESOURCE_FILES ${RESOURCE_SRCS}
    GENERATED_FILES ${RESOURCE_GENERATED_FILES}
    RESOURCE_EXECUTABLE ${ResourceCompilerExecutable}
)

SET_PROPERTY(GLOBAL APPEND PROPERTY ResxDataFiles "CVulkan.resx")

# Dynamic vs static linking differences preserved
if(_IRR_DYNAMIC_)
    target_link_libraries (CVulkan
        irrlicht
    )
    install(TARGETS CVulkan DESTINATION ${CMAKE_INSTALL_PREFIX})
    if (WIN32)
        target_link_libraries (CVulkan winmm)
    endif()
else()
    add_definitions(-D_LIB_SOURCE_ROOT="${CMAKE_CURRENT_SOURCE_DIR}")
    add_definitions(-D_IRR_COMPILE_WITH_VULKAN_)
    add_definitions(-DIRRLICHT_EXPORTS)

    # The project previously used LINK_INTERFACE_LIBRARIES; keep compatibility
    target_link_libraries (CVulkan LINK_INTERFACE_LIBRARIES
        irrlicht
    )
endif(_IRR_DYNAMIC_)

set_target_properties(CVulkan
    PROPERTIES
      FOLDER
        "RenderEngine/IrrLicht")
