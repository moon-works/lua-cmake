cmake_minimum_required(VERSION 3.16)
project(lua LANGUAGES C VERSION 5.3.0)

# Base version
set(LUA_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")

set(CMAKE_C_STANDARD 99)
include(GNUInstallDirs)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

#### Configuration section
option(LUA_INSTALL_STATIC_LIBRARY "Install static library" OFF)
option(LUA_COMPAT_ALL "Enable deprecated code" ON)
option(LUA_USE_POSIX "Enable POSIX features" ON)
option(LUA_USE_DLOPEN "Enable DLOPEN feature" ON)
option(LUA_USE_READLINE "Enable Readline library support" ON)

if(CMAKE_SYSTEM_NAME MATCHES "Linux")
	set(LUA_USE_LINUX TRUE)
endif()

if(CMAKE_SYSTEM_NAME MATCHES "Darwin")
	set(LUA_USE_MACOSX TRUE)
endif()

if(CMAKE_SYSTEM_NAME MATCHES "Windows")
	set(LUA_USE_WINDOWS TRUE)
endif()

# make sure src/luaconf.h is not present
file(REMOVE src/luaconf.h)

configure_file(src/luaconf.h.in ${CMAKE_CURRENT_BINARY_DIR}/luaconf.h)
configure_file(lua.pc.in ${CMAKE_CURRENT_BINARY_DIR}/lua${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.pc
	@ONLY)

if(LUA_USE_READLINE)
	find_package(PkgConfig)
	pkg_check_modules(READLINE REQUIRED readline)
	include_directories(${READLINE_INCLUDE_DIRS})
endif()

if(LUA_USE_DLOPEN)
	set(LUA_LIBRARIES ${CMAKE_DL_LIBS})
endif()

include_directories(src ${CMAKE_CURRENT_BINARY_DIR})

set(LUA_HEADERS
	${CMAKE_CURRENT_BINARY_DIR}/luaconf.h
	src/lua.hpp
	src/lua.h
	src/lualib.h
	src/lauxlib.h
)

set(LUA_DOCS
	doc/contents.html
	doc/index.css
	doc/logo.gif
	doc/lua.css
	doc/manual.css
	doc/manual.html
	doc/osi-certified-72x60.png
	doc/readme.html
	README
)

set(LUA_CORE_SOURCES
	src/lapi.c
	src/lcode.c
	src/lctype.c
	src/ldebug.c
	src/ldo.c
	src/ldump.c
	src/lfunc.c
	src/lgc.c
	src/llex.c
	src/lmem.c
	src/lobject.c
	src/lopcodes.c
	src/lparser.c
	src/lstate.c
	src/lstring.c
	src/ltable.c
	src/ltm.c
	src/lundump.c
	src/lvm.c
	src/lzio.c
)

set(LUA_SOURCES
	src/lua.c
)

set(LUAC_SOURCES
	src/luac.c
)

set(LUA_LIB_SOURCES
	src/lauxlib.c
	src/lbaselib.c
	src/lbitlib.c
	src/lcorolib.c
	src/ldblib.c
	src/linit.c
	src/liolib.c
	src/lmathlib.c
	src/loadlib.c
	src/loslib.c
	src/lstrlib.c
	src/ltablib.c
	src/lutf8lib.c
)

# Common object library for all targets
add_library(lua_core OBJECT ${LUA_CORE_SOURCES})

# Shared liblua library
add_library(lua_shared SHARED ${LUA_LIB_SOURCES})
set_target_properties(lua_shared PROPERTIES OUTPUT_NAME "lua${LUA_VERSION}")
target_link_libraries(lua_shared lua_core ${LUA_LIBRARIES} m)

# Static liblua library
add_library(lua_static STATIC ${LUA_LIB_SOURCES})
set_target_properties(lua_static PROPERTIES OUTPUT_NAME "lua${LUA_VERSION}")
target_link_libraries(lua_static lua_core ${LUA_LIBRARIES} m)

# Lua interpreter
add_executable(lua_bin ${LUA_SOURCES})
set_target_properties(lua_bin PROPERTIES OUTPUT_NAME "lua${LUA_VERSION}")
target_link_libraries(lua_bin lua_core lua_shared ${READLINE_LIBRARIES})

# Lua compiler
add_executable(luac_bin ${LUAC_SOURCES})
set_target_properties(luac_bin PROPERTIES OUTPUT_NAME "luac${LUA_VERSION}")
target_link_libraries(luac_bin lua_core lua_static)

install(TARGETS lua_bin luac_bin lua_shared)
if(LUA_INSTALL_STATIC_LIBRARY)
	install(TARGETS lua_static)
endif()
install(FILES ${LUA_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lua${LUA_VERSION})
install(FILES ${LUA_DOCS} TYPE DOC)

# rename man files
configure_file(doc/lua.1 ${CMAKE_CURRENT_BINARY_DIR}/lua${LUA_VERSION}.1 COPYONLY)
configure_file(doc/luac.1 ${CMAKE_CURRENT_BINARY_DIR}/luac${LUA_VERSION}.1 COPYONLY)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/lua${LUA_VERSION}.1 ${CMAKE_CURRENT_BINARY_DIR}/luac${LUA_VERSION}.1
	DESTINATION ${CMAKE_INSTALL_MANDIR}/man1
)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/lua${LUA_VERSION}.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
