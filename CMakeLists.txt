cmake_minimum_required(VERSION 3.11.3)

set(CMAKE_CXX_STANDARD 17)

project(Membot)

# Enable debug symbols and disable optimizations for profiling
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0")

# Find wxWidgets
find_package(wxWidgets REQUIRED COMPONENTS core base)
include(${wxWidgets_USE_FILE})

# Source files
file(GLOB project_SRCS src/*.cpp)

# Executable
add_executable(membot ${project_SRCS})
target_link_libraries(membot ${wxWidgets_LIBRARIES})
target_include_directories(membot PRIVATE ${wxWidgets_INCLUDE_DIRS})

# ✅ Memory profiling target (Massif) → Output in top-level rep/
add_custom_target(
    profile-memory
    COMMAND valgrind --tool=massif --massif-out-file=${CMAKE_SOURCE_DIR}/rep/massif.out ./membot
    COMMENT "Running Valgrind Massif for memory profiling..."
)

# ✅ CPU profiling target (Callgrind) → Output in top-level rep/
add_custom_target(
    profile-cpu
    COMMAND valgrind --tool=callgrind --callgrind-out-file=${CMAKE_SOURCE_DIR}/rep/callgrind.out ./membot
    COMMENT "Running Valgrind Callgrind for CPU profiling..."
)

# ✅ Memory leak check target → Output in top-level rep/
add_custom_target(
    check-leaks
    COMMAND valgrind --leak-check=full --track-origins=yes --show-leak-kinds=all --verbose --log-file=${CMAKE_SOURCE_DIR}/rep/valgrind_report.txt ./membot
    COMMENT "Running Valgrind for memory leak detection..."
)

# ✅ Open Massif results automatically from top-level rep/
add_custom_target(
    analyze-memory
    COMMAND massif-visualizer ${CMAKE_SOURCE_DIR}/rep/massif.out
    COMMENT "Opening Massif visualizer..."
)

# ✅ Open Callgrind results automatically from top-level rep/
add_custom_target(
    analyze-cpu
    COMMAND kcachegrind ${CMAKE_SOURCE_DIR}/rep/callgrind.out
    COMMENT "Opening KCachegrind..."
)
