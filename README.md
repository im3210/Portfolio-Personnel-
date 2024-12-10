#include <iostream>
#include <fstream>
#include <stdexcept>
#include <string>
#include "Bottin.h"

int main() {
    try {
        // Charger le fichier de données
        std::ifstream fichier("../bottin.txt");
        if (!fichier) {
            std::cerr << "Erreur : Impossible d'ouvrir le fichier bottin.txt." << std::endl;
            return 1;
        }

        // Initialiser le bottin
        TP3::Bottin bottin(fichier);
        fichier.close();

        // Afficher les statistiques
        std::cout << "Bottin chargé avec succès !\n";
        std::cout << "Nombre d'entrées dans le bottin : " << bottin.nombreEntrees() << "\n";
        std::cout << "Ratio de collisions (nom/prénom) : " << bottin.ratioDeCollisionsNomPrenom() << "\n";
        std::cout << "Ratio de collisions (téléphone) : " << bottin.ratioDeCollisionTelephone() << "\n";
        std::cout << "Max collisions (nom/prénom) : " << bottin.maximumNbCollisionNomPrenom() << "\n";
        std::cout << "Max collisions (téléphone) : " << bottin.maximumNbCollisionTelephone() << "\n";

        // Recherche d'entrées par l'utilisateur
        std::string choix;
        do {
            std::cout << "\n--- Menu Recherche ---\n";
            std::cout << "1. Rechercher par nom et prénom\n";
            std::cout << "2. Rechercher par numéro de téléphone fixe\n";
            std::cout << "3. Quitter\n";
            std::cout << "Votre choix : ";
            std::cin >> choix;

            if (choix == "1") {
                std::string nom, prenom;
                std::cout << "Entrez le nom : ";
                std::cin.ignore();
                std::getline(std::cin, nom);
                std::cout << "Entrez le prénom : ";
                std::getline(std::cin, prenom);
                try {
                    const auto& entree = bottin.trouverAvecNomPrenom(nom, prenom);
                    std::cout << "Entrée trouvée : " << entree.m_nom << ", " << entree.m_prenom
                              << "\t" << entree.m_telephoneFixe << "\t" << entree.m_cellulaire
                              << "\t" << entree.m_courriel << "\n";
                } catch (const std::exception& e) {
                    std::cerr << "Erreur : " << e.what() << "\n";
                }
            } else if (choix == "2") {
                std::string telephone;
                std::cout << "Entrez un numéro de téléphone fixe : ";
                std::cin.ignore();
                std::getline(std::cin, telephone);
                try {
                    const auto& entree = bottin.trouverAvecTelephone(telephone);
                    std::cout << "Entrée trouvée : " << entree.m_nom << ", " << entree.m_prenom
                              << "\t" << entree.m_telephoneFixe << "\t" << entree.m_cellulaire
                              << "\t" << entree.m_courriel << "\n";
                } catch (const std::exception& e) {
                    std::cerr << "Erreur : " << e.what() << "\n";
                }
            } else if (choix != "3") {
                std::cerr << "Choix invalide. Veuillez réessayer.\n";
            }

        } while (choix != "3");

        std::cout << "Au revoir !\n";

    } catch (const std::exception& e) {
        std::cerr << "Erreur fatale : " << e.what() << std::endl;
        return 1;
    }

    return 0;
}

