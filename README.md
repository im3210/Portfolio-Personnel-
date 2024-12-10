# Si vous utilisez des options spécifiques ou des dépendances, ajoutez-les ici
cmake_minimum_required(VERSION 3.16)
project(TP3_algo)

# Définit le standard C++
set(CMAKE_CXX_STANDARD 17)

# Ajoutez les répertoires d'inclusion pour Google Test
include_directories(/opt/homebrew/Cellar/googletest/1.15.2/include)

# Ajoutez les répertoires contenant les bibliothèques Google Test
link_directories(/opt/homebrew/Cellar/googletest/1.15.2/lib)

# Déclarez l'exécutable principal
add_executable(TP3_algo
        ContratException.cpp
        Bottin.cpp
        Bottin.h
        TableHachage.h
        TableHachage.hpp
        main.cpp
)

# Déclarez l'exécutable pour les tests
add_executable(Testeur
        Testeur.cpp
        ContratException.cpp
        Bottin.cpp

)

# Liez Google Test au binaire des tests
target_link_libraries(Testeur
        gtest
        gtest_main
        pthread
)
