
PROJECT(sisl)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)


# Organize the project in folders (VS only?)
SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)


# Include directories

INCLUDE_DIRECTORIES(
  ${sisl_SOURCE_DIR}/include
  src/main/cpp/include/
  )


# Make the sisl library

FILE(GLOB sisl_SRCS src/main/cpp/*.c src/main/cpp/include/*.h)
ADD_LIBRARY(sisl STATIC ${sisl_SRCS})
IF(CMAKE_COMPILER_IS_GNUCC)
    TARGET_LINK_LIBRARIES(sisl m)
ENDIF(CMAKE_COMPILER_IS_GNUCC)
SET_PROPERTY(TARGET sisl
  PROPERTY FOLDER "sisl/Libs")

# This line may be needed only on AMD 64 bit...
#IF(NOT WIN32)
#  SET_TARGET_PROPERTIES(sisl PROPERTIES COMPILE_FLAGS "-fPIC")
#ENDIF(NOT WIN32)


# Set various compiler flags
IF(CMAKE_COMPILER_IS_GNUXX)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-unused-but-set-variable -fPIC")
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-unused-but-set-variable -fPIC")
ENDIF(CMAKE_COMPILER_IS_GNUXX)
IF(WIN32)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP8 -D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS")
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /MP8 -D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS")
ENDIF(WIN32)


# Apps, examples, tests, ...?

FILE(GLOB sisl_APPS app/*.C)
FOREACH(app ${sisl_APPS})
  GET_FILENAME_COMPONENT(appname ${app} NAME_WE)
  ADD_EXECUTABLE(${appname} ${app})
  TARGET_LINK_LIBRARIES(${appname} sisl)
  SET_PROPERTY(TARGET ${appname}
    PROPERTY FOLDER "sisl/Apps")
ENDFOREACH(app)



# 'install' target

IF(WIN32)
  # Windows
  SET(CMAKE_INSTALL_PREFIX CACHE INTERNAL "")
  SET(sisl_INSTALL_PREFIX "$ENV{PROGRAMFILES}/SINTEF/sisl"
    CACHE PATH "Path to install SISL")
  INSTALL(TARGETS sisl DESTINATION ${sisl_INSTALL_PREFIX}/lib)
  INSTALL(DIRECTORY include/
    DESTINATION ${sisl_INSTALL_PREFIX}/include
    FILES_MATCHING PATTERN "*.h"
    PATTERN ".svn" EXCLUDE
    )
ELSE(WIN32)
  # Linux
  # lib
  INSTALL(TARGETS sisl DESTINATION lib COMPONENT sisl)
  # include
  INSTALL(DIRECTORY include/
    COMPONENT sisl-dev
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
    PATTERN ".svn" EXCLUDE
    )
ENDIF(WIN32)


# CPack stuff
IF(NOT GoTools_ALL_COMPILE)
  SET(CPACK_SOURCE_PACKAGE_FILE_NAME "sisl-4.6.0")
  SET(CPACK_SOURCE_IGNORE_FILES "/CVS/;/\\.svn/;\\.swp$;\\.#;/#;.*~")
  INCLUDE(CPack)
ENDIF(NOT GoTools_ALL_COMPILE)

SET(CPACK_STRIP_FILES ${CPACK_STRIP_FILES} libsisl.so)
