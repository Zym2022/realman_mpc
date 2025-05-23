cmake_minimum_required(VERSION 3.25)
project(ocs2_core)

find_package(ament_cmake REQUIRED)
find_package(ocs2_thirdparty REQUIRED)

find_package(Boost REQUIRED COMPONENTS
  system
  filesystem
  log_setup
  log
)
find_package(Eigen3 3.3 REQUIRED NO_MODULE)


# pthread and OpenMp
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package(Threads REQUIRED)
if (Threads_FOUND) # Rename for ament_cmake
  set(Threads_INCLUDE_DIRS ${THREADS_PTHREADS_INCLUDE_DIR})
  set(Threads_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})
endif (Threads_FOUND)
find_package(OpenMP REQUIRED)

# Load ocs2 compile flags
include(cmake/ocs2_cxx_flags.cmake)
message(STATUS "OCS2_CXX_FLAGS: " ${OCS2_CXX_FLAGS})

###########
## Build ##
###########

include_directories(
  include
  test/include
  ${EIGEN3_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

set(dependencies
  ocs2_thirdparty
  Boost
  Eigen3
  OpenMP_CXX
  Threads
)

# Declare a C++ library
add_library(${PROJECT_NAME}
  src/Types.cpp
  src/augmented_lagrangian/AugmentedLagrangian.cpp
  src/augmented_lagrangian/StateAugmentedLagrangian.cpp
  src/augmented_lagrangian/StateInputAugmentedLagrangian.cpp
  src/augmented_lagrangian/StateAugmentedLagrangianCollection.cpp
  src/augmented_lagrangian/StateInputAugmentedLagrangianCollection.cpp
  src/automatic_differentation/CppAdInterface.cpp
  src/automatic_differentation/CppAdSparsity.cpp
  src/automatic_differentation/FiniteDifferenceMethods.cpp
  src/constraint/StateConstraintCppAd.cpp
  src/constraint/StateInputConstraintCppAd.cpp
  src/constraint/StateConstraintCollection.cpp
  src/constraint/StateInputConstraintCollection.cpp
  src/constraint/LinearStateConstraint.cpp
  src/constraint/LinearStateInputConstraint.cpp
  src/control/FeedforwardController.cpp
  src/control/LinearController.cpp
  src/control/StateBasedLinearController.cpp
  src/cost/QuadraticStateCost.cpp
  src/cost/QuadraticStateInputCost.cpp
  src/cost/StateCostCollection.cpp
  src/cost/StateCostCppAd.cpp
  src/cost/StateInputCostCollection.cpp
  src/cost/StateInputCostCppAd.cpp
  src/cost/StateInputGaussNewtonCostAd.cpp
  src/dynamics/ControlledSystemBase.cpp
  src/dynamics/LinearSystemDynamics.cpp
  src/dynamics/SystemDynamicsBase.cpp
  src/dynamics/SystemDynamicsBaseAD.cpp
  src/dynamics/SystemDynamicsLinearizer.cpp
  src/dynamics/TransferFunctionBase.cpp
  src/integration/SensitivityIntegrator.cpp
  src/integration/SensitivityIntegratorImpl.cpp
  src/integration/Integrator.cpp
  src/integration/IntegratorBase.cpp
  src/integration/RungeKuttaDormandPrince5.cpp
  src/integration/OdeBase.cpp
  src/integration/Observer.cpp
  src/integration/StateTriggeredEventHandler.cpp
  src/integration/SystemEventHandler.cpp
  src/reference/ModeSchedule.cpp
  src/reference/TargetTrajectories.cpp
  src/loopshaping/LoopshapingDefinition.cpp
  src/loopshaping/LoopshapingPropertyTree.cpp
  src/loopshaping/LoopshapingFilter.cpp
  src/loopshaping/LoopshapingPreComputation.cpp
  src/loopshaping/cost/LoopshapingCost.cpp
  src/loopshaping/cost/LoopshapingStateCost.cpp
  src/loopshaping/cost/LoopshapingStateInputCost.cpp
  src/loopshaping/cost/LoopshapingCostEliminatePattern.cpp
  src/loopshaping/cost/LoopshapingCostOutputPattern.cpp
  src/loopshaping/soft_constraint/LoopshapingSoftConstraint.cpp
  src/loopshaping/soft_constraint/LoopshapingStateInputSoftConstraint.cpp
  src/loopshaping/soft_constraint/LoopshapingSoftConstraintEliminatePattern.cpp
  src/loopshaping/soft_constraint/LoopshapingSoftConstraintOutputPattern.cpp
  src/loopshaping/augmented_lagrangian/LoopshapingAugmentedLagrangian.cpp
  src/loopshaping/augmented_lagrangian/LoopshapingStateAugmentedLagrangian.cpp
  src/loopshaping/augmented_lagrangian/LoopshapingStateInputAugmentedLagrangian.cpp
  src/loopshaping/augmented_lagrangian/LoopshapingAugmentedLagrangianEliminatePattern.cpp
  src/loopshaping/augmented_lagrangian/LoopshapingAugmentedLagrangianOutputPattern.cpp
  src/loopshaping/constraint/LoopshapingConstraint.cpp
  src/loopshaping/constraint/LoopshapingStateConstraint.cpp
  src/loopshaping/constraint/LoopshapingStateInputConstraint.cpp
  src/loopshaping/constraint/LoopshapingConstraintEliminatePattern.cpp
  src/loopshaping/constraint/LoopshapingConstraintOutputPattern.cpp
  src/loopshaping/dynamics/LoopshapingDynamics.cpp
  src/loopshaping/dynamics/LoopshapingDynamicsEliminatePattern.cpp
  src/loopshaping/dynamics/LoopshapingDynamicsOutputPattern.cpp
  src/loopshaping/dynamics/LoopshapingFilterDynamics.cpp
  src/loopshaping/initialization/LoopshapingInitializer.cpp
  src/model_data/ModelData.cpp
  src/model_data/Metrics.cpp
  src/model_data/Multiplier.cpp
  src/misc/LinearAlgebra.cpp
  src/misc/Log.cpp
  src/soft_constraint/StateSoftConstraint.cpp
  src/soft_constraint/StateInputSoftConstraint.cpp
  src/soft_constraint/StateInputSoftBoxConstraint.cpp
  src/penalties/MultidimensionalPenalty.cpp
  src/penalties/Penalties.cpp
  src/penalties/penalties/RelaxedBarrierPenalty.cpp
  src/penalties/penalties/SquaredHingePenalty.cpp
  src/thread_support/ThreadPool.cpp
)
ament_target_dependencies(${PROJECT_NAME}
    ${dependencies}
)
target_compile_options(${PROJECT_NAME} PUBLIC ${OCS2_CXX_FLAGS})

find_library(DL_LIBRARY dl)
if(DL_LIBRARY)
  message(STATUS "Found dl library: ${DL_LIBRARY}") 
  target_link_libraries(${PROJECT_NAME} ${DL_LIBRARY})
else()
  message(WARNING "Could not find dl library")
endif()

add_executable(${PROJECT_NAME}_lintTarget
  src/lintTarget.cpp
)
ament_target_dependencies(${PROJECT_NAME}_lintTarget
    ${dependencies}
)
target_link_libraries(${PROJECT_NAME}_lintTarget
  ${PROJECT_NAME}
)

#########################
###   CLANG TOOLING   ###
#########################
find_package(cmake_clang_tools QUIET)
if(cmake_clang_tools_FOUND)
  message(STATUS "Run clang tooling for target " ${PROJECT_NAME}_lintTarget)
  add_clang_tooling(
    TARGETS
      ${PROJECT_NAME}
      ${PROJECT_NAME}_lintTarget
    SOURCE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/include
    CT_HEADER_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
    CF_WERROR
  )
endif(cmake_clang_tools_FOUND)

#############
## Install ##
#############

install(
  TARGETS ocs2_core
  EXPORT export_ocs2_core
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include/ocs2_core
)

install(DIRECTORY include/ DESTINATION include/${PROJECT_NAME})

install(DIRECTORY test/include/
  DESTINATION include/${PROJECT_NAME}
)

#############
## Testing ##
#############
find_package(ament_lint_auto REQUIRED)
ament_lint_auto_find_test_dependencies()
find_package(ament_cmake_gtest REQUIRED)

ament_add_gtest(test_control
  test/control/testLinearController.cpp
  test/control/testFeedforwardController.cpp
)
target_link_libraries(test_control ${PROJECT_NAME})
ament_target_dependencies(test_control ${dependencies})

ament_add_gtest(initialization_unittest
  test/initialization/InitializationTest.cpp
)
target_link_libraries(initialization_unittest ${PROJECT_NAME})
ament_target_dependencies(initialization_unittest ${dependencies})

ament_add_gtest(test_integration
  test/integration/testSensitivityIntegrator.cpp
  test/integration/IntegrationTest.cpp
  test/integration/testRungeKuttaDormandPrince5.cpp
  test/integration/TrapezoidalIntegrationTest.cpp
)
target_link_libraries(test_integration ${PROJECT_NAME})
ament_target_dependencies(test_integration ${dependencies})

ament_add_gtest(interpolation_unittest
  test/misc/testInterpolation.cpp
)
target_link_libraries(interpolation_unittest ${PROJECT_NAME})
ament_target_dependencies(interpolation_unittest ${dependencies})

ament_add_gtest(${PROJECT_NAME}_cppadcg
  test/cppad_cg/testCppADCG_dynamics.cpp
  test/cppad_cg/testSparsityHelpers.cpp
  test/cppad_cg/testCppAdInterface.cpp
)
target_link_libraries(${PROJECT_NAME}_cppadcg ${PROJECT_NAME})
ament_target_dependencies(${PROJECT_NAME}_cppadcg ${dependencies})

ament_add_gtest(test_transferfunctionbase
  test/dynamics/testTransferfunctionBase.cpp
)
target_link_libraries(test_transferfunctionbase ${PROJECT_NAME})
ament_target_dependencies(test_transferfunctionbase ${dependencies})

ament_add_gtest(${PROJECT_NAME}_loopshaping
  test/loopshaping/testLoopshapingConfiguration.cpp
  test/loopshaping/testLoopshapingAugmentedLagrangian.cpp
  test/loopshaping/testLoopshapingConstraint.cpp
  test/loopshaping/testLoopshapingCost.cpp
  test/loopshaping/testLoopshapingSoftConstraint.cpp
  test/loopshaping/testLoopshapingDefinition.cpp
  test/loopshaping/testLoopshapingDynamics.cpp
  test/loopshaping/testLoopshapingFilterDynamics.cpp
  test/loopshaping/testLoopshapingPreComputation.cpp
)
target_link_libraries(${PROJECT_NAME}_loopshaping ${PROJECT_NAME} ${Boost_LIBRARIES} -lstdc++fs)
ament_target_dependencies(${PROJECT_NAME}_loopshaping ${dependencies} Boost)

ament_add_gtest(${PROJECT_NAME}_test_misc
  test/misc/testInterpolation.cpp
  test/misc/testLinearAlgebra.cpp
  test/misc/testLogging.cpp
  test/misc/testLoadData.cpp
  test/misc/testLookup.cpp
)
target_link_libraries(${PROJECT_NAME}_test_misc
  ${PROJECT_NAME}
)
ament_target_dependencies(${PROJECT_NAME}_test_misc ${dependencies})

ament_add_gtest(test_dynamics
  test/dynamics/testSystemDynamicsLinearizer.cpp
  test/dynamics/testSystemDynamicsPreComputation.cpp
)
target_link_libraries(test_dynamics
  ${PROJECT_NAME}
)
ament_target_dependencies(test_dynamics ${dependencies})

ament_add_gtest(test_cost
  test/cost/testCostCollection.cpp
  test/cost/testCostCppAd.cpp
  test/cost/testQuadraticCostFunction.cpp
)
target_link_libraries(test_cost
  ${PROJECT_NAME}
)
ament_target_dependencies(test_cost ${dependencies})

ament_add_gtest(test_constraint
  test/constraint/testConstraintCollection.cpp
  test/constraint/testConstraintCppAd.cpp
  test/constraint/testLinearConstraint.cpp
)
target_link_libraries(test_constraint
  ${PROJECT_NAME}
)
ament_target_dependencies(test_constraint ${dependencies})

ament_add_gtest(test_metrics
  test/model_data/testMetrics.cpp
)
target_link_libraries(test_metrics
  ${PROJECT_NAME}
)
ament_target_dependencies(test_metrics ${dependencies})

ament_add_gtest(test_multiplier
  test/model_data/testMultiplier.cpp
)
target_link_libraries(test_multiplier
  ${PROJECT_NAME}
)
ament_target_dependencies(test_multiplier ${dependencies})

ament_add_gtest(test_ModelData
  test/model_data/testModelData.cpp
)
target_link_libraries(test_ModelData
  ${PROJECT_NAME}
)
ament_target_dependencies(test_ModelData ${dependencies})

ament_add_gtest(test_ModeSchedule
  test/reference/testModeSchedule.cpp
)
target_link_libraries(test_ModeSchedule
    ${PROJECT_NAME}
)
ament_target_dependencies(test_ModeSchedule ${dependencies})

ament_add_gtest(test_softConstraint
  test/soft_constraint/testSoftConstraint.cpp
  test/soft_constraint/testDoubleSidedPenalty.cpp
)
target_link_libraries(test_softConstraint
  ${PROJECT_NAME}
)
ament_target_dependencies(test_softConstraint ${dependencies})

ament_add_gtest(${PROJECT_NAME}_test_thread_support
  test/thread_support/testBufferedValue.cpp
  test/thread_support/testSynchronized.cpp
  test/thread_support/testThreadPool.cpp
)
target_link_libraries(${PROJECT_NAME}_test_thread_support
  ${PROJECT_NAME}
)
ament_target_dependencies(${PROJECT_NAME}_test_thread_support ${dependencies})

ament_add_gtest(${PROJECT_NAME}_test_core
  test/testPrecomputation.cpp
  test/testTypes.cpp
  )
target_link_libraries(${PROJECT_NAME}_test_core
  ${PROJECT_NAME}
)
ament_target_dependencies(${PROJECT_NAME}_test_core ${dependencies})

ament_export_dependencies(${dependencies})  
ament_export_include_directories("include/${PROJECT_NAME}")
ament_export_targets(export_ocs2_core HAS_LIBRARY_TARGET)
ament_package(CONFIG_EXTRAS "cmake/ocs2_cxx_flags.cmake")