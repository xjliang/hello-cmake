cmake_minimum_required(VERSION 3.2)
project(divider_tests)

#add_subdirectory(lib/googletest)
#add_subdirectory(${HELLO_GOOGLETEST_DIR})

include_directories(${DIVISION_HEADERS_DIR})
#include_directories(lib/googletest/googletest/include)
include_directories(${HELLO_GOOGLETEST_DIR}/include)


set(SOURCE_FILES main.cc
				 src/divider_tests.cc
)

add_executable(divider_tests ${SOURCE_FILES})
target_link_libraries(divider_tests division gtest)
install(TARGETS divider_tests DESTINATION bin)
