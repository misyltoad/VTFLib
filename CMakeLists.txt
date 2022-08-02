cmake_minimum_required(VERSION 3.10)

cmake_policy(SET CMP0091 NEW)

project(vtflib C CXX)

option(VTFLIB_STATIC "If enabled, builds VTFLIB as a static library" ON)
option(VTFLIB_SHARED "If enabled, builds VTFLIB as a shared library" ON)

include_directories(thirdparty/include thirdparty/zlib)

if (UNIX)
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--no-undefined")
endif()

# MT/MTd specification for Windows
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

#
# VTFLIB
#
set(VTFLIB_SRCS
	VTFLib/Resource.rc
	
	VTFLib/Error.cpp
	VTFLib/FileReader.cpp
	VTFLib/FileWriter.cpp
	VTFLib/Float16.cpp
	VTFLib/MemoryReader.cpp
	VTFLib/MemoryWriter.cpp
	VTFLib/Proc.cpp
	VTFLib/ProcReader.cpp
	VTFLib/ProcWriter.cpp
	VTFLib/VMTFile.cpp
	VTFLib/VMTGroupNode.cpp
	VTFLib/VMTIntegerNode.cpp
	VTFLib/VMTNode.cpp
	VTFLib/VMTSingleNode.cpp
	VTFLib/VMTStringNode.cpp
	VTFLib/VMTValueNode.cpp
	VTFLib/VMTWrapper.cpp
	VTFLib/VTFFile.cpp
	VTFLib/VTFLib.cpp
	VTFLib/VTFMathlib.cpp
	VTFLib/VTFWrapper.cpp)

if (VTFLIB_STATIC)
	add_library(vtflib_static STATIC ${VTFLIB_SRCS})
	target_compile_definitions(vtflib_static PUBLIC VTFLIB_STATIC)
endif()
if (VTFLIB_SHARED)
	add_library(vtflib SHARED ${VTFLIB_SRCS})
endif()

add_subdirectory(thirdparty/zlib)

# Link against compressonator libs & set runtime lib
if (UNIX)
	if (VTFLIB_SHARED)
		target_link_directories(vtflib PUBLIC thirdparty/lib/x64)
		target_link_libraries(vtflib PUBLIC CMP_Compressonator pthread)
	endif()
	if (VTFLIB_STATIC)
		target_link_directories(vtflib_static PUBLIC thirdparty/lib/x64)
		target_link_libraries(vtflib_static PUBLIC CMP_Compressonator pthread)
	endif()
else()
	if (VTFLIB_SHARED)
		target_link_directories(vtflib PUBLIC thirdparty/lib)
		target_link_libraries(vtflib PUBLIC "Compressonator_MT$<$<CONFIG:Debug>:d>")
	endif()
	if (VTFLIB_STATIC)
		target_link_libraries(vtflib_static PUBLIC "Compressonator_MT$<$<CONFIG:Debug>:d>")
		target_link_directories(vtflib_static PUBLIC thirdparty/lib)
	endif()
endif()

if (VTFLIB_SHARED)
	target_compile_definitions(vtflib PRIVATE VTFLIB_EXPORTS)
	target_precompile_headers(vtflib PRIVATE VTFLib/stdafx.h)
	target_include_directories(vtflib PUBLIC lib)
	target_link_libraries(vtflib PRIVATE zlibstatic)
endif()
if (VTFLIB_STATIC)
	target_compile_definitions(vtflib_static PRIVATE VTFLIB_EXPORTS)
	target_precompile_headers(vtflib_static PRIVATE VTFLib/stdafx.h)
	target_include_directories(vtflib_static PUBLIC lib)
	target_link_libraries(vtflib_static PRIVATE zlibstatic)
endif()
