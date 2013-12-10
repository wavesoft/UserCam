cmake_minimum_required (VERSION 2.6)
project ( usercam )

# Prepare constants
set( LIB_DIR "${PROJECT_SOURCE_DIR}/lib" )
set( BOOST_LIBS thread system filesystem regex )
set( BOOST_VERSION "1.45.0" )

# Find OpenGL Libraries
find_package(GLUT)
find_package(OpenGL)
if (OPENGL_FOUND)
	include_directories( ${OPENGL_INCLUDE_DIR} )
else (OPENGL_FOUND)

	# Try to find OpenGL Libraries from lib folder
	file( GLOB OPENGL_LIBRARIES 
		"${LIB_DIR}/opengl/lib/${LIB}/*.lib"
		"${LIB_DIR}/opengl/lib/${LIB}/*.a" )

	# Check if something was found
	if("${OPENGL_LIBRARIES}" STREQUAL "")
		message(SEND_ERROR "Could not find OpenGL on your system")
	endif()
	message("Using OpenGL library from lib/ folder")

	# Try to find GLUT Libraries from lib folder
	file( GLOB GLUT_LIBRARIES 
		"${LIB_DIR}/glut/lib/${LIB}/*.lib"
		"${LIB_DIR}/glut/lib/${LIB}/*.a" )

	if("${GLUT_LIBRARIES}" STREQUAL "")
		message(SEND_ERROR "Could not find GLUT on your system")
	endif()
	message("Using GLUT library from lib/ folder")

	# Add include folders
	set( OPENGL_INCLUDE_DIR "${LIB_DIR}/opengl/include" )
	set( GLUT_INCLUDE_DIR "${LIB_DIR}/glut/include" )
	include_directories( ${OPENGL_INCLUDE_DIR} )
	include_directories( ${GLUT_INCLUDE_DIR} )

endif (OPENGL_FOUND)

# Check if we can find OpenSSL
FIND_PACKAGE(OpenSSL)
IF (OPENSSL_FOUND)
	include_directories( ${OPENSSL_INCLUDE_DIRS} )
else()

	# Try to find OpenGL Libraries from lib folder
	file( GLOB OPENSSL_LIBRARIES 
		"${LIB_DIR}/openssl/lib/${LIB}/*.lib"
		"${LIB_DIR}/openssl/lib/${LIB}/*.a" )

	# Check if something was found
	if("${OPENSSL_LIBRARIES}" STREQUAL "")
		message(SEND_ERROR "Could not find OpenSSL on your system")
	endif()
	message("Using OpenSSL library from lib/ folder")

	# Add include folders
	set( OPENSSL_INCLUDE_DIRS "${LIB_DIR}/opengl/include" )
	include_directories( ${OPENSSL_INCLUDE_DIRS} )

ENDIF(OPENSSL_FOUND)

# Check if we can find BOOST
#set(Boost_USE_STATIC_LIBS OFF) 
#set(Boost_USE_MULTITHREADED ON)  
#set(Boost_USE_STATIC_RUNTIME OFF)
ADD_DEFINITIONS(-DBOOST_ALL_NO_LIB)
find_package(Boost ${BOOST_VERSION} COMPONENTS ${BOOST_LIBS} )
if(Boost_FOUND)
    include_directories( ${Boost_INCLUDE_DIRS} ) 
else(Boost_FOUND)
	message("Looking for BOOST library in lib/ folder")

	# Populate BOOST_LIBS
	set( Boost_LIBRARIES "" )
	set( Boost_INCLUDE_DIRS "" )
	foreach (LIB ${BOOST_LIBS})

		# Find libs 
		file( GLOB BOOST_LIB_FILES 
			"${LIB_DIR}/boost/${LIB}/lib/*.lib"
			"${LIB_DIR}/boost/${LIB}/lib/*.a" )

		# Check for missing files
		if("${BOOST_LIB_FILES}" STREQUAL "")
			message(SEND_ERROR "Could not find BOOST library '${LIB}' on your system")
		endif()
		message("Using BOOST library '${LIB}' from lib/ folder")

		# Update Boost variables
		set( Boost_LIBRARIES ${Boost_LIBRARIES} ${BOOST_LIB_FILES} )
		set( Boost_INCLUDE_DIRS ${Boost_INCLUDE_DIRS} "${LIB_DIR}/boost/${LIB}/include" )

	endforeach (LIB)

	# Include libraries
	include_directories( ${Boost_INCLUDE_DIRS} )

endif(Boost_FOUND)

# Additional includes
include_directories( ${PROJECT_SOURCE_DIR}/include )

# Locate the source files
file ( GLOB SOURCES_CPP ${PROJECT_SOURCE_DIR}/src/*.cpp  )

# Sources
add_executable( ${PROJECT_NAME} ${SOURCES_CPP} )

# Libraries
target_link_libraries( ${PROJECT_NAME} ${OPENGL_LIBRARIES} )
target_link_libraries( ${PROJECT_NAME} ${GLUT_LIBRARIES} )
target_link_libraries( ${PROJECT_NAME} ${OPENSSL_LIBRARIES} )
target_link_libraries( ${PROJECT_NAME} ${Boost_LIBRARIES} )

