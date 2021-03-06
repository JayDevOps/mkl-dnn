#===============================================================================
# Copyright 2016-2017 Intel Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#===============================================================================

if(POLICY CMP0065)
    cmake_policy(SET CMP0065 NEW)
endif()

if(UNIX OR APPLE)
    # workaround for Intel Compiler 16.0 that doesn't suppress warning on
    # deprecation with "-Wno-deprecated" compiler flag
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel" AND
            CMAKE_CXX_COMPILER_VERSION VERSION_LESS "17.0")
        set(SUPPRESS_DEPRECATE_WARNS "-diag-disable:1478")
    else()
        set(SUPPRESS_DEPRECATE_WARNS "-Wno-deprecated-declarations")
    endif()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${SUPPRESS_DEPRECATE_WARNS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${SUPPRESS_DEPRECATE_WARNS}")
endif()

#add_executable(api-check-c api_check.c)
#target_link_libraries(api-check-c ${LIB_NAME})
#add_test(api-check-c api-check-c)

# add_executable(simple_convolution-c simple_convolution.c)
# set_property(TARGET simple_convolution-c PROPERTY C_STANDARD 99)
# target_link_libraries(simple_convolution-c ${LIB_NAME})
# add_test(simple_convolution-c simple_convolution-c)

add_executable(api-c api.c)
set_property(TARGET api-c PROPERTY C_STANDARD 99)
target_link_libraries(api-c ${LIB_NAME})
add_test(api-c api-c)

if(WIN32)
    set_property(TEST api-c PROPERTY ENVIRONMENT "PATH=${CTESTCONFIG_PATH}")
    configure_file(${CMAKE_SOURCE_DIR}/config_template.vcxproj.user
        api-c.vcxproj.user @ONLY)
endif()
if(UNIX OR APPLE)
    set(test_c_symbols "${CMAKE_CURRENT_BINARY_DIR}/test_c_symbols.c")
    add_executable(test_c_symbols-c ${test_c_symbols})
    add_custom_command(
        OUTPUT ${test_c_symbols}
        COMMAND /bin/bash ${CMAKE_CURRENT_SOURCE_DIR}/generate_c_symbols_refs.sh
        ${CMAKE_CURRENT_SOURCE_DIR}/.. ${test_c_symbols}
    )
    target_link_libraries(test_c_symbols-c ${LIB_NAME})
    add_test(test_c_symbols-c test_c_symbols-c)
# elseif(WIN32)
# No Windows support for: test_c_symbols.c
endif()





add_subdirectory(gtests)

add_subdirectory(benchdnn)
