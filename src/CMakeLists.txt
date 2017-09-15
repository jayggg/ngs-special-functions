project(ngs_special_functions)

cmake_minimum_required(VERSION 3.1)

find_package(NGSolve CONFIG REQUIRED)

find_library(F2C_LIBRARY f2c REQUIRED HINTS ${NGSOLVE_LIBRARY_DIR})
find_program(F2C_COMMAND f2c REQUIRED HINTS ${NGSOLVE_LIBRARY_DIR})

macro( fetch_and_convert_slatec_sources slatec_function )
    file(DOWNLOAD http://www.netlib.org/cgi-bin/netlibfiles.tgz?format=tgz&filename=slatec%2Fsrc%2F${slatec_function}.f ${CMAKE_CURRENT_BINARY_DIR}/${slatec_function}.tgz)
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar xvf ${CMAKE_CURRENT_BINARY_DIR}/${slatec_function}.tgz WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endmacro( fetch_and_convert_slatec_sources )

fetch_and_convert_slatec_sources(zbesi)
fetch_and_convert_slatec_sources(gamln)

file(GLOB SLATEC_SOURCES_FORTRAN ${CMAKE_CURRENT_BINARY_DIR}/slatec/src/*.f )
execute_process(COMMAND ${F2C_COMMAND} ${SLATEC_SOURCES_FORTRAN} WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/slatec/src)
file(GLOB SLATEC_SOURCES_C ${CMAKE_CURRENT_BINARY_DIR}/slatec/src/*.c )

add_ngsolve_python_module(special_functions
  specialcf.cpp ${SLATEC_SOURCES_C}
)

target_link_libraries(special_functions ${F2C_LIBRARY})

install(TARGETS special_functions DESTINATION ngsolve)