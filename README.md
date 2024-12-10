/**
 * \file Bottin.h
 * \brief Classe définissant un bottin téléphonique.
 * \author Thierry Eude
 * 
 */

#ifndef BOTTIN__H
#define BOTTIN__H

#include <stdexcept>
#include <iostream>
#include <fstream> 
#include <string>
#include <vector>
#include "TableHachage.h"
#include "FoncteurHachage.hpp"


namespace TP3
{

/** 
 * \class Bottin
 *
 * \brief classe représentant un bottin
 *
 */
class Bottin
{
    class Entree;

public:

    // à compléter (voir l'énoncé du TP)
    Bottin(){};
    Bottin(std::ifstream& p_fichierEntree, size_t p_table_size = 100);
    void ajouter(const std::string& p_nom, const std::string & p_prenom, const std::string & p_telephoneFixe,
                 const std::string & p_cellulaire, const std::string & p_courriel);
    const Entree&
    trouverAvecNomPrenom(const std::string& p_nom, const std::string & p_prenom) const;
    const Entree&
    trouverAvecTelephone(const std::string & p_telephoneFixe) const;
    void afficherBottin(std::ostream &) const;
    int nombreEntrees() const;
    double ratioDeCollisionsNomPrenom() const;
    double ratioDeCollisionTelephone() const;
    int maximumNbCollisionNomPrenom() const;
    int maximumNbCollisionTelephone() const;

    friend std::ostream& operator<<(std::ostream &, const Bottin&);

private:

    /** 
     * \class Entree
     *
     * \brief classe représentant une entrée du bottin
     *
     */
    class Entree
    {
    public:

        std::string m_nom;
        std::string m_prenom;
        std::string m_telephoneFixe;
        std::string m_cellulaire;
        std::string m_courriel;

        Entree(const std::string& p_nom, const std::string & p_prenom, const std::string & p_telephobeFixe, const std::string & p_cellulaire, const std::string & p_courriel)
        : m_nom(p_nom), m_prenom(p_prenom), m_telephoneFixe(p_telephobeFixe), m_cellulaire(p_cellulaire), m_courriel(p_courriel)
        {
        }

        friend std::ostream& operator<<(std::ostream & p_out, const Entree& p_source)
        {
            p_out << p_source.m_nom << ", " << p_source.m_prenom << ", " << p_source.m_telephoneFixe << ", " << p_source.m_cellulaire << ", " << p_source.m_courriel << std::endl;
            return p_out;
        }
    };
private:
    std::vector<Entree> m_tableauDesEntrees;
    labTableHachage::TableHachage<std::string, int, labTableHachage::HString1> m_tableParTelephone;
    labTableHachage::TableHachage<std::string, int, labTableHachage::HString1> m_tableParNomPrenom;

    // Statistiques sur les collisions
    size_t m_nombreCollisionsTelephone = 0;
    size_t m_nombreCollisionsNomPrenom = 0;
    size_t m_maxCollisionsTelephone = 0;
    size_t m_maxCollisionsNomPrenom = 0;


    
};

}

#endif /* BOTTIN__H */
