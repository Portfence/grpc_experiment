cmake_minimum_required(VERSION 3.10)

# Project name
project("gRPC_Node_1" CXX)

set(CMAKE_CXX_STANDARD 17)

# name of project executable
set(PROJECT_EXEC run_server_executable)

# Set output folder where program will be created
set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/bin)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)

# define directory structure
set(PROJECT_TEST_DIR ${CMAKE_SOURCE_DIR}/unit_tests)
set(PROJECT_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/include)
set(PROJECT_PROTO_DIR "${CMAKE_SOURCE_DIR}/../protos")
set(PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/src) # point to source code instead

# Tell cmake which packages to find
find_package(Boost
	COMPONENTS
		unit_test_framework
		system
	REQUIRED
)

set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(protobuf CONFIG
	REQUIRED
)
message(STATUS "Using protobuf ${protobuf_VERSION}")

find_package(gRPC CONFIG
	REQUIRED
)
message(STATUS "Using gRPC ${gRPC_VERSION}")

# ######################## PROTOBUF and GRPC ###################################
get_target_property(gRPC_CPP_PLUGIN_EXECUTABLE gRPC::grpc_cpp_plugin
    IMPORTED_LOCATION_RELEASE)

# create variables with output locations
set(PROJECT_PROTO_SRC "${CMAKE_CURRENT_BINARY_DIR}/Service.pb.cc")
set(PROJECT_PROTO_INCLUDE "${CMAKE_CURRENT_BINARY_DIR}/Service.pb.h")
set(PROJECT_GRPC_SRC "${CMAKE_CURRENT_BINARY_DIR}/Service.grpc.pb.cc")
set(PROJECT_GRPC_INCLUDE "${CMAKE_CURRENT_BINARY_DIR}/Service.grpc.pb.h")

# create variable containing proto files
set(PROJECT_PROTO_FILES "${PROJECT_PROTO_DIR}/Service.proto")

# generate grpc services and proto messages from .protofile
# (not generated without reference in add_executable)
add_custom_command(
    OUTPUT ${PROJECT_PROTO_SRC}	${PROJECT_PROTO_INCLUDE}
					 ${PROJECT_GRPC_SRC}  ${PROJECT_GRPC_INCLUDE}
    COMMAND protobuf::protoc
    ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}"
				 --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
				  -I "${PROJECT_PROTO_DIR}"
				 --plugin=protoc-gen-grpc="${gRPC_CPP_PLUGIN_EXECUTABLE}"
					"${PROJECT_PROTO_FILES}"
)

# ############################ Create binaries ################################
add_executable(${PROJECT_EXEC}
		${PROJECT_SOURCE_DIR}/main.cpp
		${PROJECT_SOURCE_DIR}/server_implementation.cpp
		${PROJECT_PROTO_SRC}
		${PROJECT_PROTO_INCLUDE}
		${PROJECT_GRPC_SRC}
		${PROJECT_GRPC_INCLUDE}
)

# include directories for target
target_include_directories(${PROJECT_EXEC}
	PRIVATE
		${PROJECT_INCLUDE_DIR}
    ${GRPC_INCLUDE_DIR}	
    ${Protobuf_INCLUDE_DIR}
)

# Links requied libraries
target_link_libraries(${PROJECT_EXEC}
   PRIVATE	
		gRPC::grpc++
		${Protobuf_LIBRARIES}
)

# ########################## Create tests #####################################
# create executable for performing unit tests
#add_definitions(-DBOOST_TEST_DYN_LINK)

#set(CLASS_TEST_EXEC run_server_implementation_tests)
#add_executable(${CLASS_TEST_EXEC}
#		${PROJECT_TEST_DIR}/server_implementation_tests.cpp
#		${PROJECT_SOURCE_DIR}/server_implementation.cpp
#)
#
#target_include_directories(${CLASS_TEST_EXEC}
#	PRIVATE
#		${PROJECT_INCLUDE_DIR}
#		${GRPC_INCLUDE_DIR}
#		${Protobuf_INCLUDE_DIR}
#		${BOOST_INCLUDE_DIR}
#)
#
#target_link_libraries(${CLASS_TEST_EXEC}
#	PRIVATE
#		gRPC::grpc++
#		${Protobuf_LIBRARIES}
#		${Boost_LIBRARIES}
#)
#
#add_test(
#	NAME
#		ServerImplementationTest
#	COMMAND
#		${CLASS_TEST_EXEC}
#)
# #############################################################################
