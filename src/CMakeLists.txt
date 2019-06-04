set (LIB_NAME flogging)
file(GLOB sources ./*.f90)
include_directories(../include)
add_library ( ${LIB_NAME} ${sources})
target_include_directories(${LIB_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/../include)

set(CMAKE_WITH_MPI OFF CACHE STRING "build the module with MPI, so it can be executed using mpirun")
# if cmake provides a findLIBRARY module, this gets invoked via find_package(LIBRARY)
if (CMAKE_WITH_MPI)
        # find if there is an MPI setup on the system and if so, set corresponding variables
        find_package(MPI)
        if (NOT ${MPI_Fortran_FOUND})
                message(FATAL_ERROR "MPI required but not found")
        else()
                message(STATUS "found MPI_Fortran_COMPILER ${MPI_Fortran_COMPILER}")
                # set the preprocessor
                target_compile_definitions(${LIB_NAME} PRIVATE "-DUSE_MPI")
				# add the link to mpi if existing
                target_link_libraries(${LIB_NAME} ${MPI_Fortran_LIBRARIES} )
                target_include_directories(${LIB_NAME} PUBLIC ${MPI_Fortran_INCLUDE_PATH} )
        endif()
endif()
# add some compiler flags based on ${LIB_NAME} module
set(CMAKE_Fortran_FLAGS_DEBUG "${CMAKE_Fortran_FLAGS_DEBUG} -DENABLE_LOG_TRACE")
set(CMAKE_Fortran_FLAGS_RELEASE "${CMAKE_Fortran_FLAGS_DEBUG} -DDISABLE_LOG_DEBUG")

# ifort and gfortran need the flag -cpp to interpret definitions like -DMRM2MHM
# the nag compiler is not able to interpret the flag -cpp but can interpret these definitions anyway
# so we check whether the compiler is able to use the flag -cpp
# for that we need the module CheckFortranCompilerFlag
include(CheckFortranCompilerFlag)
CHECK_Fortran_COMPILER_FLAG("-cpp" CPP_FLAG)
# if the flag exists, we add it to the compilation flags
if (CPP_FLAG)
	set(ADDITIONAL_GCC_FLAGS "-cpp")
endif()

if(CMAKE_Fortran_COMPILER_ID MATCHES "GNU")
	set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -ffree-line-length-none")
endif()

set_target_properties ( ${LIB_NAME}
  PROPERTIES
  COMPILE_FLAGS "${CMAKE_Fortran_FLAGS} ${MPI_Fortran_COMPILE_FLAGS} ${ADDITIONAL_GCC_FLAGS}"
  LINK_FLAGS " ${MPI_Fortran_LINK_FLAGS}"
)