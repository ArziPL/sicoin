cmake_minimum_required(VERSION 3.25)

project(sicoin
    VERSION 0.1.0
    DESCRIPTION "Currency converter based on NBP exchange rates"
    LANGUAGES CXX
)

# --- C++ standard ------------------------------------------------------------
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# --- Qt ----------------------------------------------------------------------
find_package(Qt6 REQUIRED COMPONENTS Widgets Network)
qt_standard_project_setup()   # enables AUTOMOC/AUTOUIC, sane Qt defaults

# --- Application -------------------------------------------------------------
qt_add_executable(sicoin
    src/main.cpp
)

target_link_libraries(sicoin PRIVATE
    Qt6::Widgets
    Qt6::Network
)

target_compile_options(sicoin PRIVATE -Wall -Wextra -Wpedantic)
