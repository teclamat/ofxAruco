# ----------------------------------------------------------------------------
#   Basic Configuration
# ----------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.5)

project(aruco VERSION 1.3.0 LANGUAGES CXX)

set(PROJECT_SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")

#------------------------------------------------------
# Build type
#------------------------------------------------------

IF(NOT CMAKE_BUILD_TYPE )
   SET( CMAKE_BUILD_TYPE "Debug" )
ENDIF()

#------------------------------------------------------
# Lib Names and Dirs
#------------------------------------------------------

if(WIN32)
    # Postfix of DLLs:
    # Postfix of DLLs:
    SET(PROJECT_DLLVERSION "${PROJECT_VERSION_MAJOR}${PROJECT_VERSION_MINOR}${PROJECT_VERSION_PATCH}")
    SET(RUNTIME_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Directory for dlls and binaries")
    SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Directory for binaries")
    SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Directory for dlls")

    OPTION(BUILD_GLSAMPLES 	"Set to OFF to build static libraries" OFF)
else()
    # Postfix of so's:
    set(PROJECT_DLLVERSION)
    OPTION(BUILD_GLSAMPLES 	"Set to OFF to build static libraries" OFF)
#    SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_INSTALL_PREFIX}/lib/cmake/ /usr/lib/cmake)
endif()

OPTION(BUILD_UTILS	"Set to OFF to not compile utils " OFF)
OPTION(BUILD_SHARED_LIBS 	"Set to OFF to build static libraries" OFF)

# ----------------------------------------------------------------------------
#   Find Dependencies
# ----------------------------------------------------------------------------

find_package(OpenCV REQUIRED)
set (REQUIRED_LIBRARIES ${OpenCV_LIBS})
INCLUDE_DIRECTORIES(${OpenCV_INCLUDE_DIRS})

# Check if we have OpenCV 3
string(REPLACE "." ";" OPENCV_VERSION_LIST ${OpenCV_VERSION})
list(GET OPENCV_VERSION_LIST 0 OPENCV_VERSION_MAJOR)
IF( ${OPENCV_VERSION_MAJOR} STREQUAL "3" )
    ADD_DEFINITIONS(-DOPENCV_VERSION_3)
ENDIF()


IF(EXISTS ${GLUT_PATH})
    INCLUDE_DIRECTORIES(${GLUT_PATH}/include)
    SET(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} ${GLUT_PATH}/lib)
ENDIF()

##LOOK FOR OPENGL AND GLUT
#FIND OPENGL LIBRARY. In Android, there is no need since it is included
IF(NOT ANDROID_CREATION)
if(BUILD_GLSAMPLES)
  find_package(OpenGL)
  find_package(GLUT)#standard package
  MESSAGE(STATUS "GLUT_FOUND=${GLUT_FOUND} OPENGL_gl_LIBRARY=${OPENGL_gl_LIBRARY} GLUT_HEADER=${GLUT_HEADER}")
endif()
  IF(NOT GLUT_FOUND) #else, freeglut
    FIND_LIBRARY(GLUT_glut_LIBRARY     NAMES freeglut)
    MESSAGE(STATUS "GLUT_glut_LIBRARY=${GLUT_glut_LIBRARY}")
  ENDIF()
  IF ( (NOT GLUT_glut_LIBRARY AND NOT GLUT_FOUND) OR NOT OPENGL_gl_LIBRARY)
    set(GL_FOUND "NO")
  ELSE()
    set(GL_FOUND "YES")
    set (OPENGL_LIBS  ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY} ${GLUT_glut_LIBRARY})
  ENDIF()
ENDIF()
#Omp


if(WIN32)

ELSE()
find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    list(APPEND REQUIRED_LIBRARIES OpenMP::OpenMP_CXX)
add_definitions(-DUSE_OMP)
endif()
ENDIF()

# ----------------------------------------------------------------------------
#   PROJECT CONFIGURATION
#   force some variables that could be defined in the command line to be written to cache
# ----------------------------------------------------------------------------
OPTION(INSTALL_DOC 	"Set to ON to build/install Documentation" OFF)
IF (INSTALL_DOC)
    FIND_PACKAGE(Doxygen REQUIRED)
    MESSAGE( STATUS "INSTALL_DOC:         ${INSTALL_DOC} ")
    INCLUDE("${PROJECT_SOURCE_DIR}/generateDoc.cmake")
    GENERATE_DOCUMENTATION(${PROJECT_SOURCE_DIR}/dox.in)
ENDIF()




# ----------------------------------------------------------------------------
#   Uninstall target, for "make uninstall"
# ----------------------------------------------------------------------------
CONFIGURE_FILE( "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in" "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake" IMMEDIATE @ONLY)
ADD_CUSTOM_TARGET(uninstall "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")

# ----------------------------------------------------------------------------
# create configuration file from .in file (If you use windows take care with paths)
# ----------------------------------------------------------------------------

CONFIGURE_FILE("${PROJECT_SOURCE_DIR}/config.cmake.in" "${PROJECT_BINARY_DIR}/Find${PROJECT_NAME}.cmake")
INSTALL(FILES "${PROJECT_BINARY_DIR}/Find${PROJECT_NAME}.cmake" DESTINATION lib/cmake/ )




# ----------------------------------------------------------------------------
#   Program Optimization and debug (Extracted from OpenCV)
# ----------------------------------------------------------------------------
set(WARNINGS_ARE_ERRORS 		OFF CACHE BOOL "Treat warnings as errors")
set(WHOLE_PROGRAM_OPTIMIZATION 	OFF CACHE BOOL "Flags for whole program optimization.")

set(EXTRA_C_FLAGS "")
set(EXTRA_C_FLAGS_RELEASE "")
set(EXTRA_C_FLAGS_DEBUG "")
set(EXTRA_EXE_LINKER_FLAGS "")
set(EXTRA_EXE_LINKER_FLAGS_RELEASE "")
set(EXTRA_EXE_LINKER_FLAGS_DEBUG "")

IF(CMAKE_COMPILER_IS_GNUCXX OR MINGW)
    set(ENABLE_PROFILING 		OFF CACHE BOOL "Enable profiling in the GCC compiler (Add flags: -g -pg)")
    set(USE_OMIT_FRAME_POINTER 	ON CACHE BOOL "Enable -fomit-frame-pointer for GCC")
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES arm*) # We can use only -O2 because the -O3 causes gcc crash
        set(USE_O2 ON CACHE BOOL "Enable -O2 for GCC")
        set(USE_FAST_MATH ON CACHE BOOL "Enable -ffast-math for GCC")
    endif()
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES powerpc*)
        set(USE_O3 ON CACHE BOOL "Enable -O3 for GCC")
        set(USE_POWERPC ON CACHE BOOL "Enable PowerPC for GCC")
    endif ()
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES amd64* OR ${CMAKE_SYSTEM_PROCESSOR} MATCHES x86_64*)
        set(USE_O3 ON CACHE BOOL "Enable -O3 for GCC")
        set(USE_FAST_MATH ON CACHE BOOL "Enable -ffast-math for GCC")
        set(USE_MMX ON CACHE BOOL "Enable MMX for GCC")
        set(USE_SSE ON CACHE BOOL "Enable SSE for GCC")
        set(USE_SSE2 ON CACHE BOOL "Enable SSE2 for GCC")
        set(USE_SSE3 ON CACHE BOOL "Enable SSE3 for GCC")
    endif()
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES i686* OR ${CMAKE_SYSTEM_PROCESSOR} MATCHES x86)
        set(USE_O3 ON CACHE BOOL "Enable -O3 for GCC")
        set(USE_FAST_MATH ON CACHE BOOL "Enable -ffast-math for GCC")
        set(USE_MMX ON CACHE BOOL "Enable MMX for GCC")
        set(USE_SSE OFF CACHE BOOL "Enable SSE for GCC")
        set(USE_SSE2 OFF CACHE BOOL "Enable SSE2 for GCC")
        set(USE_SSE3 OFF CACHE BOOL "Enable SSE3 for GCC")
    endif ()

    set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -Wall")

    if(WARNINGS_ARE_ERRORS)
        set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -Werror")
    endif()

    # The -Wno-long-long is required in 64bit systems when including sytem headers.
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES x86_64* OR ${CMAKE_SYSTEM_PROCESSOR} MATCHES amd64*)
		set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -Wno-long-long")
    endif()

    # Whole program optimization
    if(WHOLE_PROGRAM_OPTIMIZATION)
        set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -fwhole-program --combine")
    endif()

    # Other optimizations
    if(USE_OMIT_FRAME_POINTER)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -fomit-frame-pointer")
    endif()
    if(USE_O2)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -O2")
    endif()
    if(USE_O3)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -O3")
    endif()
    if(USE_FAST_MATH)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -ffast-math")
    endif()
    if(USE_POWERPC)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -mcpu=G3 -mtune=G5")
    endif()
    if(USE_MMX)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -mmmx")
    endif()
    if(USE_SSE)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -msse")
    endif()
    if(USE_SSE2)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -msse2")
    endif()
    if(USE_SSE3 AND NOT MINGW) # SSE3 should be disabled under MingW because it generates compiler errors
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -msse3")
    endif()

    if(ENABLE_PROFILING)
        set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -pg -g")
    else()
        if(NOT APPLE)
            set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -ffunction-sections")
        endif()
    endif()

    # Parallel mode
    if(ENABLE_OPENMP)
        set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS}  -fopenmp")
        set(LINKER_LIBS ${LINKER_LIBS} gomp)
    endif()
    IF(${CMAKE_SYSTEM_PROCESSOR} MATCHES armv7l) # In ARM_COrtex8 with neon, enalble vectorized operations
	SET(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -mcpu=cortex-a8 -mfpu=neon -mfloat-abi=softfp -ftree-vectorize ")
    ENDIF()


    set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -DNDEBUG")
    set(EXTRA_C_FLAGS_DEBUG "-g3 -O0 -DDEBUG -D_DEBUG -W -Wextra -Wno-return-type")

    MESSAGE( STATUS "-------------------------------------------------------------------------------" )
    message( STATUS "GNU COMPILER")
    MESSAGE( STATUS "-------------------------------------------------------------------------------" )

    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${EXTRA_C_FLAGS}")
    SET(CMAKE_CXX_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE}")
    SET(CMAKE_CXX_FLAGS_DEBUG "${EXTRA_C_FLAGS_DEBUG}")
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${EXTRA_EXE_LINKER_FLAGS}")
    SET(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} ${EXTRA_EXE_LINKER_FLAGS_RELEASE}")
    SET(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} ${EXTRA_EXE_LINKER_FLAGS_DEBUG}")
ENDIF	()

# ----------------------------------------------------------------------------
#   PACKAGE GENERATOR
# ----------------------------------------------------------------------------

IF (UNIX)
SET(CPACK_GENERATOR "DEB")
SET(CPACK_PACKAGE_CONTACT "rmsalinas@uco.es")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "rmsalinas@uco.es")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "ArUco Library")
SET(CPACK_PACKAGE_VENDOR "Rafael Munoz-Salinas. AVA Group. University of Cordoba. Spain")
SET(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
SET(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
SET(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
SET(CPACK_PACKAGE_INSTALL_DIRECTORY "CMake ${CMake_VERSION_MAJOR}.${CMake_VERSION_MINOR}")
SET(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
SET(CPACK_STRIP_FILES "bin/lib${PROJECT_NAME}")
SET(CPACK_SOURCE_STRIP_FILES "")
SET(CPACK_PACKAGING_INSTALL_PREFIX "/usr/")
IF (CPACK_GENERATOR STREQUAL "DEB")
    SET(CPACK_DEBIAN_PACKAGE_DEPENDS "opencv (>= 2.1.0)")
ENDIF()
INCLUDE(CPack)
ENDIF()

#------------------------------------------------
# DIRS
#------------------------------------------------
ADD_SUBDIRECTORY(src)
IF(BUILD_UTILS)
  ADD_SUBDIRECTORY(utils)
  ADD_SUBDIRECTORY(utils_hrm)
ENDIF()



# ----------------------------------------------------------------------------
# display status message for important variables
# ----------------------------------------------------------------------------
message( STATUS )
MESSAGE( STATUS "-------------------------------------------------------------------------------" )
message( STATUS "General configuration for ${PROJECT_NAME} ${PROJECT_VERSION}")
MESSAGE( STATUS "-------------------------------------------------------------------------------" )
message( STATUS )
message("    Built as dynamic libs?:" ${BUILD_SHARED_LIBS})
message("    Compiler:"                   "${CMAKE_COMPILER}"   "${CMAKE_CXX_COMPILER}")
message( STATUS "C++ flags (Release):       ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELEASE}")
message( STATUS "C++ flags (Debug):         ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_DEBUG}")
message( STATUS "CMAKE_CXX_FLAGS:         ${CMAKE_CXX_FLAGS}")
message( STATUS "CMAKE_BINARY_DIR:         ${CMAKE_BINARY_DIR}")

MESSAGE( STATUS )
MESSAGE( STATUS "CMAKE_SYSTEM_PROCESSOR = ${CMAKE_SYSTEM_PROCESSOR}" )
MESSAGE( STATUS "BUILD_SHARED_LIBS = ${BUILD_SHARED_LIBS}" )
MESSAGE( STATUS "CMAKE_INSTALL_PREFIX = ${CMAKE_INSTALL_PREFIX}" )
MESSAGE( STATUS "CMAKE_BUILD_TYPE = ${CMAKE_BUILD_TYPE}" )
MESSAGE( STATUS "CMAKE_MODULE_PATH = ${CMAKE_MODULE_PATH}" )

MESSAGE( STATUS )
MESSAGE( STATUS "---------------------------     Documentation     -----------------------------" )
MESSAGE( STATUS )
MESSAGE( STATUS "INSTALL_DOC = ${INSTALL_DOC}" )
MESSAGE( STATUS "USE_LATEX = ${USE_LATEX}" )
MESSAGE( STATUS "USE_DOT = ${USE_DOT}" )
MESSAGE( STATUS "USE_CHM = ${USE_CHM}" )
MESSAGE( STATUS )
MESSAGE( STATUS "FOUND OPENGL=${GL_FOUND}    LIBS=${OPENGL_LIBS}")
MESSAGE( STATUS "OpenCV_LIB_DIR=${OpenCV_LIB_DIR}")
MESSAGE( STATUS "CMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}")
MESSAGE( STATUS )
MESSAGE( STATUS )
MESSAGE( STATUS "Change a value with: cmake -D<Variable>=<Value>" )
MESSAGE( STATUS )

# pkg-config
# configure_file(aruco.pc.in aruco.pc @ONLY)
# configure_file(aruco-uninstalled.pc.in aruco-uninstalled.pc @ONLY)
# install(FILES "${PROJECT_BINARY_DIR}/aruco-uninstalled.pc" "${PROJECT_BINARY_DIR}/aruco.pc" DESTINATION lib/pkgconfig)
