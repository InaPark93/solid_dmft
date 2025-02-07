# all other tests
set(basic_tests
    test_afm_mapping
    test_dict_to_h5
    test_interaction_hamiltonian
    test_manipulate_chemical_potential
    test_observables
    test_postproc_toml_dict
    test_postproc_toml_dict_integration
    test_convergence
    test_matheval
    test_plot_correlated_bands
    test_respack_sfo
    )

  # copy input files for PCB test from documentation tutorial
  set (test_dir ${CMAKE_CURRENT_BINARY_DIR})
  foreach(file svo.wout svo_example.h5 svo_hr.dat)
    FILE(COPY ../../doc/tutorials/correlated_bandstructure/${file} DESTINATION ${test_dir})
  endforeach()

  # copy reference data for PCB test
  FILE(COPY test_pcb_ref.h5 DESTINATION ${test_dir})

  # copy reference data for respack test
  FILE(COPY respack_sfo_data DESTINATION ${test_dir})

  foreach(test ${basic_tests})
  get_filename_component(test_name ${test} NAME_WE)
  get_filename_component(test_dir ${test} DIRECTORY)
  add_test(NAME ${test_name} COMMAND ${TRIQS_PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/${test_dir}/${test_name}.py WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${test_dir})
  set_property(TEST ${test_name} APPEND PROPERTY ENVIRONMENT PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH})
endforeach()

# ------------------------------#

# integration tests
set (integration_tests
     svo_hubbardI_basic
     svo_hartree
     lno_hubbardI_mag
     svo_cthyb_basic_crm
     svo_cthyb_basic_tf
     lco_ftps
     nio_cthyb_hartree
     )

# copy other test files
FILE(COPY UIJKL DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

foreach(test ${integration_tests})
  set (test_dir ${CMAKE_CURRENT_BINARY_DIR}/${test})

  foreach(file dmft_config.toml inp.h5 ref.h5 test.py)
    FILE(COPY ${test}/${file} DESTINATION ${test_dir})
  endforeach()

  add_test(NAME ${test}
           # COMMAND bash ${test}.sh
           COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} ${MPIEXEC_PREFLAGS} ${TRIQS_PYTHON_EXECUTABLE} test.py
           # COMMAND ${TRIQS_PYTHON_EXECUTABLE} test.py
           WORKING_DIRECTORY ${test_dir}
          )

  set_property(TEST ${test} APPEND PROPERTY ENVIRONMENT PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH})
endforeach()

# ------------------------------#

# MaxEnt test on svo_hubbardI_basic
set (test svo_hubbardI_basic)

foreach(file test_maxent.py helper.py)
    FILE(COPY ${file} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
endforeach()

add_test(NAME maxent_${test}
         COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} ${MPIEXEC_PREFLAGS} ${TRIQS_PYTHON_EXECUTABLE} test_maxent.py
         WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        )

set_property(TEST maxent_${test} APPEND PROPERTY ENVIRONMENT PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH})
set_property(TEST maxent_${test} APPEND PROPERTY DEPENDS "svo_hubbardI_basic")

# ------------------------------#

# integration test for restarting svo_hartree
set (restart_integration_test
     svo_hubbardI_basic)

foreach(test ${restart_integration_test})
  set (test_dir ${CMAKE_CURRENT_BINARY_DIR}/${test})

  foreach(file ref_restart.h5 test_restart.py)
    FILE(COPY ${test}/${file} DESTINATION ${test_dir})
  endforeach()

  add_test(NAME restart_${test}
           COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} ${MPIEXEC_PREFLAGS} ${TRIQS_PYTHON_EXECUTABLE} test_restart.py
           WORKING_DIRECTORY ${test_dir}
          )

  set_property(TEST restart_${test} APPEND PROPERTY ENVIRONMENT PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH})
endforeach()
set_property(TEST maxent_${test} APPEND PROPERTY DEPENDS "svo_hubbardI_basic")

# ------------------------------#

# GW embedding tests
option(Test_GW_embedding "Run GW embedding tests" OFF)
if(Test_GW_embedding)

set(gw_emb_basic_tests
    test_gw_embedding
    )

foreach(test ${gw_emb_basic_tests})
  get_filename_component(test_name ${test} NAME_WE)
  get_filename_component(test_dir ${test} DIRECTORY)
  add_test(NAME ${test_name} COMMAND ${TRIQS_PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/${test_dir}/${test_name}.py WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${test_dir})
  set_property(TEST ${test_name} APPEND PROPERTY ENVIRONMENT PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH})
endforeach()

set (gw_emb_integration_tests
     svo_cthyb_crpa
     svo_ctseg_dyn
     svo_gw_emb_stat
     # svo_gw_emb_dyn
     )

foreach(test ${gw_emb_integration_tests})
  set (test_dir ${CMAKE_CURRENT_BINARY_DIR}/${test})

  foreach(file dmft_config.toml inp.h5 ref.h5 test.py)
    FILE(COPY ${test}/${file} DESTINATION ${test_dir})
  endforeach()

  add_test(NAME ${test}
           # COMMAND bash ${test}.sh
           COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} ${MPIEXEC_PREFLAGS} ${TRIQS_PYTHON_EXECUTABLE} test.py
           # COMMAND ${TRIQS_PYTHON_EXECUTABLE} test.py
           WORKING_DIRECTORY ${test_dir}
          )

  set_property(TEST ${test} APPEND PROPERTY ENVIRONMENT PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH})
endforeach()

endif()


