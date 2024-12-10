#include <gtest/gtest.h>
#include "Bottin.h"
#include <fstream>

// Fixture pour initialiser un objet Bottin à partir d'un fichier
class BottinTest : public ::testing::Test {
protected:
    void SetUp() override {
        std::ifstream fichier("../bottin.txt");
        if (!fichier) {
            std::cerr << "Erreur d'ouverture du fichier!" << std::endl;
            throw std::runtime_error("Fichier introuvable ou illisible.");
        }
        bottin = new TP3::Bottin(fichier);
        fichier.close();
    }

    void TearDown() override {
        delete bottin;
    }

    TP3::Bottin* bottin;
};

// Test pour recherche par téléphone (entrée valide)
TEST_F(BottinTest, TestTrouverAvecTelephoneValide) {
    const auto& entree = bottin->trouverAvecTelephone("(530) 752-3457");
    EXPECT_EQ(entree.m_nom, "Adams Jr");
    EXPECT_EQ(entree.m_prenom, "Theodore E");
    EXPECT_EQ(entree.m_telephoneFixe, "(530) 752-3457");
    EXPECT_EQ(entree.m_cellulaire, "(530) 752-4361");
    EXPECT_EQ(entree.m_courriel, "tjadams@ucdavis.edu");
}

// Test pour recherche par téléphone (entrée invalide)
TEST_F(BottinTest, TestTrouverAvecTelephoneInvalide) {
    EXPECT_THROW(bottin->trouverAvecTelephone("(514) 000-0000"), std::invalid_argument);
}

// Test pour recherche par nom/prénom (entrée valide)
TEST_F(BottinTest, TestTrouverAvecNomPrenomValide) {
    const auto& entree = bottin->trouverAvecNomPrenom("Abbott", "Ursula K");
    EXPECT_EQ(entree.m_nom, "Abbott");
    EXPECT_EQ(entree.m_prenom, "Ursula K");
    EXPECT_EQ(entree.m_telephoneFixe, "(530) 752-7325");
    EXPECT_EQ(entree.m_cellulaire, "(530) 752-8960");
    EXPECT_EQ(entree.m_courriel, "sandra.alexander@ucop.edu");
}

// Test pour recherche par nom/prénom (entrée invalide)
TEST_F(BottinTest, TestTrouverAvecNomPrenomInvalide) {
    EXPECT_THROW(bottin->trouverAvecNomPrenom("Brown", "Bob"), std::invalid_argument);
}

// Test pour afficher le bottin
TEST_F(BottinTest, TestAfficherBottin) {
    std::ostringstream oss;
    bottin->afficherBottin(oss); // Génère le contenu du bottin

    // Charger tout le contenu attendu depuis le fichier bottin.txt
    std::ifstream fichier("../bottin.txt");
    ASSERT_TRUE(fichier.is_open()) << "Erreur : Impossible d'ouvrir le fichier bottin.txt";

    // Ignorer la première ligne (le nombre d'entrées)
    std::string ligne;
    std::getline(fichier, ligne);

    // Lire tout le contenu restant
    std::string contenuAttendu((std::istreambuf_iterator<char>(fichier)),
                                std::istreambuf_iterator<char>());
    fichier.close();

    // Comparer les contenus
    EXPECT_EQ(oss.str(), contenuAttendu) << "Le contenu affiché du bottin ne correspond pas au contenu attendu.";
}



// Test pour vérifier le nombre d'entrées
TEST_F(BottinTest, TestNombreEntrees) {
    EXPECT_EQ(bottin->nombreEntrees(), 1438);
}

// Test pour le ratio de collisions noms/prénoms
TEST_F(BottinTest, TestRatioCollisionsNomPrenom) {
    double ratio = bottin->ratioDeCollisionsNomPrenom();
    EXPECT_GE(ratio, 0.0);
}

// Test pour le ratio de collisions téléphones
TEST_F(BottinTest, TestRatioCollisionsTelephone) {
    double ratio = bottin->ratioDeCollisionTelephone();
    EXPECT_GE(ratio, 0.0);
}

// Test pour le maximum de collisions noms/prénoms
TEST_F(BottinTest, TestMaxCollisionsNomPrenom) {
    int maxCollisions = bottin->maximumNbCollisionNomPrenom();
    EXPECT_GE(maxCollisions, 0);
}

// Test pour le maximum de collisions téléphones
TEST_F(BottinTest, TestMaxCollisionsTelephone) {
    int maxCollisions = bottin->maximumNbCollisionTelephone();
    EXPECT_GE(maxCollisions, 0);
}

// Main pour exécuter les tests
int main(int argc, char** argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
