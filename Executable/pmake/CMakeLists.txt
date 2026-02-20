add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/pmake.hpp
    COMMAND $<TARGET_FILE:phasor_cpp_compiler> ${CMAKE_CURRENT_SOURCE_DIR}/../../pmake.phs -o ${CMAKE_CURRENT_BINARY_DIR}/pmake.hpp -H --nologo
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/../../pmake.phs phasor_cpp_compiler
    COMMENT "Building PHS bytecode pmake.hpp"
)

add_executable(pmake
    pmake.rc
    pmake_main.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/pmake.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/../../pmake.phs
)
target_include_directories(pmake PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(pmake phasor_native_runtime_static)
set_target_properties(pmake PROPERTIES EXCLUDE_FROM_ALL TRUE)