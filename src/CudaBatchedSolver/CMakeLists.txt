set(CUDASOLVER_SOURCES solve.cu inverse.cu)
set(CUDASOLVER_HEADERS solve.h inverse.h)

CUDA_ADD_LIBRARY(CudaBatchedSolver ${CUDASOLVER_SOURCES} ${CUDASOLVER_HEADERS})

set_target_properties(CudaBatchedSolver PROPERTIES PUBLIC_HEADER "${CUDASOLVER_HEADERS}")

set_target_properties(CudaBatchedSolver
        PROPERTIES SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
        VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")

# Targets:
#   * <prefix>/lib/libCudaBatchedSolver.a
#   * header location after install: <prefix>/include/
install(TARGETS CudaBatchedSolver
        EXPORT "${targets_export_name}"
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        PUBLIC_HEADER DESTINATION "${include_install_dir}/CudaBatchedSolver"
        INCLUDES DESTINATION "${include_install_dir}/CudaBatchedSolver")