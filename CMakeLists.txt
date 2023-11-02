find_package(Poco COMPONENTS Foundation REQUIRED)

add_subdirectory(libs/aruco-1.3.0)

set(OFX_ARUCO_SOURCES
  src/ofxAruco.h
  src/ofxAruco.cpp
)
add_library(ofxAruco STATIC ${OFX_ARUCO_SOURCES})
add_library(ofx::Svg ALIAS ofxAruco)
target_include_directories(ofxAruco PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)
target_link_libraries(ofxAruco PUBLIC openFrameworks aruco ofx::Cv Poco::Foundation)


set(OFX_ARUCO_EXAMPLE_SOURCES
  example/src/main.cpp
  example/src/ofApp.cpp
  example/src/ofApp.h
)
add_executable(ofxArucoExample ${OFX_ARUCO_EXAMPLE_SOURCES})
target_link_libraries(ofxArucoExample PRIVATE ofxAruco)
file(COPY example/bin/data/ DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
