CMAKE_MINIMUM_REQUIRED(VERSION 3.1)
PROJECT(dqrobotics-interface-vrep)
set (CMAKE_CXX_STANDARD 17)

option(WITH_PCL_OPTIMIZATION_FLAGS "Use optimization flags used by PCL to compile a compatible interface to EIGEN types" ON)
set(PCL_OPTIMIZATION_FLAGS)
if (${WITH_PCL_OPTIMIZATION_FLAGS})
    find_package(PCL REQUIRED COMPONENTS common)
    get_target_property(OUTPUT pcl_common INTERFACE_COMPILE_OPTIONS)
    set(PCL_OPTIMIZATION_FLAGS ${OUTPUT})
    message(STATUS "PCL optimization flags = ${PCL_OPTIMIZATION_FLAGS}")
endif ()

if(APPLE)
    INCLUDE_DIRECTORIES(
        /usr/local/include/
        /usr/local/include/eigen3
        # Most recent versions of brew install here
        /opt/homebrew/include
        /opt/homebrew/include/eigen3)
    ADD_COMPILE_OPTIONS(-Werror=return-type -Wall -Wextra -Wmissing-declarations -Wredundant-decls -Woverloaded-virtual)
    # The library is installed here when using the regular cmake ., make, sudo make install
    LINK_DIRECTORIES(
        /usr/local/lib/)
endif()

if(WIN32)
    ADD_DEFINITIONS(-D_USE_MATH_DEFINES)
    FIND_PACKAGE(Eigen3 CONFIG REQUIRED)
    INCLUDE_DIRECTORIES(${EIGEN3_INCLUDE_DIR})
endif()

add_definitions(-DNON_MATLAB_PARSING)
add_definitions(-DMAX_EXT_API_CONNECTIONS=255)
add_definitions(-DDO_NOT_USE_SHARED_MEMORY)

INCLUDE_DIRECTORIES(${PROJECT_NAME} include)
INCLUDE_DIRECTORIES(${PROJECT_NAME} coppeliarobotics/remoteApi)
INCLUDE_DIRECTORIES(${PROJECT_NAME} coppeliarobotics/include)

################################################################
# DEFINE AND INSTALL LIBRARY AND INCLUDE FOLDER
################################################################

ADD_LIBRARY(${PROJECT_NAME} SHARED 
    src/dqrobotics/interfaces/vrep/DQ_VrepInterface.cpp
    src/dqrobotics/interfaces/vrep/DQ_VrepInterfaceMapElement.cpp
    src/dqrobotics/interfaces/vrep/DQ_VrepRobot.cpp
    src/dqrobotics/interfaces/vrep/DQ_SerialVrepRobot.cpp
    src/dqrobotics/interfaces/vrep/robots/LBR4pVrepRobot.cpp
    src/dqrobotics/interfaces/vrep/robots/YouBotVrepRobot.cpp
    src/dqrobotics/interfaces/vrep/robots/FrankaEmikaPandaVrepRobot.cpp
    coppeliarobotics/remoteApi/extApi.c
    coppeliarobotics/remoteApi/extApiPlatform.c
    )

SET_TARGET_PROPERTIES(${PROJECT_NAME} 
    PROPERTIES PUBLIC_HEADER
    "include/dqrobotics/interfaces/vrep/DQ_VrepInterface.h"
    )

INSTALL(TARGETS ${PROJECT_NAME} 
    LIBRARY DESTINATION "lib"
    PUBLIC_HEADER DESTINATION "include/dqrobotics/interfaces/vrep"
    PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)

# This does not seem to be required on Ubuntu
if(APPLE)
    TARGET_LINK_LIBRARIES(${PROJECT_NAME}
        -ldqrobotics)
endif()

target_compile_options(${PROJECT_NAME} PUBLIC ${PCL_OPTIMIZATION_FLAGS})

################################################################
# INSTALL HEADERS IN SUBFOLDERS
################################################################

# Other Headers
INSTALL(FILES
    include/dqrobotics/interfaces/vrep/DQ_VrepRobot.h
    include/dqrobotics/interfaces/vrep/DQ_SerialVrepRobot.h
    include/dqrobotics/interfaces/vrep/DQ_VrepInterfaceMapElement.h
    DESTINATION "include/dqrobotics/interfaces/vrep")

# Robots
INSTALL(FILES
    include/dqrobotics/interfaces/vrep/robots/LBR4pVrepRobot.h
    include/dqrobotics/interfaces/vrep/robots/YouBotVrepRobot.h
    include/dqrobotics/interfaces/vrep/robots/FrankaEmikaPandaVrepRobot.h
    DESTINATION "include/dqrobotics/interfaces/vrep/robots")

################################################################
# INSTALL SOURCE FILES (So that the debugger can find them)
################################################################

# base folder
INSTALL(FILES 
    src/dqrobotics/interfaces/vrep/DQ_VrepInterface.cpp
    src/dqrobotics/interfaces/vrep/DQ_VrepInterfaceMapElement.cpp
    src/dqrobotics/interfaces/vrep/DQ_VrepRobot.cpp
    src/dqrobotics/interfaces/vrep/DQ_SerialVrepRobot.cpp
    DESTINATION "src/dqrobotics/interfaces/vrep")

# robots folder
INSTALL(FILES
    src/dqrobotics/interfaces/vrep/robots/LBR4pVrepRobot.cpp
    src/dqrobotics/interfaces/vrep/robots/YouBotVrepRobot.cpp
    src/dqrobotics/interfaces/vrep/robots/FrankaEmikaPandaVrepRobot.cpp
    DESTINATION "src/dqrobotics/interfaces/vrep/robots")
