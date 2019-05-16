cmake_minimum_required(VERSION 3.1)

set(FFTW_VERSION 3.3.8)

get_cmake_property(vars CACHE_VARIABLES)
foreach(var ${vars})
  get_property(currentHelpString CACHE "${var}" PROPERTY HELPSTRING)
    if("${currentHelpString}" MATCHES "No help, variable specified on the command line." OR "${currentHelpString}" STREQUAL "")
        # message("${var} = [${${var}}]  --  ${currentHelpString}") # uncomment to see the variables being processed
        list(APPEND CL_ARGS "-D${var}=${${var}}")
    endif()
endforeach()

include(ExternalProject)

project(csdr)

add_library(lib${PROJECT_NAME} SHARED fft_fftw.c libcsdr_wrapper.c)

add_definitions(-DLIBCSDR_GPL)
add_definitions(-DUSE_IMA_ADPCM)
add_definitions(-DUSE_FFTW)

ExternalProject_Add(project_fftw
  URL http://www.fftw.org/fftw-${FFTW_VERSION}.tar.gz
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}/fftw-${FFTW_VERSION}
  BUILD_IN_SOURCE 1
  CMAKE_ARGS  ${CL_ARGS} -DENABLE_FLOAT=1 -DDISABLE_FORTRAN=1 -DBUILD_SHARED_LIBS=0
  INSTALL_COMMAND ""
)

ExternalProject_Get_Property(project_fftw source_dir)

include_directories(${source_dir}/api)
add_library(fftw STATIC IMPORTED)
set_property(TARGET fftw PROPERTY IMPORTED_LOCATION ${source_dir}/libfftw3f.a)
add_dependencies(fftw project_fftw)

target_link_libraries(lib${PROJECT_NAME} fftw)
target_link_libraries(lib${PROJECT_NAME} m)
install(TARGETS lib${PROJECT_NAME} DESTINATION lib)
install(FILES lib${PROJECT_NAME}.h DESTINATION include)

option(BUILD_CSDR "Build csdr command line tool" ON)
if(${BUILD_CSDR})
    add_executable(${PROJECT_NAME} csdr.c)
    target_link_libraries(${PROJECT_NAME} lib${PROJECT_NAME})
    install(TARGETS ${PROJECT_NAME} DESTINATION bin)
endif()

option(BUILD_NMUX "Build nmux command line tool" ON)
if(${BUILD_NMUX})
    add_executable(nmux nmux.cpp tsmpool.cpp)
    target_link_libraries(nmux lib${PROJECT_NAME})
    target_link_libraries(nmux pthread)
    install(TARGETS nmux DESTINATION bin)
endif()

option(BUILD_DCDD "Build ddcd command line tool" OFF)
if(${BUILD_DCDD})
    add_executable(ddcd ddcd.cpp)
    target_link_libraries(ddcd lib${PROJECT_NAME})
    target_link_libraries(ddcd pthread)
    install(TARGETS ddcd DESTINATION bin)
endif()
