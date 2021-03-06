project(SMHasher)

cmake_minimum_required(VERSION 2.4)

set(CMAKE_BUILD_TYPE Release)
#set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

IF(CMAKE_SYSTEM_NAME MATCHES "Linux")
   EXEC_PROGRAM(cat ARGS "/proc/cpuinfo" OUTPUT_VARIABLE CPUINFO)
   STRING(REGEX REPLACE "^.*(sse2).*$" "\\1" SSE_THERE ${CPUINFO})
   STRING(COMPARE EQUAL "sse2" "${SSE_THERE}" SSE2_TRUE)
   STRING(REGEX REPLACE "^.*(sse4_2).*$" "\\1" SSE_THERE ${CPUINFO})
   STRING(COMPARE EQUAL "sse4_2" "${SSE_THERE}" SSE42_TRUE)
ELSEIF(CMAKE_SYSTEM_NAME MATCHES "Darwin")
   EXEC_PROGRAM("/usr/sbin/sysctl -n machdep.cpu.features" OUTPUT_VARIABLE
      CPUINFO)
   STRING(REGEX REPLACE "^.*[^S](SSE2).*$" "\\1" SSE_THERE ${CPUINFO})
   STRING(COMPARE EQUAL "SSE2" "${SSE_THERE}" SSE2_TRUE)
   STRING(REGEX REPLACE "^.*(SSE4.2).*$" "\\1" SSE_THERE ${CPUINFO})
   STRING(COMPARE EQUAL "SSE4.2" "${SSE_THERE}" SSE42_TRUE)
ENDIF(CMAKE_SYSTEM_NAME MATCHES "Linux")

IF (SSE2_TRUE)
   set(SSE2_FOUND true CACHE BOOL "SSE2 available")
ELSE (SSE2_TRUE)
   set(SSE2_FOUND false CACHE BOOL "SSE2 not available")
ENDIF (SSE2_TRUE)
IF (SSE42_TRUE)
   set(SSE4_2_FOUND true CACHE BOOL "SSE4.2 available")
ELSE (SSE42_TRUE)
  set(SSE4_2_FOUND false CACHE BOOL "SSE4.2 not available")
ENDIF (SSE42_TRUE)

IF(SSE4_2_FOUND)
  set(CMAKE_C_FLAGS   "-msse2 -msse4.2")
  set(CMAKE_CXX_FLAGS "-msse2 -msse4.2")
  set(CMAKE_EXE_LINKER_FLAGS "-lpthread")
  set(SSE2_SRC "hasshe2.c")
  set(SSE4_SRC "crc32_hw.c")
  set(SSE4_SRC1 "crc32_hw1.c")
  set(SIPHASH_SRC "siphash_ssse3.c")
ELSEIF(SSE2_FOUND)
  set(CMAKE_C_FLAGS   "-msse2")
  set(CMAKE_CXX_FLAGS "-msse2")
  set(SSE2_SRC "hasshe2.c")
  set(SIPHASH_SRC "siphash_sse2.c")
ELSE()
  set(SIPHASH_SRC "siphash.c")
ENDIF(SSE4_2_FOUND)

add_library(
  SMHasherSupport
  AvalancheTest.cpp
  Bitslice.cpp
  Bitvec.cpp
  CityTest.cpp
  City.cpp
  crc.cpp
  DifferentialTest.cpp
  Hashes.cpp
  ${SSE2_SRC}
  ${SSE4_SRC} 
  ${SSE4_SRC1} 
  KeysetTest.cpp
  lookup3.cpp
  md5.cpp
  MurmurHash1.cpp
  MurmurHash2.cpp
  MurmurHash3.cpp
  Platform.cpp
  Random.cpp
  sha1.cpp
  ${SIPHASH_SRC}
  SpeedTest.cpp
  Spooky.cpp
  SpookyTest.cpp
  Stats.cpp
  SuperFastHash.cpp
  Types.cpp
  PMurHash.c
)

add_executable(
  SMHasher
  main.cpp
)

target_link_libraries(
  SMHasher
  SMHasherSupport
)
