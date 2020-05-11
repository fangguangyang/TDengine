CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
PROJECT(TDengine)

INCLUDE_DIRECTORIES(inc)
INCLUDE_DIRECTORIES(jni)
INCLUDE_DIRECTORIES(${TD_COMMUNITY_DIR}/src/inc)
INCLUDE_DIRECTORIES(${TD_OS_DIR}/inc)
AUX_SOURCE_DIRECTORY(./src SRC)

IF ((TD_LINUX_64) OR (TD_LINUX_32 AND TD_ARM))
  INCLUDE_DIRECTORIES(${TD_COMMUNITY_DIR}/deps/jni/linux)

  # set the static lib name
  ADD_LIBRARY(tscli_static STATIC ${SRC})
  TARGET_LINK_LIBRARIES(tscli_static trpc tutil pthread m rt)
  SET_TARGET_PROPERTIES(tscli_static PROPERTIES OUTPUT_NAME "tscli_static")
  SET_TARGET_PROPERTIES(tscli_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

  # generate dynamic library (*.so)
  ADD_LIBRARY(tscli SHARED ${SRC})
  TARGET_LINK_LIBRARIES(tscli trpc tutil pthread m rt)
  SET_TARGET_PROPERTIES(tscli PROPERTIES CLEAN_DIRECT_OUTPUT 1)

  #set version of .so
  #VERSION    so version
  #SOVERSION  api version
  execute_process(COMMAND chmod 777 ${TD_COMMUNITY_DIR}/packaging/tools/get_version.sh)
  execute_process(COMMAND ${TD_COMMUNITY_DIR}/packaging/tools/get_version.sh ${TD_COMMUNITY_DIR}/src/util/src/version.c
        OUTPUT_VARIABLE
        VERSION_INFO)
  MESSAGE(STATUS "build version ${VERSION_INFO}")
  SET_TARGET_PROPERTIES(tscli PROPERTIES VERSION ${VERSION_INFO} SOVERSION 1)
  
ELSEIF (TD_WINDOWS_64)
  INCLUDE_DIRECTORIES(${TD_COMMUNITY_DIR}/deps/jni/windows)
  INCLUDE_DIRECTORIES(${TD_COMMUNITY_DIR}/deps/jni/windows/win32)
  INCLUDE_DIRECTORIES(${TD_COMMUNITY_DIR}/deps/pthread)
  
  ADD_LIBRARY(tscli_static STATIC ${SRC})
  TARGET_LINK_LIBRARIES(tscli_static trpc tutil)

  # generate dynamic library (*.dll)
  ADD_LIBRARY(tscli SHARED ${SRC})
  IF (NOT TD_GODLL)
    SET_TARGET_PROPERTIES(tscli PROPERTIES LINK_FLAGS /DEF:${TD_COMMUNITY_DIR}/src/client/src/tscli.def)
  ENDIF ()
  TARGET_LINK_LIBRARIES(tscli trpc)

ELSEIF (TD_DARWIN_64)
  SET(CMAKE_MACOSX_RPATH 1)
  INCLUDE_DIRECTORIES(${TD_COMMUNITY_DIR}/deps/jni/linux)

  ADD_LIBRARY(tscli_static STATIC ${SRC})
  TARGET_LINK_LIBRARIES(tscli_static trpc tutil pthread m)
  SET_TARGET_PROPERTIES(tscli_static PROPERTIES OUTPUT_NAME "tscli_static")

  # generate dynamic library (*.dylib)
  ADD_LIBRARY(tscli SHARED ${SRC})
  TARGET_LINK_LIBRARIES(tscli trpc tutil pthread m)

  SET_TARGET_PROPERTIES(tscli PROPERTIES CLEAN_DIRECT_OUTPUT 1)

  #set version of .so
  #VERSION    so version
  #SOVERSION  api version
  execute_process(COMMAND chmod 777 ${TD_COMMUNITY_DIR}/packaging/tools/get_version.sh)
  execute_process(COMMAND ${TD_COMMUNITY_DIR}/packaging/tools/get_version.sh ${TD_COMMUNITY_DIR}/src/util/src/version.c
      OUTPUT_VARIABLE
      VERSION_INFO)
  MESSAGE(STATUS "build version ${VERSION_INFO}")
  SET_TARGET_PROPERTIES(tscli PROPERTIES VERSION ${VERSION_INFO} SOVERSION 1)
ENDIF ()

