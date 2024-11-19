# Portfolio-Personnel-
Description

This is my personal resume website built with ReactJS. I am using my own React resume website template that can be found HERE.
# Trouver Google Test
find_package(GTest REQUIRED)

# Ajouter les fichiers source
add_executable(${PROJECT_NAME} TesteurReseau.cpp)

# Inclure GTest et lier les bibliothèques
target_link_libraries(${PROJECT_NAME} GTest::GTest GTest::Main pthread)
