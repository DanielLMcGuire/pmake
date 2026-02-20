add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/pmake.hpp
    COMMAND $<TARGET_FILE:phasor_cpp_compiler> ${CMAKE_CURRENT_SOURCE_DIR}pmake.phs -o ${CMAKE_CURRENT_BINARY_DIR}pmake.hpp -H --nologo
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}pmake.phs phasor_cpp_compiler
    COMMENT "Building PHS bytecode pmake.hpp"
)

add_subdirectory(executable)