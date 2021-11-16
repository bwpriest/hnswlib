cmake_minimum_required (VERSION 3.14)
project(hnswlib
        VERSION   0.5.2 
        LANGUAGES CXX)



add_library(hnswlib INTERFACE)
add_library(hnswlib::hnswlib ALIAS hnswlib)
target_compile_features(hnswlib INTERFACE cxx_std_11)
target_include_directories(hnswlib INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/hnswlib>
    $<INSTALL_INTERFACE:hnswlib>
) 

if (HNSWLIB_INSTALL)
    include(GNUInstallDirs)
    include(CMakePackageConfigHelpers)

    set(HNSWLIB_EXPORT_TARGETS hnswlib)

    install(TARGETS             ${HNSWLIB_EXPORT_TARGETS}
            EXPORT              ${PROJECT_NAME}Targets
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

    install(EXPORT      ${PROJECT_NAME}Targets
            FILE        ${PROJECT_NAME}Targets.cmake
            NAMESPACE   ${PROJECT_NAME}::
            DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

    install(DIRECTORY   ${PROJECT_SOURCE_DIR}/hnswlib
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/hnswlib)

    # create version file
    write_basic_package_version_file(
        "${PROJECT_NAME}ConfigVersion.cmake"
        VERSION       ${PROJECT_VERSION}
        COMPATIBILITY ExactVersion)

    # create config file
    configure_package_config_file(
        "${PROJECT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

    install(FILES
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

endif()

if (JUST_INSTALL_HNSWLIB)
    return()
endif()

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    set(CMAKE_CXX_STANDARD 11)

    if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
      SET( CMAKE_CXX_FLAGS  "-Ofast -DNDEBUG -std=c++11 -DHAVE_CXX0X -openmp -march=native -fpic -ftree-vectorize")
    elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      SET( CMAKE_CXX_FLAGS  "-Ofast -lrt -DNDEBUG -std=c++11 -DHAVE_CXX0X -march=native -fpic -w -fopenmp -ftree-vectorize -ftree-vectorizer-verbose=0" )
    elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
      SET( CMAKE_CXX_FLAGS  "-Ofast -lrt -DNDEBUG -std=c++11 -DHAVE_CXX0X -openmp -march=native -fpic -w -fopenmp -ftree-vectorize" )
    endif()

    add_executable(test_updates examples/updates_test.cpp)
    target_link_libraries(test_updates hnswlib)

    add_executable(searchKnnCloserFirst_test examples/searchKnnCloserFirst_test.cpp)
    target_link_libraries(searchKnnCloserFirst_test hnswlib)

    add_executable(main main.cpp sift_1b.cpp)
    target_link_libraries(main hnswlib)
endif()
