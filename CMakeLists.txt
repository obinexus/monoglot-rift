cmake_minimum_required(VERSION 3.14)
project(RIFT LANGUAGES C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Build LibRift as a static library
add_library(rift STATIC
    src/core/placeholder.c  # We'll create this
)

target_include_directories(rift PUBLIC
    ${PROJECT_SOURCE_DIR}/include
)

# Optional: Example executable
add_executable(string_pattern_detector
    examples/floyd/string_pattern_detector.c  # Your example file
)

target_link_libraries(string_pattern_detector PRIVATE rift)
# Add -lm if needed for math later: target_link_libraries(string_pattern_detector PRIVATE rift m)
