if (CHIP_BUILD_HIPBLAS)
    add_hip_test(hipBlas-sgemm.cpp)
    target_link_libraries(hipBlas-sgemm hipblas)
endif()