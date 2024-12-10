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

