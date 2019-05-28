cmake_minimum_required (VERSION 2.6) 
project (Plist) 

include_directories (include)

set (SCRIPT_EXT sh)
set (MY_BUILD_TYPE ${CMAKE_BUILD_TYPE})
IF(APPLE)
	set (ARCH_DIR OSX)
	# on Mavericks, need to link to libstdc++ directly because libc++ is default
	# and won't work
	if(NOT ${CMAKE_SYSTEM_VERSION} VERSION_LESS 13.0)
		set(EXTRA_CXX_FLAGS "-stdlib=libstdc++")
		set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -lstdc++")
		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -lstdc++")
	endif()
ELSE()
	set (ARCH_DIR Linux)
ENDIF()

link_directories (${CMAKE_SOURCE_DIR}/${ARCH_DIR}/lib)
include_directories (include ${CMAKE_SOURCE_DIR}/${ARCH_DIR}/include)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${EXTRA_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${EXTRA_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${EXTRA_CXX_FLAGS} -Wall -DTEST_VERBOSE")

add_executable(runTests src/runTests.cpp src/plistTests.cpp src/pugixml.cpp
    src/Plist.cpp src/PlistDate.cpp)

set(libUnitTest_DIR "/hackmitel/tools/unittest-cpp")

find_library(LIBUNITTEST_CPP         libUnitTest++.a         "${libUnitTest_DIR}/lib"                              ) 

message(STATUS "Found (LIBUNITTEST_CPP): ${LIBUNITTEST_CPP}")

target_link_libraries(runTests ${LIBUNITTEST_CPP})

ADD_CUSTOM_COMMAND(
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
	TARGET runTests
	POST_BUILD
	COMMAND ./runTests.${SCRIPT_EXT} ${MY_BUILD_TYPE} ${ARCH_DIR}
	)

set(HDRS 
    src/base64.hpp
    src/Plist.hpp
    src/PlistDate.hpp
    src/pugiconfig.hpp
    src/pugixml.hpp)

set(SRCS 
    src/pugixml.cpp)

add_library(plistpp ${SRCS})

foreach(x ${HDRS})
    install(FILES ${x} DESTINATION include)
endforeach(x)
