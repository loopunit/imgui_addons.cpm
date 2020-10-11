cmake_minimum_required( VERSION 3.12 )
project(imgui_addons
	VERSION 1.7.8)

include(CMakeDependentOption)

option(IMGUI_BUILD_EXAMPLES "Build examples." OFF)
cmake_dependent_option(IMGUI_BUILD_APP_FW "Build app framework." OFF "IMGUI_BUILD_EXAMPLES" ON)
option(IMGUI_BUILD_APP_WIN32_DX11 "Win32 DX11" ON)
option(IMGUI_BUILD_APP_WIN32_DX12 "Win32 DX12" ON)
option(IMGUI_BUILD_APP_GLFW_VULKAN "GLFW Vulkan" OFF)

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info

include(cmake/CPM.cmake)

####

CPMAddPackage(
  NAME PackageProject.cmake
  GITHUB_REPOSITORY TheLartians/PackageProject.cmake
  VERSION 1.3)

####

CPMAddPackage(
	NAME imgui_cpm
	GITHUB_REPOSITORY loopunit/imgui.cpm
	GIT_TAG 0de0f6a7f784c0dbeb61dec57ea5f097a39f6998
	OPTIONS
		"IMGUI_BUILD_EXAMPLES ${IMGUI_BUILD_EXAMPLES}"
		"IMGUI_BUILD_APP_FW ${IMGUI_BUILD_APP_FW}"
		"IMGUI_BUILD_APP_WIN32_DX11 ${IMGUI_BUILD_APP_WIN32_DX11}"
		"IMGUI_BUILD_APP_WIN32_DX12 ${IMGUI_BUILD_APP_WIN32_DX12}"
		"IMGUI_BUILD_APP_GLFW_VULKAN ${IMGUI_BUILD_APP_GLFW_VULKAN}")

if( NOT ${imgui_cpm_ADDED} )
	message( FATAL_ERROR "Could not find imgui.cpm" )
endif()

#############

CPMAddPackage(
	NAME implot
	GITHUB_REPOSITORY epezent/implot
	GIT_TAG 90cbb561f06be0634228a4030efae5fe850562ca
	DOWNLOAD_ONLY true)

if (NOT implot_ADDED)
	message( FATAL_ERROR "Could not find implot" )
endif()

file(GLOB implot_public_headers 
	"${implot_SOURCE_DIR}/implot.h"
	"${implot_SOURCE_DIR}/implot_internal.h"
)

file(GLOB implot_source
	"${implot_SOURCE_DIR}/*.cpp"
)

#############

CPMAddPackage(
	NAME ImGuiColorTextEdit
	GITHUB_REPOSITORY BalazsJako/ImGuiColorTextEdit
	GIT_TAG 0a88824f7de8d0bd11d8419066caa7d3469395c4
	DOWNLOAD_ONLY true)

if (NOT ImGuiColorTextEdit_ADDED)
	message( FATAL_ERROR "Could not find ImGuiColorTextEdit" )
endif()

file(GLOB ImGuiColorTextEdit_public_headers 
	"${ImGuiColorTextEdit_SOURCE_DIR}/TextEditor.h"
)

file(GLOB ImGuiColorTextEdit_source
	"${ImGuiColorTextEdit_SOURCE_DIR}/*.cpp"
)

#############

CPMAddPackage(
	NAME imgui_markdown
	GITHUB_REPOSITORY juliettef/imgui_markdown
	GIT_TAG 9cd1d889a7940a1339044ef18889b5c4e29aba5d
	DOWNLOAD_ONLY true)

if (NOT imgui_markdown_ADDED)
	message( FATAL_ERROR "Could not find imgui_markdown" )
endif()

file(GLOB imgui_markdown_public_headers 
	"${imgui_markdown_SOURCE_DIR}/imgui_markdown.h"
)

#############

CPMAddPackage(
	NAME imnodes
	GITHUB_REPOSITORY Nelarius/imnodes
	GIT_TAG 8ecdd397860e083659347dae310e07dc6f9797d5
	DOWNLOAD_ONLY true)

if (NOT imnodes_ADDED)
	message( FATAL_ERROR "Could not find imnodes" )
endif()

file(GLOB imnodes_public_headers 
	"${imnodes_SOURCE_DIR}/imnodes.h"
)

file(GLOB imnodes_source
	"${imnodes_SOURCE_DIR}/*.cpp"
)

#############

CPMAddPackage(
	NAME imgui_console
	GITHUB_REPOSITORY rmxbalanque/imgui-console
	GIT_TAG a9a1f83b016c3b455f2b88361aa2b79f25f3f961
	DOWNLOAD_ONLY true)
	

if (NOT imgui_console_ADDED)
	message( FATAL_ERROR "Could not find imgui_console" )
endif()

file(GLOB imgui_console_public_headers 
	"${imgui_console_SOURCE_DIR}/include/imgui_console/imgui_console.h"
)

file(GLOB imgui_console_public_headers_csys
	"${imgui_console_SOURCE_DIR}/include/csys/*.*"
)

file(GLOB imgui_console_source
	"${imgui_console_SOURCE_DIR}/src/imgui_console.cpp"
)

#############

CPMAddPackage(
	NAME ImGuiFileDialog
	GITHUB_REPOSITORY aiekick/ImGuiFileDialog
	GIT_TAG 27cf103c3be5fe11084a07c902c4c180d4e0fff6
	DOWNLOAD_ONLY true)

if (NOT ImGuiFileDialog_ADDED)
	message( FATAL_ERROR "Could not find ImGuiFileDialog" )
endif()

file(GLOB ImGuiFileDialog_public_headers 
	"${ImGuiFileDialog_SOURCE_DIR}/ImGuiFileDialog/ImGuiFileDialog.h"
	"${ImGuiFileDialog_SOURCE_DIR}/ImGuiFileDialog/ImGuiFileDialogConfig.h"
	"${ImGuiFileDialog_SOURCE_DIR}/3rdparty/dirent/include/dirent.h")

file(GLOB ImGuiFileDialog_source
	"${ImGuiFileDialog_SOURCE_DIR}/ImGuiFileDialog/*.cpp")

#############

set(STAGING_ROOT ${CMAKE_CURRENT_BINARY_DIR}/imgui_addons_staging)

file(COPY 
	${implot_public_headers}
	${ImGuiColorTextEdit_public_headers}
	${imgui_markdown_public_headers}
	${imnodes_public_headers}
	${ImGuiFileDialog_public_headers} 
	${imgui_console_public_headers}
	DESTINATION ${STAGING_ROOT}/include)

file(COPY 
	${ImTerm_public_headers}
	DESTINATION ${STAGING_ROOT}/include/ImTerm)

file(COPY 
	${imgui_console_public_headers_csys}
	DESTINATION ${STAGING_ROOT}/include/csys)


file(COPY 
	${implot_source}
	${ImGuiColorTextEdit_source}
	${imnodes_source}
	${ImGuiFileDialog_source}
	${imgui_console_source}
	DESTINATION ${STAGING_ROOT}/src)

file(GLOB sources ${STAGING_ROOT}/src/*.*)

add_library(imgui_addons STATIC ${sources})

set_target_properties(imgui_addons PROPERTIES CXX_STANDARD 20)

target_link_libraries(imgui_addons PRIVATE imgui)

target_include_directories(imgui_addons PUBLIC $<BUILD_INTERFACE:${STAGING_ROOT}/include>
                 $<INSTALL_INTERFACE:imgui_addons-${PROJECT_VERSION}>)

target_compile_definitions(imgui_addons PUBLIC SPDLOG_COMPILED_LIB SPDLOG_FMT_EXTERNAL)

packageProject(
  # the name of the target to export
  NAME imgui_addons
  # the version of the target to export
  VERSION ${PROJECT_VERSION}
  # a temporary directory to create the config files
  BINARY_DIR ${PROJECT_BINARY_DIR}
  # location of the target's public headers
  INCLUDE_DIR ${STAGING_ROOT}/include
  # should match the target's INSTALL_INTERFACE include directory
  INCLUDE_DESTINATION include/imgui_addons-${PROJECT_VERSION}
  # semicolon separated list of the project's dependencies
  DEPENDENCIES ""
)

################

if(IMGUI_BUILD_ADDON_EXAMPLES)
	file(GLOB example_sources 
		${CMAKE_CURRENT_LIST_DIR}/examples/main.cpp)

    add_executable(imgui_addons_example
        ${example_sources})
    
	set_target_properties(imgui_addons_example PROPERTIES CXX_STANDARD 20)

    target_include_directories(imgui_addons_example
        PRIVATE 
            ${imgui_SOURCE_DIR}/examples)
    
    target_link_libraries(imgui_addons_example
        PUBLIC
            imgui_app_fw imgui_addons)
endif()