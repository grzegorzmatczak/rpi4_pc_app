 cmake_minimum_required(VERSION 3.9 FATAL_ERROR)
project(rpi-pc-app
  VERSION 0.0.1
  DESCRIPTION "rpi-pc-app"
  LANGUAGES C CXX)
  
enable_testing()

set(DEFAULT_BUILD_TYPE "Debug")
set(CMAKE_BUILD_TYPE "Debug")
set(TARGET_NAME "rpi-pc-app")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FLAGS_COMMON} -O1 " )

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)


file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

if (WIN32)
	list(APPEND CMAKE_PREFIX_PATH "E:\\repo\\opencvBuild")
	list(APPEND CMAKE_PREFIX_PATH "E:\\Qt\\5.12.8\\msvc2017_64")
endif (WIN32)


configure_file(config.json ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/config.json)

configure_file(config.json ${CMAKE_BINARY_DIR}/config.json)


set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
find_package(Qt5Core REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Qt5Test REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5Network REQUIRED)
#include fix for OpenCV
include_directories(${OpenCV_INCLUDE_DIRS})

add_subdirectory(spdlog)

set(SRC
	ConfigReader/src/configreader.cc
	Broadcaster/src/broadcaster.cc
	QtCommunicatorPlugin/src/message.cc
	QtCommunicatorPlugin/src/mqt.cc
	QtCommunicatorPlugin/src/sockio.cc
)

set(HEADERS
	ConfigReader/include/configreader.h
	Broadcaster/include/broadcaster.h
	QtCommunicatorPlugin/include/message.h
	QtCommunicatorPlugin/include/mqt.h
	QtCommunicatorPlugin/include/sockio.h

)

file(GLOB_RECURSE SRC_GLOB src/*.cc src/*.c src/*.cpp)
file(GLOB_RECURSE HEADERS_GLOB src/*.h src/*.hpp)

file(GLOB_RECURSE SRC_IMAGE_PROCESSING_GLOB ImageProcessing/*.cc ImageProcessing/*.c ImageProcessing/*.cpp)
file(GLOB_RECURSE HEADERS_IMAGE_PROCESSING_GLOB ImageProcessing/*.h ImageProcessing/*.hpp)


add_executable(${TARGET_NAME}
    ${SRC} ${HEADERS}
	${SRC_GLOB} ${HEADERS_GLOB}
	${SRC_IMAGE_PROCESSING_GLOB} ${HEADERS_IMAGE_PROCESSING_GLOB}
)

add_executable(testModules 
	#ProcessingModules/src/Test/test.cc
	${SRC} ${HEADERS}
	${SRC_GLOB} ${HEADERS_GLOB}
	${SRC_IMAGE_PROCESSING_GLOB} ${HEADERS_IMAGE_PROCESSING_GLOB}
)
	

add_test(testModules testModules)


target_link_libraries(testModules spdlog::spdlog)
target_link_libraries(testModules ${QT_LIBRARIES} ${OpenCV_LIBS} Threads::Threads)
target_link_libraries(testModules Qt5::Test)

qt5_use_modules(${PROJECT_NAME} Network Widgets Test)


if (UNIX)
	target_link_libraries(${PROJECT_NAME} spdlog::spdlog)
	target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES} ${OpenCV_LIBS} Threads::Threads Qt5::Test)
endif (UNIX)

if (WIN32)
	target_link_libraries(${PROJECT_NAME} spdlog::spdlog)
	target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES} ${OpenCV_LIBS} Threads::Threads Qt5::Test)
endif (WIN32)



message("CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE} ")
if (CMAKE_BUILD_TYPE STREQUAL "Debug" AND WIN32)

set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

endif()
if (CMAKE_BUILD_TYPE STREQUAL "Release" AND WIN32)

set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

endif()

if (WIN32)
    get_target_property(QT5_QMAKE_EXECUTABLE Qt5::qmake IMPORTED_LOCATION)
    get_filename_component(QT5_WINDEPLOYQT_EXECUTABLE ${QT5_QMAKE_EXECUTABLE} PATH)
    set(QT5_WINDEPLOYQT_EXECUTABLE "${QT5_WINDEPLOYQT_EXECUTABLE}/windeployqt.exe")

    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
       COMMAND ${QT5_WINDEPLOYQT_EXECUTABLE} --qmldir ${CMAKE_SOURCE_DIR} $<TARGET_FILE_DIR:${PROJECT_NAME}>)
endif(WIN32)

if (WIN32)
	macro(qt5_copy_dll APP DLL)
		# find the release *.dll file
		get_target_property(Qt5_${DLL}Location Qt5::${DLL} LOCATION)
		# find the debug *d.dll file
		get_target_property(Qt5_${DLL}LocationDebug Qt5::${DLL} IMPORTED_LOCATION_DEBUG)
		add_custom_command(TARGET ${APP} POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different $<$<CONFIG:Debug>:${Qt5_${DLL}LocationDebug}> $<$<NOT:$<CONFIG:Debug>>:${Qt5_${DLL}Location}> $<TARGET_FILE_DIR:${APP}>)
	endmacro()
	qt5_copy_dll( ${PROJECT_NAME} Core)

	message("CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}/bin/Release ")


	set(OPENCV_PREFIX_BUILD_TYPE "")
	message("Copy OpenCV file:" )
	if(${CMAKE_BUILD_TYPE} STREQUAL  "Release")
		set(OPENCV_PREFIX_BUILD_TYPE "")
	endif()
	if(${CMAKE_BUILD_TYPE} STREQUAL  "Debug")
		set(OPENCV_PREFIX_BUILD_TYPE "d")

	endif()
	message("Copy OpenCV file:" )
	add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_stitching450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_bgsegm450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_tracking450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_videoio450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_video450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_imgcodecs450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_imgproc450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_core450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_calib3d450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_features2d450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_flann450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_dnn450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_highgui450${OPENCV_PREFIX_BUILD_TYPE}.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/
		COMMAND ${CMAKE_COMMAND} -E copy_if_different  ${OpenCV_DIR}/bin/${CMAKE_BUILD_TYPE}/opencv_videoio_ffmpeg450_64.dll   ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/

		)


endif(WIN32)