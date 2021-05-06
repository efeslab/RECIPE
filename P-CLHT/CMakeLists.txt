cmake_minimum_required(VERSION 2.8)
project(P-CLHT)

if( NOT CMAKE_BUILD_TYPE )
    message(STATUS "No build type selected, default to Release")
    set( CMAKE_BUILD_TYPE Release)
else()
    message(STATUS "Build type is set to ${CMAKE_BUILD_TYPE}")
endif()

# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -march=native -mrtm -mcx16 -mavx -mavx2 -mbmi2 -mlzcnt -Wno-deprecated-declarations -Wall -Wextra -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -faligned-new=64 -DNDEBUG")

# To use wllvm                                                                 
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -std=c++17 -march=skylake -fheinous-gnu-extensions -Wno-deprecated-declarations -Wall -Wextra -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -faligned-new=64 -DNDEBUG")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0 -fheinous-gnu-extensions")

execute_process(COMMAND cat /proc/cpuinfo COMMAND grep clflush OUTPUT_VARIABLE ENABLE_CLFLUSH)
execute_process(COMMAND cat /proc/cpuinfo COMMAND grep clflushopt OUTPUT_VARIABLE ENABLE_CLFLUSHOPT)
execute_process(COMMAND cat /proc/cpuinfo COMMAND grep clwb OUTPUT_VARIABLE ENABLE_CLWB)

if(ENABLE_CLWB)
    add_definitions(-DCLWB)
    message(STATUS "Looking for clwb instruction - found")
elseif(ENABLE_CLFLUSHOPT)
    add_definitions(-DCLFLUSH_OPT)
    message(STATUS "Looking for clwb instruction - not found")
    message(STATUS "Looking for clflushopt instruction - found")
elseif(ENABLE_CLFLUSH)
    add_definitions(-DCLFLUSH)
    message(STATUS "Looking for clwb instruction - not found")
    message(STATUS "Looking for clflushopt instruction - not found")
    message(STATUS "Looking for clflush instruction - found")
else()
    message(FATAL_ERROR "Cannot find any flush instructions (clflush, clflushopt, clwb)")
endif()

add_definitions(-DADD_PADDING)

include_directories(include external/include)

find_library(JemallocLib jemalloc)
find_library(TbbLib tbb)
find_package(Threads)

set(LOCAL_SOURCE src/clht_lb_res.c src/clht_gc.c external/sspfd/sspfd.c external/ssmem/src/ssmem.c)
set(LOCAL_LIBS ${TbbLib} ${JemallocLib} boost_system boost_thread pthread pmemobj pmem)
if (PMDK_LIBS)
    set(LOCAL_SOURCE ${LOCAL_SOURCE})
endif()

set(P_CLHT_TEST example.cpp ${LOCAL_SOURCE})
# add_executable(p-clht_example ${P_CLHT_TEST})
add_pmdk_target(TARGET p-clht_example SOURCES ${P_CLHT_TEST} VERSION "1.8" 
                EXTRA_LIBS ${TbbLib} ${JemallocLib} boost_system boost_thread pthread)

# target_link_libraries(p-clht_example ${LOCAL_LIBS})

# add_executable(p-clht_test ${P_CLHT_TEST})
# add_pmdk_target(TARGET p-clht_test SOURCES ${P_CLHT_TEST} VERSION "1.9.2")

# target_link_libraries(example ${TbbLib} ${JemallocLib} pmemobj pmem boost_system
#     boost_thread pthread)

# set(P_CLHT_TEST test.cpp src/clht_lb_res.c src/clht_gc.c
#     external/sspfd/sspfd.c external/ssmem/src/ssmem.c)
# add_executable(test ${P_CLHT_TEST})

# target_link_libraries(test ${TbbLib} ${JemallocLib} pmemobj pmem boost_system
#     boost_thread pthread)
