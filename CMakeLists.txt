#
# xUnit++ test framework
#
cmake_minimum_required(VERSION 3.0)
project(xUnitpp)

# Helpers
# -------------------------------------------------------------------------------------------------
function(globrecurse_c_cpp_sources result folder)
	file(GLOB_RECURSE res
		"${folder}/*.c"
		"${folder}/*.cpp"
		"${folder}/*.h"
		"${folder}/*.hpp")
	set(${result} ${res} PARENT_SCOPE)
endfunction()

function(xunit_compiler_options target)
	# Compiler-specific options
	if(WIN32)
		# Fast compilation
		target_compile_options(${target} PRIVATE "/MP")

		# No warning about not secure CRT functions
		target_compile_definitions(${target} PRIVATE _CRT_SECURE_NO_WARNINGS)
	endif()

	if(MINGW OR CMAKE_COMPILER_IS_GNUCXX OR ${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
		# Enable C++11 if gcc or clang
		target_compile_options(${target} PUBLIC "-std=c++11")

		# Debug infos in debug
		target_compile_options(${target} PRIVATE $<$<CONFIG:Debug>:"-g">)

		# Optimizations
		target_compile_options(${target} PRIVATE $<$<NOT:$<CONFIG:Debug>>:"-O2">)
		
		# Remove __stdcall
		target_compile_definitions(${target} PRIVATE "__stdcall=")
	endif()
endfunction()

function(setup_target_paths Target LibDir)
	set_target_properties(${Target} PROPERTIES
		OUTPUT_NAME ${Target}
		DEBUG_POSTFIX _d
		RELWITHDEBINFO_POSTFIX _rd
		MINSIZEREL_POSTFIX _rm
		RUNTIME_OUTPUT_DIRECTORY ${LibDir}
		LIBRARY_OUTPUT_DIRECTORY ${LibDir}
		ARCHIVE_OUTPUT_DIRECTORY ${LibDir}
		PDB_OUTPUT_DIRECTORY ${LibDir}
		RUNTIME_OUTPUT_DIRECTORY_DEBUG ${LibDir}
		LIBRARY_OUTPUT_DIRECTORY_DEBUG ${LibDir}
		ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${LibDir}
		PDB_OUTPUT_DIRECTORY_DEBUG ${LibDir}
		RUNTIME_OUTPUT_DIRECTORY_RELEASE ${LibDir}
		LIBRARY_OUTPUT_DIRECTORY_RELEASE ${LibDir}
		ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${LibDir}
		PDB_OUTPUT_DIRECTORY_RELEASE ${LibDir}
		RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LibDir}
		LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LibDir}
		ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LibDir}
		PDB_OUTPUT_DIRECTORY_RELWITHDEBINFO ${LibDir}
		RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${LibDir}
		LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL ${LibDir}
		ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL ${LibDir}
		PDB_OUTPUT_DIRECTORY_MINSIZEREL ${LibDir}
	)
endfunction()

# Folders
# -------------------------------------------------------------------------------------------------
set(XUNITPP_ROOT_DIR "${CMAKE_CURRENT_LIST_DIR}")

# Core target
# -------------------------------------------------------------------------------------------------
# Sources
globrecurse_c_cpp_sources(XUNITPP_SRC_FILES_CORE ${XUNITPP_ROOT_DIR}/xUnit++)
source_group(Source FILES ${XUNITPP_SRC_FILES_CORE})

# Shared library
add_library(xunitpp_core SHARED ${XUNITPP_SRC_FILES_CORE})
target_include_directories(xunitpp_core
	PUBLIC ${XUNITPP_ROOT_DIR}/xUnit++
	PRIVATE ${XUNITPP_ROOT_DIR}/xUnit++/xUnit++)
if(NOT MSVC)
	target_link_libraries(xunitpp_core pthread)
endif()
xunit_compiler_options(xunitpp_core)
setup_target_paths(xunitpp_core ${XUNITPP_ROOT_DIR}/bin)

# Utility target
# -------------------------------------------------------------------------------------------------
# Sources
globrecurse_c_cpp_sources(XUNITPP_SRC_FILES_UTIL ${XUNITPP_ROOT_DIR}/xUnit++.Utility)
source_group(Source FILES ${XUNITPP_SRC_FILES_UTIL})

# Library
add_library(xunitpp_util ${XUNITPP_SRC_FILES_UTIL})
target_include_directories(xunitpp_util PUBLIC ${XUNITPP_ROOT_DIR}/xUnit++.Utility)
target_link_libraries(xunitpp_util xunitpp_core)
xunit_compiler_options(xunitpp_util)
setup_target_paths(xunitpp_util ${XUNITPP_ROOT_DIR}/bin)

# Console target
# -------------------------------------------------------------------------------------------------
# Sources
globrecurse_c_cpp_sources(XUNITPP_SRC_FILES_CONSOLE ${XUNITPP_ROOT_DIR}/xUnit++.console)
source_group(Source FILES ${XUNITPP_SRC_FILES_CONSOLE})

# Executable
add_executable(xunitpp_console ${XUNITPP_SRC_FILES_CONSOLE})
target_include_directories(xunitpp_console PRIVATE ${XUNITPP_ROOT_DIR}/xUnit++.console)
target_link_libraries(xunitpp_console xunitpp_core xunitpp_util)
if(UNIX)
	target_link_libraries(xunitpp_console dl)
endif()
xunit_compiler_options(xunitpp_console)
setup_target_paths(xunitpp_console ${XUNITPP_ROOT_DIR}/bin)

