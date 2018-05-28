
cmake_minimum_required(VERSION 3.7.0)

project(simh)

# Do not pollute source directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/BIN)

# Global options
set(BUILD_ALL ON CACHE BOOL "Build all simulators")
set(ENABLE_ASYNC ON CACHE BOOL "Enable asynchronous I/O support")
set(ENABLE_GRAPHICS ON CACHE BOOL "Enable graphics support")
set(ENABLE_NETWORK ON CACHE BOOL "Enable network support")

# Find system libraries
find_library(LIBM_LIBRARIES m)
find_library(LIBDL_LIBRARIES dl)
find_library(LIBRT_LIBRARIES rt)
# Find curses
find_package(Curses)
# Find threads
find_package(Threads)
set(THREAD_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})
# Find pkg-config dependencies
find_package(PkgConfig)
pkg_check_modules(PCRE libpcre)
pkg_check_modules(PCREPOSIX libpcreposix)
if(ENABLE_GRAPHICS)
  pkg_check_modules(SDL1 sdl)
  pkg_check_modules(SDL2 sdl)
endif()
if(ENABLE_NETWORK)
  pkg_check_modules(VDEPLUG vdeplug)
endif()

# GNU toolchain
if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU")
  # should be C90, but code uses to many C++ comments...
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
  # enable lots of warnings
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-address")
  # allow setting the CPU architecture
  set(COMPILER_USE_ARCH "" CACHE STRING "Compile using specific CPU architecture")
  if(COMPILER_USE_ARCH)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=${COMPILER_USE_ARCH}")
  endif()
  # tell the optimizer about various pitfalls
  set(COMMON_SAFETY
	"-fno-strict-aliasing -fno-strict-overflow -fno-unsafe-loop-optimizations"
	)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_SAFETY}")
  # optimizations for release builds
  set(COMMON_OPTIMIZATIONS
	"-fgcse-after-reload -finline-functions"
	)
  # some optimizations for relwithdebinfo builds
  if("${CMAKE_BUILD_TYPE}" STREQUAL "RelWithDebInfo")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_OPTIMIZATIONS}")
  endif()
  # additional optimizations for release builds
  if("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
	set(COMPILER_OMIT_FP ON CACHE BOOL "Compile without frame pointers")
	set(COMPILER_USE_LTO ON CACHE BOOL "Compile with link-time optimization")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_OPTIMIZATIONS}")
	# omit frame pointer
	if(COMPILER_OMIT_FP)
	  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fomit-frame-pointer")
	endif()
	# enable link-time optimization
	if(COMPILER_USE_LTO)
	  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -flto")
	  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-fat-lto-objects")
	endif()
  endif()
endif()

# Definitions common to all simulators
set(SIM_DEFINITIONS
  _GNU_SOURCE
  HAVE_GLOB
  )
set(SIM_LIBRARIES
  ${LIBM_LIBRARIES}
  ${THREAD_LIBRARIES}
  )
set(SIM_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/scp.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_card.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_console.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_disk.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_ether.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_fio.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_frontpanel.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_imd.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_serial.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_sock.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_tape.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_timer.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_tmxr.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sim_video.c
  )

# Asynchronous I/O
if(ENABLE_ASYNC)
  set(SIM_DEFINITIONS ${SIM_DEFINITIONS} SIM_ASYNCH_IO)
endif()

# Dynamic linker
if(LIBDL_LIBRARIES)
  string(SUBSTRING ${CMAKE_SHARED_LIBRARY_SUFFIX} 1 -1 LIBDL_SUFFIX)
  set(SIM_DEFINITIONS ${SIM_DEFINITIONS} HAVE_DLOPEN=${LIBDL_SUFFIX})
  set(SIM_LIBRARIES ${SIM_LIBRARIES} ${LIBDL_LIBRARIES})
endif()

# Shared memory support
if(LIBRT_LIBRARIES)
  set(SIM_DEFINITIONS ${SIM_DEFINITIONS} HAVE_SHM_OPEN)
  set(SIM_LIBRARIES ${SIM_LIBRARIES} ${LIBRT_LIBRARIES})
endif()

# Regular expression support
if(PCRE_FOUND AND PCREPOSIX_FOUND)
  set(SIM_DEFINITIONS ${SIM_DEFINITIONS} HAVE_PCREPOSIX_H)
  set(SIM_INCLUDES ${SIM_INCLUDES} ${PCRE_INCLUDE_DIRS} ${PCREPOSIX_INCLUDE_DIRS})
  set(SIM_LIBRARIES ${SIM_LIBRARIES} ${PCRE_LIBRARIES} ${PCREPOSIX_LIBRARIES})
else()
  set(SIM_DEFINITIONS ${SIM_DEFINITIONS} HAVE_REGEX_H)
endif()

# Curses support
if(CURSES_FOUND)
  set(SIM_DEFINITIONS ${SIM_DEFINITIONS} HAVE_NCURSES)
  set(SIM_INCLUDES ${SIM_INCLUDES} ${CURSES_INCLUDE_DIRS})
  set(SIM_LIBRARIES ${SIM_LIBRARIES} ${CURSES_LIBRARIES})
endif()

# Graphics support
if(ENABLE_GRAPHICS)
  set(GRAPHICS_DISPLAY_DEFINITIONS
	USE_DISPLAY
	)
  set(GRAPHICS_DISPLAY_SOURCES
	${CMAKE_SOURCE_DIR}/display/display.c
	${CMAKE_SOURCE_DIR}/display/sim_ws.c
	)
  set(GRAPHICS_VT_SOURCES
	${CMAKE_SOURCE_DIR}/display/vt11.c
	)
  if(SDL2_FOUND)
	set(GRAPHICS_DEFINITIONS USE_SIM_VIDEO HAVE_LIBSDL)
	set(GRAPHICS_CFLAGS ${SDL2_CFLAGS})
	set(GRAPHICS_INCLUDES ${SDL2_INCLUDE_DIRS})
	set(GRAPHICS_LIBRARIES ${SDL2_LIBRARIES})
  elseif(SDL1_FOUND)
	set(GRAPHICS_DEFINITIONS USE_SIM_VIDEO HAVE_LIBSDL)
	set(GRAPHICS_CFLAGS ${SDL1_CFLAGS})
	set(GRAPHICS_INCLUDES ${SDL1_INCLUDE_DIRS})
	set(GRAPHICS_LIBRARIES ${SDL1_LIBRARIES})
  endif()
endif()

# Network support
if(ENABLE_NETWORK)
  set(NETWORK_DEFINITIONS USE_NETWORK)
  if(VDEPLUG_FOUND)
	set(NETWORK_DEFINITIONS ${NETWORK_DEFINITIONS} HAVE_VDE_NETWORK)
	set(NETWORK_INCLUDES ${NETWORK_INCLUDES} ${VDEPLUG_INCLUDE_DIRS})
	set(NETWORK_LIBRARIES ${NETWORK_LIBRARIES} ${VDEPLUG_LIBRARIES})
  endif()
endif()

# Convenience function for defining simulators
function(add_simulator NAME)
  string(TOUPPER ${NAME} NAMEU)
  if(BUILD_ALL)
	set(SIMULATOR_${NAMEU}_ENABLE ON)
  else()
	set(SIMULATOR_${NAMEU}_ENABLE OFF CACHE BOOL "Build the ${NAME} simulator")
  endif()
  if(SIMULATOR_${NAMEU}_ENABLE)
	add_executable(${NAME} ${SIM_SOURCES} ${${NAMEU}_SOURCES})
	target_link_libraries(${NAME} PUBLIC
	  ${SIM_LIBRARIES}
	  ${${NAMEU}_LIBRARIES})
	target_compile_definitions(${NAME} PUBLIC
	  ${SIM_DEFINITIONS}
	  ${${NAMEU}_DEFINITIONS}
	  )
	target_include_directories(${NAME} PUBLIC
	  ${SIM_INCLUDES}
	  ${${NAMEU}_INCLUDES}
	  ${CMAKE_SOURCE_DIR}
	  ${CMAKE_CURRENT_SOURCE_DIR}
	  )
	install(TARGETS ${NAME}
	  RUNTIME DESTINATION bin
	  )
  endif()
endfunction()

# Function for adding graphics support
function(add_simulator_graphics NAME)
  string(TOUPPER ${NAME} NAMEU)
  if(SIMULATOR_${NAMEU}_ENABLE)
	if(ENABLE_GRAPHICS)
	  set(SIMULATOR_${NAMEU}_GRAPHICS ON CACHE BOOL "Enable ${NAME} graphics support")
	  if(${SIMULATOR_${NAMEU}_GRAPHICS})
		target_link_libraries(${NAME} PUBLIC ${GRAPHICS_LIBRARIES})
		target_compile_definitions(${NAME} PUBLIC ${GRAPHICS_DEFINITIONS})
		target_include_directories(${NAME} PUBLIC ${GRAPHICS_INCLUDES})
	  endif()
	endif()
  endif()
endfunction()

# Function for adding network support
function(add_simulator_network NAME)
  string(TOUPPER ${NAME} NAMEU)
  if(SIMULATOR_${NAMEU}_ENABLE)
	if(ENABLE_NETWORK)
	  set(SIMULATOR_${NAMEU}_NETWORK ON CACHE BOOL "Enable ${NAME} network support")
	  target_link_libraries(${NAME} PUBLIC ${NETWORK_LIBRARIES})
	  target_compile_definitions(${NAME} PUBLIC ${NETWORK_DEFINITIONS})
	  target_include_directories(${NAME} PUBLIC ${NETWORK_INCLUDES})
	endif()
  endif()
endfunction()

# Advanced Computer Design
add_subdirectory(PDQ-3)
# AT&T
add_subdirectory(3B2)
# Burroughs Corporation
add_subdirectory(B5500)
# Control Data Corporation (CDC)
add_subdirectory(CDC1700)
# Data General (DG)
add_subdirectory(NOVA)
# Digital Equipment Corporation (DEC)
add_subdirectory(alpha)
add_subdirectory(PDP1)
add_subdirectory(PDP8)
add_subdirectory(PDP10)
add_subdirectory(PDP11)
add_subdirectory(PDP18B)
add_subdirectory(VAX)
# GRI Computer Corporation
add_subdirectory(GRI)
# Hewlett-Packard (HP)
add_subdirectory(HP2100)
add_subdirectory(HP3000)
# Honeywell
add_subdirectory(H316)
# Intel-based computers
add_subdirectory(Intel-Systems)
# Interdata / Perkin-Elmer
add_subdirectory(Interdata)
# International Business Machines (IBM)
add_subdirectory(Ibm1130)
add_subdirectory(I650)
add_subdirectory(I1401)
add_subdirectory(I1620)
add_subdirectory(I7000)
add_subdirectory(I7094)
add_subdirectory(S3)
# Manchester University
add_subdirectory(SSEM)
# Massachusetts Institute of Technology (MIT)
add_subdirectory(TX-0)
# Micro Instrumentation and Telemetry Systems (MITS)
add_subdirectory(ALTAIR)
add_subdirectory(AltairZ80)
# Royal MacBee
add_subdirectory(LGP)
# Sage Technology
add_subdirectory(SAGE)
# Scientific Data Systems (SDS)
add_subdirectory(SDS)
add_subdirectory(sigma)
# Soviet union
add_subdirectory(BESM6)
# Southwest Technical Products
add_subdirectory(swtp6800)
