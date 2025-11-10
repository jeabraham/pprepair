# prepair
#
# Copyright © 2009-2022,
# Ken Arroyo Ohori    k.ohori@tudelft.nl
# Hugo Ledoux         h.ledoux@tudelft.nl
# Martijn Meijers     b.m.meijers@tudelft.nl
# All rights reserved.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.

cmake_minimum_required(VERSION 3.5...3.29)
cmake_policy(VERSION 3.5)

project(pprepair)


if(POLICY CMP0071)
  cmake_policy(SET CMP0071 NEW)
endif()

# Silence Boost policy warning on recent CMake
if(POLICY CMP0167)
  cmake_policy(SET CMP0167 OLD)
endif()

set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE)
set(CMAKE_BUILD_TYPE "Release")


# CGAL
find_package(CGAL REQUIRED)

if ( NOT CGAL_FOUND )
  message(SEND_ERROR "pprepair requires the CGAL library")
  return()
endif()

# Modern CGAL no longer defines CGAL_USE_FILE.
# Instead, link against CGAL::CGAL and enable include dirs automatically.
if (TARGET CGAL::CGAL)
  message(STATUS "Using modern CGAL targets")
else()
  message(WARNING "CGAL::CGAL target not found; using legacy include pattern if available")
  if (EXISTS "${CGAL_USE_FILE}")
    include(${CGAL_USE_FILE})
  endif()
endif()


# Boost
find_package( Boost 1.50.0 REQUIRED )

if ( NOT Boost_FOUND )
  message(SEND_ERROR "pprepair requires the Boost library version 1.50 or higher")
  return()
endif()

INCLUDE_DIRECTORIES( ${Boost_INCLUDE_DIR} )

# GDAL
find_package( GDAL 3.0 )

if ( NOT GDAL_FOUND )
  message(SEND_ERROR "pprepair requires the GDAL library")
endif()

include_directories( ${GDAL_INCLUDE_DIR} )

# Creating entries for target: pprepair
# ############################

add_executable( pprepair  FaceInfo.cpp IOWorker.cpp PlanarPartition.cpp PolygonHandle.cpp pprepair.cpp )

set_property(TARGET pprepair PROPERTY CXX_STANDARD 17)

add_to_cached_list( CGAL_EXECUTABLE_TARGETS pprepair )

# Modern CGAL integration (replaces deprecated CGAL_USE_FILE)
if (TARGET CGAL::CGAL)
  message(STATUS "Using modern CGAL targets")
  target_link_libraries(pprepair PRIVATE CGAL::CGAL ${GDAL_LIBRARY} ${Boost_LIBRARIES})
else()
  message(WARNING "Modern CGAL targets not found; falling back to legacy linkage")
  target_include_directories(pprepair PRIVATE ${CGAL_INCLUDE_DIRS})
  target_link_libraries(pprepair PRIVATE ${CGAL_LIBRARIES} ${GDAL_LIBRARY} ${Boost_LIBRARIES})
endif()

install(TARGETS pprepair DESTINATION bin)