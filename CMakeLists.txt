cmake_minimum_required(VERSION 3.5)

project(libMultiRobotPlanning)

# Without a build type nothing is optimized, which makes cbs_ta and ecbs_ta take
# minutes on a 32x32 map (all pairs shortest paths, see shortest_path_heuristic.hpp)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    Debug Release RelWithDebInfo MinSizeRel)
endif()

find_package(Boost 1.58 REQUIRED COMPONENTS program_options)
find_package(PkgConfig)
pkg_check_modules(YamlCpp yaml-cpp)

# check if Doxygen is installed
find_package(Doxygen)

set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in)
set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

if (DOXYGEN_FOUND AND EXISTS ${DOXYGEN_IN})

  # request to configure the file
  configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
  message("Doxygen build started")

  # note the option ALL which allows to build the docs together with the application
  add_custom_target( docs
    COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating API documentation with Doxygen"
    VERBATIM )
else ()
  message("Doxygen and ${DOXYGEN_IN} are needed to generate the doxygen documentation")
endif ()

# clang-tidy target (linter & static code analysis)
add_custom_target(clang-tidy
  COMMAND CMAKE_EXPORT_COMPILE_COMMANDS=ON run-clang-tidy ${CMAKE_CURRENT_SOURCE_DIR})

# clang-format
set(ALL_SOURCE_FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/a_star.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/a_star_epsilon.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/assignment.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/cbs.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/cbs_ta.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/ecbs.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/ecbs_ta.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/mcts.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/neighbor.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/next_best_assignment.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/planresult.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libMultiRobotPlanning/sipp.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/a_star.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/a_star_epsilon.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/assignment.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/cbs.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/cbs_ta.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ecbs.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/ecbs_ta.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/mapf_prioritized_sipp.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/mcts_nonoverlap.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/next_best_assignment.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shortest_path_heuristic.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/shortest_path_heuristic.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/sipp.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/timer.hpp
)

add_custom_target(clang-format
  COMMAND clang-format -i ${ALL_SOURCE_FILES}
)

add_custom_target(everything
  DEPENDS clang-format clang-tidy docs
)

# Enable C++14 and warnings
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# Creates compile database used by clang-tidy.
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include_directories(
  include
)

# Examples

## a_star
add_executable(a_star
  src/a_star.cpp
)
target_link_libraries(a_star
  ${Boost_LIBRARIES}
)

## a_star_epsilon
add_executable(a_star_epsilon
  src/a_star_epsilon.cpp
)
target_link_libraries(a_star_epsilon
  ${Boost_LIBRARIES}
)

## assignment
add_executable(assignment
  src/assignment.cpp
)
target_link_libraries(assignment
  ${Boost_LIBRARIES}
)

## cbs_roadmap
add_executable(cbs_roadmap
  src/cbs_roadmap.cpp
)
target_link_libraries(cbs_roadmap
  ${Boost_LIBRARIES}
  yaml-cpp
)

## cbs
add_executable(cbs
  src/cbs.cpp
)
target_link_libraries(cbs
  ${Boost_LIBRARIES}
  yaml-cpp
)

## cbs_ta
add_executable(cbs_ta
  src/cbs_ta.cpp
)
target_link_libraries(cbs_ta
  ${Boost_LIBRARIES}
  yaml-cpp
)

## ecbs
add_executable(ecbs
  src/ecbs.cpp
)
target_link_libraries(ecbs
  ${Boost_LIBRARIES}
  yaml-cpp
)

## ecbs_ta
add_executable(ecbs_ta
  src/ecbs_ta.cpp
)
target_link_libraries(ecbs_ta
  ${Boost_LIBRARIES}
  yaml-cpp
)

## next_best_assignment
add_executable(next_best_assignment
  src/next_best_assignment.cpp
)
target_link_libraries(next_best_assignment
  ${Boost_LIBRARIES}
  yaml-cpp
)

## sipp
add_executable(sipp
  src/sipp.cpp
)
target_link_libraries(sipp
  ${Boost_LIBRARIES}
  yaml-cpp
)

## mapf_prioritized_sipp
add_executable(mapf_prioritized_sipp
  src/mapf_prioritized_sipp.cpp
)
target_link_libraries(mapf_prioritized_sipp
  ${Boost_LIBRARIES}
  yaml-cpp
)

## mcts_nonoverlap
add_executable(mcts_nonoverlap
  src/mcts_nonoverlap.cpp
)
target_link_libraries(mcts_nonoverlap
  ${Boost_LIBRARIES}
  yaml-cpp
)
