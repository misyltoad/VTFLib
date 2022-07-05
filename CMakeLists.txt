cmake_minimum_required(VERSION 3.10)

project(vtflib C CXX)

option(VTFLIB_STATIC "If enabled, builds VTFLIB as a static library rather than shared" ON)

include_directories(thirdparty/include)

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
	add_library(vtflib STATIC ${VTFLIB_SRCS})
	target_compile_definitions(vtflib PUBLIC VTFLIB_STATIC)
else()
	add_library(vtflib SHARED ${VTFLIB_SRCS})
endif()

# Link against compressonator libs
if (UNIX)
	target_link_directories(vtflib PUBLIC thirdparty/lib/x64)
	target_link_libraries(vtflib PUBLIC CMP_Compressonator pthread)
else()
	target_link_directories(vtflib PUBLIC thirdparty/lib)
	# Make sure to link against the correct variant
	target_link_libraries(vtflib PUBLIC "Compressonator_MT$<$<CONFIG:Debug>:d>")
endif()

target_compile_definitions(vtflib PRIVATE VTFLIB_EXPORTS)

target_precompile_headers(vtflib PRIVATE VTFLib/stdafx.h)
target_include_directories(vtflib PUBLIC lib)

