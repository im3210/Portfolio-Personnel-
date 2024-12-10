#include "Bottin.h"
#include <sstream>
#include <iostream>
#include <stdexcept>

namespace TP3 {

Bottin::Bottin(std::ifstream& p_fichierEntree, size_t p_table_size)
    : m_tableParTelephone(p_table_size), m_tableParNomPrenom(p_table_size) {
    PRECONDITION(p_fichierEntree.is_open());

    std::string ligne;

    if (!std::getline(p_fichierEntree, ligne)) {
        throw PreconditionException(__FILE__, __LINE__, "Le fichier est vide ou mal formaté.");
    }

    int nombrePersonnes;
    try {
        nombrePersonnes = std::stoi(ligne);
    } catch (...) {
        throw PreconditionException(__FILE__, __LINE__, "La première ligne ne contient pas un entier valide.");
    }

    for (int i = 0; i < nombrePersonnes; ++i) {
        if (!std::getline(p_fichierEntree, ligne)) {
            throw PreconditionException(__FILE__, __LINE__, "Le fichier contient moins d'entrées que prévu.");
        }

        std::istringstream fluxLigne(ligne);
        std::string nom_prenom, telephoneFixe, telephoneCellulaire, courriel;

        if (!(std::getline(fluxLigne, nom_prenom, '\t') &&
              std::getline(fluxLigne, telephoneFixe, '\t') &&
              std::getline(fluxLigne, telephoneCellulaire, '\t') &&
              std::getline(fluxLigne, courriel, '\t'))) {
            throw PreconditionException(__FILE__, __LINE__, "Une ligne du fichier est mal formatée.");
        }

        size_t posVirgule = nom_prenom.find(", ");
        if (posVirgule == std::string::npos) {
            throw PreconditionException(__FILE__, __LINE__, "Le format du champ 'Nom, Prenom' est incorrect.");
        }
        std::string nom = nom_prenom.substr(0, posVirgule);
        std::string prenom = nom_prenom.substr(posVirgule + 2);

        ajouter(nom, prenom, telephoneFixe, telephoneCellulaire, courriel);
    }

    double ratioCollisionsNomPrenom, ratioCollisionsTelephone;
    int totalCollisionsNomPrenom, maxCollisionsNomPrenom;
    int totalCollisionsTelephone, maxCollisionsTelephone;

    m_tableParNomPrenom.statistiques(ratioCollisionsNomPrenom, totalCollisionsNomPrenom, maxCollisionsNomPrenom);
    m_tableParTelephone.statistiques(ratioCollisionsTelephone, totalCollisionsTelephone, maxCollisionsTelephone);

    std::cout << "Statistiques des collisions :\n";
    std::cout << "- Table des noms/prénoms : ratio = " << ratioCollisionsNomPrenom
              << ", collisions totales = " << totalCollisionsNomPrenom
              << ", max par insertion = " << maxCollisionsNomPrenom << '\n';
    std::cout << "- Table des téléphones : ratio = " << ratioCollisionsTelephone
              << ", collisions totales = " << totalCollisionsTelephone
              << ", max par insertion = " << maxCollisionsTelephone << '\n';

    POSTCONDITION(nombreEntrees() == nombrePersonnes);
}



    void Bottin::ajouter(const std::string& p_nom, const std::string& p_prenom,
                         const std::string& p_telephoneFixe, const std::string& p_cellulaire,
                         const std::string& p_courriel) {
    PRECONDITION(!p_nom.empty());
    PRECONDITION(!p_prenom.empty());
    PRECONDITION(!p_telephoneFixe.empty());
    PRECONDITION(!p_cellulaire.empty());
    PRECONDITION(!p_courriel.empty());
    PRECONDITION(m_tableParTelephone.contient(p_telephoneFixe) == false);
    PRECONDITION(m_tableParNomPrenom.contient(p_nom + p_prenom) == false);

    m_tableauDesEntrees.emplace_back(p_nom, p_prenom, p_telephoneFixe, p_cellulaire, p_courriel);
    int index = m_tableauDesEntrees.size() - 1;

    m_tableParTelephone.inserer(p_telephoneFixe, index);
    m_tableParNomPrenom.inserer(p_nom + p_prenom, index);

    POSTCONDITION(m_tableParTelephone.contient(p_telephoneFixe));
    POSTCONDITION(m_tableParNomPrenom.contient(p_nom + p_prenom));
}

    const Bottin::Entree& Bottin::trouverAvecNomPrenom(const std::string& p_nom, const std::string& p_prenom) const {
    PRECONDITION(!p_nom.empty());
    PRECONDITION(!p_prenom.empty());

    std::string cleNomPrenom = p_nom + p_prenom;

    if (!m_tableParNomPrenom.contient(cleNomPrenom)) {
        throw std::invalid_argument("Nom et prénom introuvables dans le bottin.");
    }

    int index = m_tableParNomPrenom.element(cleNomPrenom);

    return m_tableauDesEntrees[index];
}

    const Bottin::Entree& Bottin::trouverAvecTelephone(const std::string& p_telephoneFixe) const {
    PRECONDITION(!p_telephoneFixe.empty());

    if (!m_tableParTelephone.contient(p_telephoneFixe)) {
        throw std::invalid_argument("Numéro de téléphone introuvable dans le bottin.");
    }

    int index = m_tableParTelephone.element(p_telephoneFixe);
    return m_tableauDesEntrees[index];
}



    void Bottin::afficherBottin(std::ostream& p_out) const {
    for (const auto& entree : m_tableauDesEntrees) {
        p_out << entree.m_nom << ", " << entree.m_prenom << "\t"
              << entree.m_telephoneFixe << "\t" << entree.m_cellulaire << "\t"
              << entree.m_courriel << '\n';
    }

}



    int Bottin::nombreEntrees() const {
    return m_tableauDesEntrees.size();
}


// Ratio de collisions
    double Bottin::ratioDeCollisionsNomPrenom() const {
    double ratio;
    int totalCollisions, maxCollisions;
    m_tableParNomPrenom.statistiques(ratio, totalCollisions, maxCollisions);
    return ratio;
}

    double Bottin::ratioDeCollisionTelephone() const {
    double ratio;
    int totalCollisions, maxCollisions;
    m_tableParTelephone.statistiques(ratio, totalCollisions, maxCollisions);
    return ratio;
}


    int TP3::Bottin::maximumNbCollisionNomPrenom() const {
    double ratio;
    int totalCollisions, maxCollisions;

    m_tableParNomPrenom.statistiques(ratio, totalCollisions, maxCollisions);

    return maxCollisions;
}
    int TP3::Bottin::maximumNbCollisionTelephone() const {
    double ratio;
    int totalCollisions, maxCollisions;

    m_tableParTelephone.statistiques(ratio, totalCollisions, maxCollisions);

    return maxCollisions;
}

} // namespace TP3
