#Use the lastest Cmake policy on FindCmake Includes
cmake_policy(SET CMP0017 NEW)

#Lua is a powerful, fast, lightweight, embeddable scripting 
#language developed by a team at PUC-Rio, the Pontifical Catholic University 
#of Rio de Janeiro in Brazil. Lua is free software used in many 
#products and projects around the world. 
#Main Github repository @ https://github.com/Stratagus/Lua
#bradc6 Github repository @ https://github.com/bradc6/Lua

project(lua)
cmake_minimum_required(VERSION 2.6)

#Set the current version of the library
set(LIBLUA_MAJOR_VERSION 5)
set(LIBLUA_MINOR_VERSION 2)
set(LIBLUA_PATCH_VERSION 3)

#Sets the directory for cmake to find Cmake configration packages,
#these packages allow for cmake to find libraries that are not included
#in the standard installation of Cmake. The standard packages can be
#found @ /usr/share/cmake-2.8/Modules
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules
                      ${CMAKE_MODULE_PATH})

#Source directory variables
#These allow for folders in the source and unittesting to be moved
#without having to manually change each individual filepath, they
#also act as shortcuts.
set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Source")
set(SAMPLESOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/SampleSource")
set(UNITTESTS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/UnitTests")

#These variables will act as our list of include folders and linker flags
#for compilation
set(LuaLinkerFlags)
set(LuaIncludeDirectories)
set(LuaLinkerDirectories)

###Platform dependant include/linker Directories
###Add fakeroot directores for different platforms
if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    list(APPEND LuaIncludeDirectories "$ENV{HOMEPATH}/dev/fakeroot/include")
    list(APPEND LuaLinkerDirectories "$ENV{HOMEPATH}/dev/fakeroot/lib")
    include_directories("C:/fakeroot/include")
    link_directories("C:/fakeroot/lib")
    add_definitions(-DTARGETWINDOWS=1)
    if(MSVC)
        add_definitions(/arch:SSE2)
    endif(MSVC)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    list(APPEND LuaIncludeDirectories "~/dev/fakeroot/include")
    list(APPEND LuaLinkerDirectories "~/dev/fakeroot/lib")
    list(APPEND LuaIncludeDirectories "/opt/local/include")
    list(APPEND LuaLinkerDirectories "/opt/local/lib")
    #Default towards macports over /usr/include-lib
    list(APPEND CMAKE_PREFIX_PATH /opt/local)
    add_definitions(-DLUA_USE_MACOSX=1)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	dd_definitions(-DLUA_USE_LINUX=1)
else()
    list(APPEND LuaIncludeDirectories "~/dev/fakeroot/include")
    list(APPEND LuaLinkerDirectories "~/dev/fakeroot/lib")
endif()

#Find dependant libraries
find_package(Readline REQUIRED)
list(APPEND LuaIncludeDirectories "${Readline_INCLUDE_DIR}")
list(APPEND LuaLinkerFlags "${Readline_LIBRARY}")

include_directories(${LuaIncludeDirectories})
link_directories(${LuaLinkerDirectories})

set(LUA_PUBLIC_HEADERS
		${SOURCE_DIR}/lua.h
		${SOURCE_DIR}/lua.hpp
		${SOURCE_DIR}/luaconf.h
		${SOURCE_DIR}/lualib.h
		${SOURCE_DIR}/lauxlib.h
	)

set(LUA_CORE_SOURCE
		${SOURCE_DIR}/lapi.h
		${SOURCE_DIR}/lapi.c
		${SOURCE_DIR}/lcode.h
		${SOURCE_DIR}/lcode.c
		${SOURCE_DIR}/lctype.h
		${SOURCE_DIR}/lctype.c
		${SOURCE_DIR}/ldebug.h
		${SOURCE_DIR}/ldebug.c
		${SOURCE_DIR}/ldo.h
		${SOURCE_DIR}/ldo.c
		${SOURCE_DIR}/ldump.c
		${SOURCE_DIR}/lfunc.h
		${SOURCE_DIR}/lfunc.c
		${SOURCE_DIR}/lgc.h
		${SOURCE_DIR}/lgc.c
		${SOURCE_DIR}/llex.h
		${SOURCE_DIR}/llex.c
		${SOURCE_DIR}/lmem.h
		${SOURCE_DIR}/lmem.c
		${SOURCE_DIR}/lobject.h
		${SOURCE_DIR}/lobject.c
		${SOURCE_DIR}/lopcodes.h
		${SOURCE_DIR}/lopcodes.c
		${SOURCE_DIR}/lparser.h
		${SOURCE_DIR}/lparser.c
		${SOURCE_DIR}/lstate.h
		${SOURCE_DIR}/lstate.c
		${SOURCE_DIR}/lstring.h
		${SOURCE_DIR}/lstring.c
		${SOURCE_DIR}/ltable.h
		${SOURCE_DIR}/ltable.c
		${SOURCE_DIR}/ltm.h
		${SOURCE_DIR}/ltm.c
		${SOURCE_DIR}/lundump.h
		${SOURCE_DIR}/lundump.c
		${SOURCE_DIR}/lvm.h
		${SOURCE_DIR}/lvm.c
		${SOURCE_DIR}/lzio.h
		${SOURCE_DIR}/lzio.c
	)

set(LUA_LIBRARY_SOURCE
		${SOURCE_DIR}/lauxlib.c
		${SOURCE_DIR}/lbaselib.c
		${SOURCE_DIR}/lbitlib.c
		${SOURCE_DIR}/lcorolib.c
		${SOURCE_DIR}/ldblib.c
		${SOURCE_DIR}/liolib.c
		${SOURCE_DIR}/lmathlib.c
		${SOURCE_DIR}/loslib.c
		${SOURCE_DIR}/lstrlib.c
		${SOURCE_DIR}/ltablib.c
		${SOURCE_DIR}/loadlib.c
		${SOURCE_DIR}/linit.c
	)

set(LUA_INTERPRETER_SOURCE
		${SOURCE_DIR}/lua.c
	)

set(LUAC_INTERPRETER_SOURCE
		${SOURCE_DIR}/luac.c
	)

#Source Groups allow for IDEs to look pretty
source_group(Core FILES ${LUA_CORE_SOURCE})
source_group(Library FILES ${LUA_LIBRARY_SOURCE})
source_group(Lua FILES ${LUA_INTERPRETER_SOURCE})
source_group(Luac FILES ${LUAC_INTERPRETER_SOURCE})

#Added the library statically
add_executable(lua-interpreter ${LUA_CORE_SOURCE} ${LUA_LIBRARY_SOURCE} ${LUA_INTERPRETER_SOURCE})
target_link_libraries(lua-interpreter ${LuaLinkerFlags})
set_target_properties(lua-interpreter PROPERTIES OUTPUT_NAME "lua")

add_executable(luac-interpreter ${LUA_CORE_SOURCE} ${LUA_LIBRARY_SOURCE} ${LUAC_INTERPRETER_SOURCE})
target_link_libraries(luac-interpreter ${LuaLinkerFlags})
set_target_properties(luac-interpreter PROPERTIES OUTPUT_NAME "luac")

add_library(lua SHARED ${LUA_CORE_SOURCE} ${LUA_LIBRARY_SOURCE})
target_link_libraries(lua ${LuaLinkerFlags})
set_target_properties(lua PROPERTIES 
						OUTPUT_NAME "lua.${LIBLUA_MAJOR_VERSION}.${LIBLUA_MINOR_VERSION}.${LIBLUA_PATCH_VERSION}")

add_library(lua-static STATIC ${LUA_CORE_SOURCE} ${LUA_LIBRARY_SOURCE})
target_link_libraries(lua-static ${LuaLinkerFlags})
set_target_properties(lua-static PROPERTIES 
						OUTPUT_NAME "lua.${LIBLUA_MAJOR_VERSION}.${LIBLUA_MINOR_VERSION}.${LIBLUA_PATCH_VERSION}")