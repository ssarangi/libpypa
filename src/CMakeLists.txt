include(CheckCXXCompilerFlag)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${libpypa_SOURCE_DIR}/modules/")

if (NOT MSVC)
	find_package(GMP REQUIRED)
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
if (MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
else()
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -W -Wall -Werror -pedantic -Wno-unused-parameter -Wno-unused-const-variable")
endif()

check_cxx_compiler_flag(-Wno-unused-local-typedefs CXX_NO_UNUSED_LOCAL_TYPEDEFS)
if(CXX_NO_UNUSED_LOCAL_TYPEDEFS)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-local-typedefs")
endif()

set(CLANG_FLAGS "-Qunused-arguments -fcolor-diagnostics" CACHE STRING "Clang specific C and CXX flags")
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CLANG_FLAGS}")
endif()

add_subdirectory(double-conversion)

add_library(pypa pypa/ast/ast.cc
                 pypa/ast/dump.cc
                 pypa/filebuf.cc
                 pypa/lexer/lexer.cc
                 pypa/parser/parser.cc
                 pypa/parser/make_string.cc
                 pypa/parser/symbol_table.cc)

# lexer_test
add_executable(lexer-test EXCLUDE_FROM_ALL pypa/parser/test.cc)
add_dependencies(lexer-test pypa)

if(MSVC)
	target_link_libraries(lexer-test pypa double-conversion)	
else()
	target_link_libraries(lexer-test pypa gmp double-conversion)
endif()

# parser_test
add_executable(parser-test EXCLUDE_FROM_ALL pypa/parser/test.cc)
add_dependencies(parser-test pypa)

if(MSVC)
	target_link_libraries(parser-test pypa double-conversion)
else()
	target_link_libraries(parser-test pypa gmp double-conversion)
endif()

# install
install(TARGETS pypa ARCHIVE DESTINATION lib)
install(DIRECTORY pypa DESTINATION include FILES_MATCHING PATTERN "*.hh" PATTERN "*.inl")
