## A library that contains the C model and support code.

add_library(riscv_model
    riscv_callbacks.cpp
    riscv_callbacks.h
    riscv_config.h
    riscv_platform.cpp
    riscv_platform.h
    riscv_platform_impl.cpp
    riscv_platform_impl.h
    riscv_prelude.cpp
    riscv_prelude.h
    riscv_sail.h
    riscv_softfloat.c
    riscv_softfloat.h
    config_utils.h
    config_utils.cpp
    "${CMAKE_BINARY_DIR}/sail_riscv_model.c"
    "${CMAKE_BINARY_DIR}/sail_riscv_model.h"
)

target_include_directories(riscv_model
    PUBLIC
        # So the generated C can find riscv_platform/prelude.h"
        "${CMAKE_CURRENT_SOURCE_DIR}"
        # So `riscv_sail.h` can find `sail_riscv_model.h`.
        "${CMAKE_BINARY_DIR}"
)

target_link_libraries(riscv_model
    PUBLIC softfloat sail_runtime
)

add_dependencies(riscv_model generated_sail_riscv_model)

## A binary that can load an ELF file into the model and run it.

add_executable(sail_riscv_sim
    "riscv_sim.cpp"
    "rvfi_dii.h"
    "rvfi_dii.cpp"
)

# The generated C model is not warnings-clean, silence them.
set(_generated_c_warning_opt_out
    -Wno-extra
    -Wno-unused
    -Wno-uninitialized
)
set_source_files_properties(
    "${CMAKE_BINARY_DIR}/sail_riscv_model.c"
    "${CMAKE_BINARY_DIR}/sail_riscv_model.h"
    PROPERTIES COMPILE_OPTIONS "${_generated_c_warning_opt_out}"
)

add_dependencies(sail_riscv_sim generated_sail_riscv_model)

target_link_libraries(sail_riscv_sim
    PRIVATE riscv_model default_config
)

# TODO: Enable warnings when we use the #include trick
# to include the generated Sail code. Currently it
# generates too many warnings to turn these on globally.

# target_compile_options(sail_riscv_sim PRIVATE
#     -Wall -Wextra
#     # Too annoying at the moment.
#     -Wno-unused-parameter
# )

# Enable Sail coverage collection.
if (COVERAGE)
    target_compile_definitions(sail_riscv_sim
        PRIVATE "SAILCOV"
    )
endif()

install(TARGETS sail_riscv_sim
    OPTIONAL
    RUNTIME DESTINATION "bin"
)
