# Portfolio-Personnel-
Description# Trouver Google Test
find_package(Threads REQUIRED)

add_executable(${PROJECT_NAME} TesteurReseau.cpp)

# Lien avec les bibliothèques Google Test
target_link_libraries(${PROJECT_NAME}
    gtest
    gtest_main
    Threads::Threads
)
    sudo apt update
sudo apt install libgtest-dev

