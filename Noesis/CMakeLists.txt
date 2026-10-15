SET(MODUL_LINKAGE "SHARED")
SET(MODUL_LINK_LIBS "")
SET(MODUL_LINK_FLAGS "")

option(IRR_BUILD_NOESIS "Enable Noesis support" ON)

# Paths for optional vendored Noesis dependency
set(NOESIS_DIR "${CMAKE_SOURCE_DIR}/dep/NoesisGui")
set(NOESIS_INCLUDE_DIR "${NOESIS_DIR}/include")
set(NOESIS_LIB_DIR "${NOESIS_DIR}/lib")

# If Noesis is enabled, check that headers exist and try to make an imported target if needed.
if(IRR_BUILD_NOESIS)
  if(EXISTS "${NOESIS_INCLUDE_DIR}")
    message(STATUS "Noesis: found headers at ${NOESIS_INCLUDE_DIR}")
    # If an imported target 'noesisgui' doesn't already exist, try to create one using a library from dep/NoesisGui/lib
    if(NOT TARGET noesisgui)
      file(GLOB NOESIS_LIBS RELATIVE "${NOESIS_LIB_DIR}" "${NOESIS_LIB_DIR}/*noesis*.*")
      if(NOESIS_LIBS)
        list(GET NOESIS_LIBS 0 NOESIS_LIB_FIRST)
        add_library(noesisgui SHARED IMPORTED)
        set_target_properties(noesisgui PROPERTIES IMPORTED_LOCATION "${NOESIS_LIB_DIR}/${NOESIS_LIB_FIRST}")
        target_include_directories(noesisgui INTERFACE "${NOESIS_INCLUDE_DIR}")
        message(STATUS "Noesis: created imported target 'noesisgui' -> ${NOESIS_LIB_DIR}/${NOESIS_LIB_FIRST}")
      else()
        message(STATUS "Noesis: no library file found in ${NOESIS_LIB_DIR}; assuming external target may be provided elsewhere")
      endif()
    endif()
  else()
    message(WARNING "NoesisGui headers not found at ${NOESIS_INCLUDE_DIR}; disabling Noesis support")
    set(IRR_BUILD_NOESIS OFF CACHE BOOL "Disable Noesis support" FORCE)
  endif()
endif()

file(GLOB_RECURSE LibCommon_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp *.h *.c *.cc)

if (BUILD_HAS_RESOURCE_SYSTEM)
	group_source(${LibCommon_SRCS})
endif (BUILD_HAS_RESOURCE_SYSTEM)

CreateLibrary("NoesisApp" "${LibCommon_SRCS}" "${MODUL_LINKAGE}" "${MODUL_LINK_LIBS}" "${MODUL_LINK_FLAGS}")

# Always link the internal dependencies; only link noesisgui if Noesis is enabled and target exists.
target_link_libraries(NoesisApp
PRIVATE
   CFramework
   irrlicht
)

if (IRR_BUILD_NOESIS AND TARGET noesisgui)
  target_link_libraries(NoesisApp PUBLIC noesisgui)
endif()

target_include_directories(NoesisApp
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    PRIVATE
		${CMAKE_CURRENT_SOURCE_DIR}
        ${PROTOBUF_INCLUDE_DIRS}
        ${CMAKE_CURRENT_BINARY_DIR}/.Assembly
)

target_compile_definitions(NoesisApp
    PUBLIC  NS_IRR_NOESISAPP_API
    PRIVATE TC_API_EXPORT_NOESISAPP)

set_target_properties(NoesisApp
    PROPERTIES
      FOLDER
        "Contrib")
