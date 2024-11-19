/**
 * \file Graphe.hpp
 * \brief Implémentation des méthodes définies dans Graphe.h
 */

#include "Graphe.h"

#include <list>
#include <stack>
#include <algorithm>
#include <limits>
#include <functional>
#include <iostream>
#include "ContratException.h"
#include <queue>

using namespace std;

//classe interne Sommet

/**
 * \brief		Constructeur avec paramètres
 * \param[in] p_numero    Le numéro à assigner au sommet
 * \param[in] p_tiquette L'étiquette à assigner au sommet
 * \post		Le sommet est initialisé avec les paramètres indiqués
 */
template <typename Objet>
Graphe<Objet>::Sommet::Sommet(int p_numero, const Objet& p_etiquette)
        : m_numero(p_numero),
          m_etiquette(p_etiquette),
          m_listeDest(nullptr),
          m_etat(true),
          m_predecesseur(nullptr),
          m_cout(0),
          m_precedent(nullptr),
          m_suivant(nullptr)
{
    PRECONDITION(p_numero >=0);

    INVARIANTS();

    POSTCONDITION(m_numero == p_numero);
    POSTCONDITION(m_listeDest == nullptr);
    POSTCONDITION(m_etiquette == p_etiquette);
    POSTCONDITION(m_etat == true);
    POSTCONDITION(m_precedent == nullptr);
    POSTCONDITION(m_predecesseur == nullptr);
    POSTCONDITION(m_suivant == nullptr);
    POSTCONDITION(m_cout == 0);
}

/**
 * \brief Constructeur de copie
 * @param p_source Le sommet à copier
 * \post Une copie profonde du sommet a été effectuée à partir du sommet source
 */
template <typename Objet>
Graphe<Objet>::Sommet::Sommet(Sommet* p_source)
        : m_numero(p_source->numero),
          m_etiquette(p_source->m_etiquette),
          m_listeDest(nullptr),
          m_etat(true),
          m_predecesseur(nullptr),
          m_cout(0),
          m_precedent(nullptr),
          m_suivant(nullptr)
{
    PRECONDITION(p_source != nullptr);

    INVARIANTS();

    POSTCONDITION(m_numero == p_source->numero);
    POSTCONDITION(m_listeDest == nullptr);
    POSTCONDITION(m_etiquette == p_source->m_etiquette);
    POSTCONDITION(m_etat == true);
    POSTCONDITION(m_precedent == nullptr);
    POSTCONDITION(m_predecesseur == nullptr);
    POSTCONDITION(m_suivant == nullptr);
    POSTCONDITION(m_cout == 0);
}

/**
 * \brief destructeur
 */
template <typename Objet>
Graphe<Objet>::Sommet::~Sommet() { }

/**
 * \brief vérifie les invariants
 */
template <typename Objet>
void Graphe<Objet>::Sommet::verifieInvariant() const
{
    INVARIANT(m_numero >= 0);
}
// classe interne Arc

/**
 * \brief constructeur avec paramètres
 * @param p_dest Le sommet destination
 * @param p_cout Le coût de l'arc
 * \post Un arc est créé avec les param�tres indiqués
 */
template <typename Objet>
Graphe<Objet>::Arc::Arc(Sommet* p_dest, int p_cout)
        : m_dest(p_dest),
          m_cout(p_cout),
          m_suivDest(nullptr)
{
    PRECONDITION(p_dest != nullptr);
    PRECONDITION(p_cout >= 0);

    INVARIANTS();

    POSTCONDITION(m_dest == p_dest);
    POSTCONDITION(m_cout == p_cout);
    POSTCONDITION(m_suivDest == nullptr);
}

/**
 * \brief Destructeur
 */
template <typename Objet>
Graphe<Objet>::Arc::~Arc ()
{
    m_dest = nullptr;
    m_suivDest = nullptr;
}

template <typename Objet>
void Graphe<Objet>::Arc::verifieInvariant() const
{
    INVARIANT(m_cout >= 0);
    INVARIANT(m_dest != nullptr);
}


//Classe interne ComparateurCoutSommet
/**
 * \brief Surcharge de l'opérateur () prenant 2 sommets à comparer.  On veut savoir si cout(S0) > cout(S1) ?
 * \param[in] p_S0 Le 1er sommet à comparer
 * \param[in] p_S1 Le 2e sommet à comparer
 * \pre les sommets ne sont pas nulls
 * \return  "vrai" si cout(S0) > cout(S1), "faux" sinon.
*/
template <typename Objet>
bool Graphe<Objet>::ComparateurCoutSommet::operator()(const Sommet* p_S0, const Sommet* p_S1)
{
    PRECONDITION(p_S0 != nullptr);
    PRECONDITION(p_S1 != nullptr);
    return p_S0->m_cout > p_S1->m_cout;
}

// méthodes de Graphe

/**
 * \brief Constructeur par défaut
 * \post Un graphe vide est créé
 */
template <typename Objet>
Graphe<Objet>::Graphe() : m_nbSommets(0),
                          m_listSommets(nullptr)
{

}

// Classe Graphe

/**
 * \brief Constructeur de copie
 * @param p_source Le graphe à copier
 */
template <typename Objet>
Graphe<Objet>::Graphe(const Graphe& p_source): m_nbSommets(0),
                                              m_listSommets(nullptr)
{
    _copierGraphe(p_source);
}

/**
 * \brief Destructeur
 */
template <typename Objet>
Graphe<Objet>::~Graphe()
{
    _detruireGraphe();
}

/**
 * \brief Copie d'un graphe
 * @param p_source Le graphe à copier
 * \post Une copie profonde à partir du graphe source a été effectuée
 */
template <typename Objet>
void Graphe<Objet>::_copierGraphe(const Graphe<Objet>& p_source)
{
    // Copie d'abord les sommets
    const Sommet* lSACopier = p_source.m_listSommets;
    while(lSACopier)
    {
        Sommet* lNvS = new Sommet(lSACopier->m_numero, lSACopier->m_etiquette);
        lNvS->m_suivant = m_listSommets;

        if (m_listSommets)
        {
            m_listSommets->m_precedent = lNvS;
        }

        m_listSommets = lNvS;
        ++m_nbSommets;
        lSACopier = lSACopier->m_suivant;
    }
    // Ensuite on recommene au premier sommet, mais là on copie les arêtes
    lSACopier = p_source.m_listSommets;
    while (lSACopier)
    {
        Sommet* lS0 = _getSommet(lSACopier->m_numero);
        Arc* lArcACopier = lSACopier->m_listeDest;
        while (lArcACopier)
        {
            Sommet* lS1 = _getSommet(lArcACopier->m_dest->m_numero);
            Arc* lNvArc = new Arc(lS1, lArcACopier->m_cout);
            lNvArc->m_suivDest = lS0->m_listeDest;
            lS0->m_listeDest = lNvArc;
            lArcACopier = lArcACopier->m_suivDest;
        }

        lSACopier = lSACopier->m_suivant;
    }

}

/**
 * \brief Destruction du graphe, libération de la mémoire
 */
template <typename Objet>
void Graphe<Objet>::_detruireGraphe()
{
    // On détruit tous les sommets et tous les arcs...
    Sommet* lSSuiv = 0;
    while (m_listSommets)
    {
        //std::cout<< "On a " << nbSommets << " à détruire" <<std::endl;
        lSSuiv = m_listSommets->m_suivant;
        Arc* lArcSuiv = 0;
        while (m_listSommets->m_listeDest)
        {
            lArcSuiv = m_listSommets->m_listeDest->m_suivDest;
            delete m_listSommets->m_listeDest;
            m_listSommets->m_listeDest = lArcSuiv;
        }

        delete m_listSommets;
        m_listSommets = lSSuiv;
        --m_nbSommets;
    }
    m_nbSommets = 0;
}

/**
 * \brief Surcharge de l'op�rateur d'affectation. Copie en profondeur
 * \post Le graphe a un contenu identique à 'source'.
 * @param p_source Le graphe à copier
 * @return Une référence au graphe courant pour les appels en cascade
 */
template <typename Objet>
const Graphe<Objet>& Graphe<Objet>::operator=(const Graphe<Objet>& p_source)
{
    if (this != &p_source)
    {
        _detruireGraphe();
        _copierGraphe(p_source);
    }
    return *this;
}

/**
 *   * \brief Ajout d'un sommet au graphe

 * @param p_numero Le numéro affecté au sommet à ajouter
 * @param p_etiquette L'étiquette à associer au sommet à ajouter
 * \pre Il n'y a aucun sommet avec le même numéro dans le graphe
 * \post Un sommet contenant les informations passées en argument a été ajouté au Graphe
 */
template <typename Objet>
void Graphe<Objet>::ajouterSommet(int p_numero, const Objet& p_etiquette)
{
    PRECONDITION(p_numero >= 0);
    PRECONDITION(!sommetExiste(p_numero));
    Sommet* lNvSommet = new Sommet(p_numero, p_etiquette);
    lNvSommet->m_suivant = m_listSommets;

    if (m_listSommets)
    {
        m_listSommets->m_precedent = lNvSommet;
    }

    m_listSommets = lNvSommet;
    ++m_nbSommets;
}

/**
 * \param[in] p_numero Le numéro de sommet à chercher
 * \return true si le sommet existe, false sinon
 */
template <typename Objet>
bool Graphe<Objet>::sommetExiste(int p_numero) const
{
    PRECONDITION(p_numero >= 0);
    Sommet* lSommet = m_listSommets;
    bool lExiste = false;
    while (!lExiste && lSommet)
    {
        lExiste = p_numero == lSommet->m_numero;
        lSommet = lSommet->m_suivant;
    }
    return lExiste;
}

/**
 * @return  Le nombre de sommets du graphe
 */
template <typename Objet>
int Graphe<Objet>::nombreSommets() const
{
    return m_nbSommets;
}

/**
 *
 * @return "true" si le graph est vide, false sinon.
 */
template <typename Objet>
bool Graphe<Objet>::estVide() const
{
    return m_nbSommets == 0;
}

/**
 *
 * @return Un vecteur contenant les numéros des sommets du graphe.
 */
template <typename Objet>
std::vector<int> Graphe<Objet>::listerSommets() const
{
    Sommet* lSommet = m_listSommets;
    std::vector<int> lVectSommet;
    lVectSommet.reserve(m_nbSommets);

    while (lSommet)
    {
        lVectSommet.push_back (lSommet->m_numero);
        lSommet = lSommet->m_suivant;
    }
    return lVectSommet;
}

/**
 *
 * @return Un vecteur contenant les étiquettes des sommets du graphe.
 */
template <typename Objet>
std::vector<Objet> Graphe<Objet>::listerEtiquetteSommets() const
{
    Sommet* lSommet = m_listSommets;
    std::vector<Objet> lVectSommet;
    lVectSommet.reserve (m_nbSommets);

    while (lSommet)
    {
        lVectSommet.push_back(lSommet->m_etiquette);
        lSommet = lSommet->m_suivant;
    }

    return lVectSommet;
}

/**
 *
 * @param p_etiquette  L'étiquette du sommet recherché
 * \pre Le sommet doit faire partie du graphe
 * @return Le numéro du sommet ayant l'étiquette demandée
 */
template <typename Objet>
int Graphe<Objet>
::getNumeroSommet(const Objet& p_etiquette) const
{
    Sommet* lSommet = m_listSommets;
    while (lSommet)
    {
        if (p_etiquette == lSommet->m_etiquette) break;
        lSommet = lSommet->m_suivant;
    }

    if (lSommet == nullptr)
    {
        throw std::logic_error("Erreur, aucun sommet ne porte l'étiquette demandée");
    }

    return lSommet->m_numero;
}

/**
 * \brief Retourne l'étiquette d'un sommet
 * @param p_numero Le numéro du sommet recherché
 * \pre Le sommet doit exister
 * @return L'étiquette  du sommet ayant le numéro demandé
 */
template <typename Objet>
Objet Graphe<Objet>::getEtiquetteSommet(int p_numero) const
{
    PRECONDITION(m_nbSommets > 0);
    PRECONDITION(sommetExiste(p_numero));

    Sommet* lSommet = _getSommet(p_numero);

    return lSommet->m_etiquette;
}

/**
 *   * \brief Trouve l'adresse d'un arc entre deux sommets

 * @param p_sommet1 Le 1er sommet composant l'arc
 * @param p_sommet2 Le 2e  sommet composant l'arc
 * \pre p_sommet1 est non nul.
 * \pre p_sommet2 est non nul.
 * @return Pointeur (éventuellement nul) sur l'arc reliant les 2 sommets passés
 */
template <typename Objet>
typename Graphe<Objet>::Arc* Graphe<Objet>::_getArc(Sommet* p_sommet1, Sommet* p_sommet2) const
{
    PRECONDITION(p_sommet1 != nullptr);
    PRECONDITION(p_sommet2 != nullptr);

    //On regarde les arc de sommet1 pour y rechercher le sommet2:
    Arc* lArc = p_sommet1->m_listeDest;
    while (lArc && lArc->m_dest != p_sommet2)
    {
        lArc = lArc->m_suivDest;
    }
    return lArc;
}

/**
 * \brief Trouve l'adresse d'un sommet à partir de son numéro
 * @param p_numero Le numéro du sommet recherché
 * @return le pointeur au sommet demandé, "0" s'il n'existe pas
 */
template <typename Objet>
typename Graphe<Objet>::Sommet* Graphe<Objet>::_getSommet(int p_numero) const
{
    PRECONDITION(p_numero >= 0);

    Sommet* lSommet = m_listSommets;
    while (lSommet)
    {
        if (p_numero == lSommet->m_numero)
        {
            break;
        }

        lSommet = lSommet->m_suivant;
    }

    return lSommet;
}

/**
 * \brief Retourne l'existence d'un arc
 * @param p_numOrigine Le numéro du sommet d'origine de l'arc
 * @param p_numDestination Le numéro du sommet de destination de l'arc
 * \pre Les sommets doivent appartenir au graphe
 * @return  "true" si l'arc existe, false sinon
 */
template <typename Objet>
bool Graphe<Objet>::arcExiste (int p_numOrigine, int p_numDestination) const
{
    PRECONDITION(_getSommet(p_numOrigine) != nullptr);
    PRECONDITION(_getSommet(p_numDestination) != nullptr);

    //On demande d'abord les 2 sommets
    Sommet* lS0 = _getSommet(p_numOrigine);
    Sommet* lS1 = _getSommet(p_numDestination);
    //Ensuite, on parcours les arcs du sommet d'orgine pour tenter de trouver le sommet de destination parmi
    // ces arcs
    Arc* lArc = lS0->m_listeDest;
    while (lArc)
    {
        if (lArc->m_dest->m_numero == p_numDestination)
        {
            break;
        }

        lArc = lArc->m_suivDest;
    }

    return lArc != nullptr;
}

/**
 * \brief Ajout d'un arc au graphe
 * \param[in] p_numOrigine Le numéro du sommet d'origine de l'arc à ajouter
 * \param[in] p_numDestination Le numéro du sommet de destination de l'arc à ajouter
 * \param[in] p_cout Le coût associé à cet arc
 * \pre Les deux sommets doivent exister
 * \pre L'arc n'existe pas déjà
 * \post Un arc a été ajouté entre le sommet 'numeroOrigine' et le sommet 'numeroDestination'
 */
template <typename Objet>
void Graphe<Objet>::ajouterArc(int p_numOrigine, int p_numDestination, int p_cout)
{
    PRECONDITION(_getSommet(p_numOrigine) != nullptr);
    PRECONDITION(_getSommet(p_numDestination) != nullptr);
    PRECONDITION(!arcExiste(p_numOrigine, p_numDestination));

    Arc* lNvArc = new Arc(_getSommet (p_numDestination), p_cout);
    Sommet* lS0 = _getSommet(p_numOrigine);
    lNvArc->m_suivDest = lS0->m_listeDest;
    lS0->m_listeDest = lNvArc;
}

/**
 * \brief Enlève un arc du graphe
 * @param p_numOrigine Le numéro du sommet d'origine de l'arc à enlever
 * @param p_numDestination Le numéro du sommet de destination de l'arc à enlever
 * \pre Les deux sommets identifiés par leur numéro doivent appartenir au graphe.
 * \pre Un arc reliant les deux sommets doit exister.
 * \post Le graphe contient un arc en moins, le graphe inchangé sinon
 */
template <typename Objet>
void Graphe<Objet>::enleverArc(int p_numOrigine, int p_numDestination)
{
    PRECONDITION(_getSommet(p_numOrigine) != nullptr);
    PRECONDITION(_getSommet(p_numDestination) != nullptr);
    PRECONDITION(arcExiste(p_numOrigine, p_numDestination));

    //On récupère les 2 sommets
    Sommet* lS0 = _getSommet (p_numOrigine);
    Sommet* lS1 = _getSommet (p_numDestination);
    //Ensuite, on parcours les arcs du sommet d'orgine pour tenter de trouver le sommet de destination parmi
    // ces arcs

    Arc* lArc = lS0->m_listeDest;
    Arc* lArcPrec = 0;
    while (lArc)
    {
        if (lArc->m_dest->m_numero == p_numDestination)
        {
            break;
        }
        lArcPrec = lArc;
        lArc = lArc->m_suivDest;
    }
    if (lArc)
    {
        if (lArcPrec)
        {
            lArcPrec->m_suivDest = lArc->m_suivDest;
        }
        else
        {
            lS0->m_listeDest = lArc->m_suivDest;
        }
        delete lArc;
    }
}

/**
 * \brief Enlève un sommet du graphe ainsi que tous les arcs qui vont et partent de ce sommet
 * @param p_numero Le numéro du sommet à enlever
 * \pre Il y a un sommet avec le numéro 'numero' dans le graphe
 * \post Le sommet identifié par 'numero' a été enlevé du graphe
 */
template <typename Objet>
void Graphe<Objet>::enleverSommet(int p_numero)
{
    PRECONDITION(_getSommet(p_numero) != nullptr);

    //On parcours tous les sommets car nous devons aussi enlever tous les arcs reliés à ce sommet...
    Sommet* lSPrec = nullptr;
    Sommet* lS = m_listSommets;
    Sommet* lSAEffacer = nullptr;
    bool lTrouve = false;
    while (lS)
    {
        //On enlève complètement ce sommet est ses arcs...
        if (p_numero == lS->m_numero)
        {
            //Ah! On doit se déchaîner ici... ;-)
            //std::cout << "Trouvé le sommet #" << numero << " qu'on va déconnecter du sommet # " << (lSPrec ? lSPrec->numero : -1) <<std::endl;
            Sommet* lSSuiv = lS->m_suivant;
            if (lSSuiv)
            {
                lSSuiv->m_precedent = lSPrec;
            }
            if (lSPrec)
            {
                lSPrec->m_suivant = lSSuiv;
            }
            lSAEffacer = lS;
            lTrouve = true;
            --m_nbSommets;

            //On efface *tous* ses arcs!
            Arc* lArc = lS->m_listeDest;
            Arc* lArcSuiv = 0;
            while (lArc)
            {
                lArcSuiv = lArc->m_suivDest;
                delete lArc;
                lArc = lArcSuiv;
            }
        }
        else
        {
            //On parcours les arcs de ce sommet qui n'est *pas* celui à effacer...
            Arc* lArcPrec = 0;
            Arc* lArc = lS->m_listeDest;
            while (lArc)
            {
                //Si on trouve un référence au sommet à effacer, on efface l'arc
                if (p_numero == lArc->m_dest->m_numero)
                {
                    if (lArcPrec)
                    {
                        lArcPrec->m_suivDest = lArc->m_suivDest;
                    }
                    else
                    {
                        lS->m_listeDest = lArc->m_suivDest;
                    }
                    delete lArc;
                    // On a terminé, on sait qu'il n'y a pas 2 arcs identiques...
                    break;
                }
                else
                {
                    lArcPrec = lArc;
                    lArc = lArc->m_suivDest;
                }
            }
        }
        lSPrec = lS;
        lS = lS->m_suivant;
    }
    if (lTrouve)
    {
        Sommet* lSSuiv = lSAEffacer->m_suivant;
        delete lSAEffacer;
        if (m_listSommets == lSAEffacer)
        {
            m_listSommets = lSSuiv;
        }
    }
}

/**
 *
 * @param p_numero Le numéro du sommet à interroger
 * \pre Le sommet doit exister
 * @return l'ordre de sortie du sommet.
 */
template <typename Objet>
int Graphe<Objet>::ordreSortieSommet(int p_numero) const
{
    PRECONDITION(_getSommet(p_numero) != nullptr);

    Sommet* lSommet = _getSommet(p_numero);

    //On calcul le nombre d'arête en parcourant la liste:
    int lOrdreSortie = 0;
    Arc* lArc = lSommet->m_listeDest;
    while (lArc)
    {
        lArc = lArc->m_suivDest;
        ++lOrdreSortie;
    }
    return lOrdreSortie;
}

/**
 *
 * @param p_numero Le numéro du sommet à interroger
 * \pre Le sommet doit exister
 * \post Le graphe reste inchangé
 * @return  l'ordre d'entrée du sommet.
 */
template <typename Objet>
int Graphe<Objet>::ordreEntreeSommet(int p_numero) const
{
    PRECONDITION(_getSommet(p_numero) != nullptr);

    Sommet* lSommetDest = _getSommet(p_numero);

    // On vérifie si n'importe quel sommet a un arc vers ce sommet:
    int lOrdreEntree = 0;

    Sommet* lSommet = m_listSommets;
    while (lSommet)
    {
        Arc* lArc = lSommet->m_listeDest;
        while (lArc)
        {
            if (lArc->m_dest == lSommetDest)
            {
                ++lOrdreEntree;
            }

            lArc = lArc->m_suivDest;
        }
        lSommet = lSommet->m_suivant;
    }
    return lOrdreEntree;
}

/**
 * \brief Retourne la liste des num�ros des sommets adjacents au sommet pass� en argument dans un vecteur d'entiers
 * @param p_numeroSommetsAdjacents Le numéro du sommet à interroger
 * \pre Le sommet doit appartenir au graphe
 * @return Un vector contenant les numéros des sommets adjacents à celui demandé
 * \post Le graphe reste inchangé.
 */
template <typename Objet>
std::vector<int> Graphe<Objet>::listerSommetsAdjacents(int p_numeroSommetsAdjacents) const
{
    PRECONDITION(_getSommet(p_numeroSommetsAdjacents) != nullptr);

    Sommet* lSommet = _getSommet(p_numeroSommetsAdjacents);

    //On calcul le nombre d'arête en parcourant la liste:
    std::vector<int> lSommetsAdjacents;
    Arc* lArc = lSommet->m_listeDest;
    while (lArc)
    {
        lSommetsAdjacents.push_back(lArc->m_dest->m_numero);
        lArc = lArc->m_suivDest;
    }

    return lSommetsAdjacents;
}

/**
 * \brief Retourne le cout d'un arc passé en argument
 * @param p_numOrigine Le numéro du sommet d'origine de l'arc à interroger
 * @param p_numDestination Le numéro du sommet de destination de l'arc à interroger
 * \pre L'arc doit exister
 * \exception logic_error Le sommet source et / ou destination n'existent pas
 * \exception logic_error L'arc n'existe pas
 * @return Le coût associé à l'arc demandé
 */
template <typename Objet>
int Graphe<Objet>::getCoutArc(int p_numOrigine, int p_numDestination) const
{
    PRECONDITION(_getSommet(p_numOrigine) != nullptr);
    PRECONDITION(_getSommet(p_numDestination) != nullptr);
    PRECONDITION(arcExiste(p_numOrigine, p_numDestination));

    Sommet* lS0 = _getSommet(p_numOrigine);
    Sommet* lS1 = _getSommet(p_numDestination);
    Arc* lArc = _getArc(lS0, lS1);

    return lArc->m_cout;
}


/**
 * \brief Détermine les composantes fortement connexes et les mémorise dans un conteneur passé en paramètre
 * \param[out] p_composantes Le vecteur contenant les étiquettes des composantes fortement connexes groupées dans un autre vecteur.
 * \post Les composantes fortement connexes du graphe sont placées dans 'composantes'
 * \exception bad_alloc Il n'y a pas assez de mémoire pour placer les composantes fortements connexes dans 'composantes'
 */
template <typename Objet>
void Graphe<Objet>::getComposantesFortementConnexes (std::vector<std::vector<Objet>>& p_composantes) const
{
    //First things first:
    p_composantes.clear();

    // \note Ici j'implémente l'algorithme de Kosaraju!

    //On parcours le graphe en DFS en ajoutant les sommets à  une stack
    _initEtatSommets(false);
    std::stack<Sommet*> lPileAVisiterGOrig;

    //On cherche un sommet non-visité et on le met dans la pile et on lance le DFS:
    int lNbVisites = 0;
    while (lNbVisites != m_nbSommets)
    {
        std::list<Sommet*> lPileSommetsVisitesGOrig;
        Sommet* lSommetNonVisite = m_listSommets;
        while (lSommetNonVisite)
        {
            if (lSommetNonVisite->m_etat == false)
            {
                lPileAVisiterGOrig.push(lSommetNonVisite);
                lSommetNonVisite->m_etat = true;
                break;
            }
            lSommetNonVisite = lSommetNonVisite->m_suivant;
        }
        while (!lPileAVisiterGOrig.empty())
        {
            Sommet* lSV = lPileAVisiterGOrig.top();
            lPileAVisiterGOrig.pop();
            ++lNbVisites;
            //std::cout << "DFS, visite le sommet #" << lSV->numero << ", nbvisites: " << lNbVisites << std::endl;
            lPileSommetsVisitesGOrig.push_back(lSV);
            lSV->m_etat = true;

            //On boucle sur les arcs de lSV:
            Arc* lArc = lSV->m_listeDest;
            while (lArc)
            {
                if (lArc->m_dest->m_etat == false)
                {
                    lPileAVisiterGOrig.push(lArc->m_dest);
                    lArc->m_dest->m_etat = true;
                }
                lArc = lArc->m_suivDest;
            }
        }

        // Rendu ici, on a la pile lPileVisitee qui contient la liste des sommets dans l'ordre de visite.
        // Maintenant, on crée le graph transposé!
        // On commence par dupliquer les sommets:
        Graphe<Objet> lGrapheTranspose;
        typename std::list<Sommet*>::iterator lItSVGO = lPileSommetsVisitesGOrig.begin();
        typename std::list<Sommet*>::iterator lItSVGOFin = lPileSommetsVisitesGOrig.end();
        while (lItSVGO != lItSVGOFin)
        {
            lGrapheTranspose.ajouterSommet((*lItSVGO)->m_numero, (*lItSVGO)->m_etiquette);
            ++lItSVGO;
        }

        // Parcours tous les arcs de chaque sommet et on crée leur transpose dans le graphe transposé!
        lItSVGO = lPileSommetsVisitesGOrig.begin();
        while (lItSVGO != lItSVGOFin)
        {

            Arc* lArc = (*lItSVGO)->m_listeDest;
            while (lArc)
            {
                //On ajoute l'arc seulement si le sommet destination existe dans le sous-graph
                if (lGrapheTranspose.sommetExiste(lArc->m_dest->m_numero))
                {
                    lGrapheTranspose.ajouterArc(lArc->m_dest->m_numero,
                                                 (*lItSVGO)->m_numero,
                                                 lArc->m_cout);
                }
                lArc = lArc->m_suivDest;
            }
            ++lItSVGO;
        }

        //std::cout << "Voici le sous-graphe transpose:" << std::endl<<"TTTTTTTTTTTTTTTTTTTTTTTTT" << std::endl;
        //std::cout << lGrapheTranspose << std::endl;
        //std::cout << "TTTTTTTTTTTTTTTTTTTTTTTTT" << std::endl;

        // Maintenant, on lance un parcours en profoneur à partir du 1er sommet de la liste visitée
        // Toutes les composantes visitées font partie d'un sous-graphe fortement connexe!
        int lNbVisitesSG = 0;
        lGrapheTranspose._initEtatSommets(false);
        while (!lPileSommetsVisitesGOrig.empty())
        {
            //On cherche la prochaine composante non-visitée:
            Sommet* lSV = nullptr;
            while (!lPileSommetsVisitesGOrig.empty())
            {
                const int lIndiceSommet = lPileSommetsVisitesGOrig.front()->m_numero;
                lPileSommetsVisitesGOrig.pop_front();
                lSV = lGrapheTranspose._getSommet(lIndiceSommet);
                if (lSV->m_etat == false)
                {
                    //On a trouvé une nouvelle composante à partir de laquelle on initiera un parcours!
                    //std::cout << "Trouvé 1 sommet non-visité:" << lSV->numero << std::endl;
                    break;
                }
                else
                {
                    //std::cout << " ZUT! sommet déjà visité:" << lSV->numero << std::endl;
                    lSV = nullptr;
                }
            }
            if (lSV)
            {
                std::list<Sommet*> lListSGFC;
                lGrapheTranspose._parcoursDFS(lSV, lListSGFC);
                //On récupère les étiquettes
                std::vector<Objet> lSGFC (lListSGFC.size());
                typename std::list<Sommet*>::iterator lIt = lListSGFC.begin();
                typename std::list<Sommet*>::iterator lItFin = lListSGFC.end();
                typename std::vector<Objet>::iterator lItSGFC = lSGFC.begin();
                //std::cout << "Composantes fortement connexes: " << lListSGFC.size() << std::endl;
                while (lIt != lItFin)
                {
                    *lItSGFC = (*lIt)->m_etiquette;
                    //std::cout << *lItSGFC << " -> " << std::endl;
                    ++lIt;
                    ++lItSGFC;
                    ++lNbVisitesSG;
                }
                //std::cout << std::endl << std::endl;

                p_composantes.push_back(lSGFC);
            }
        }
    }
}
/**
 * \brief	Lance un parcours en profondeur (DFS) et retourne les sommets visités dans l'ordre
 * \post	Le chemin a été correctement initialisé
 * \pre Le sommet n'est pas nul.
 * \param[in] p_Origine Le sommet d'origine du parcours
 * \param[out] p_ListeSommetsVisites La liste des sommets visités (dans l'ordre)
*/
template <typename Objet>
void Graphe<Objet>::_parcoursDFS(Sommet* p_origine, std::list<Sommet*>& p_listeSommetsVisites) const
{
    PRECONDITION(p_origine != nullptr);
    p_listeSommetsVisites.push_back(p_origine);
    p_origine->m_etat = true;

    //Les voisins:
    Arc* lArc = p_origine->m_listeDest;
    while (lArc)
    {
        if (lArc->m_dest->m_etat == false)
        {
            _parcoursDFS(lArc->m_dest, p_listeSommetsVisites);
        }
        lArc = lArc->m_suivDest;
    }

}
/**
 * \brief	Initialise l'attibut etat de l'ensemble des sommets à la valeur \a pEtat
 * \param[in] p_etat le booléen à affecter à l'ensemble des sommets.
*/
template <typename Objet>
void Graphe<Objet>::_initEtatSommets(bool p_etat) const
{
    Sommet* lSommetDepart = m_listSommets;
    while (lSommetDepart)
    {
        lSommetDepart->m_etat = p_etat;
        lSommetDepart = lSommetDepart->m_suivant;
    }
}

/**
 * \brief Initialise les sommets du graphe dans le but d'utiliser des méthodes pour trouver des chemins
 * \param[in] p_eOrigine L'étiquette du sommet d'origine à rechercher
 * \param[in] p_eDestination L'étiquette du sommet destination à rechercher
 * \param[out] p_origine Le pointeur au sommet d'origine
 * \param[out] p_destination Le pointeur au sommet destination
 * \pre Le sommet d'origine et le sommet de destination existent
 * \post Le chemin a été correctement initialise
 */
template <typename Objet>
void Graphe<Objet>::_initPathFinding (const Objet& p_eOrigine,
                                      const Objet& p_eDestination,
                                      Sommet*& p_origine,
                                      Sommet*& p_destination)
{
    PRECONDITION(_getSommet(getNumeroSommet (p_eOrigine)) != nullptr);
    PRECONDITION(_getSommet(getNumeroSommet (p_eDestination)) != nullptr);

    //On vérifie que les sommets d'origine et de destination existent:
    p_origine = _getSommet(getNumeroSommet (p_eOrigine));
    p_destination = _getSommet(getNumeroSommet (p_eDestination));

    //On met le "cout" à "infini" pour tous les sommets et on les marque non-visités:
    Sommet* lS = m_listSommets;
    while (lS)
    {
        lS->m_etat = false;
        // Astuce pour pouvoir additionner max+max sans overflow
        lS->m_cout = std::numeric_limits<int>::max () / 2;
        lS->m_predecesseur = nullptr;
        lS = lS->m_suivant;
    }

    //On met le coût de "0" pour la source:
    p_origine->m_cout = 0;
}


/**
 * \brief Récupère le chemin à partir d'un vector dans les donn�és sont inversées

 * \param[in] p_destination Pointeur sur le sommet destination de parcours déjà effectué
 * \param[out] p_chemin Le vecteur contenant le chemin se terminant à \a destination *
 * \pre p_destination existe
 * \post Le chemin en ordre (non inverse) est retourné
 */
template <typename Objet>
void Graphe<Objet>::_recupererChemin(Sommet* p_destination, std::vector<Objet>& p_chemin)
{
    PRECONDITION(p_destination != nullptr);
    //On reconstruit le chemin pour un sommet de départ et de destination donnés.

    //On calcule le chemin (s'il existe) ainsi que le coût!
    if (p_destination->m_predecesseur != 0)
    {
        //Pour retrouver le chemin, on doit empiler les prédécesseurs en commençant par la destination.
        //On se crée un liste temporaire
        std::list<Sommet*> lChemin;
        Sommet* lSP = p_destination;
        while (lSP != 0)
        {
            lChemin.push_front (lSP);
            lSP = lSP->m_predecesseur;
        }
        // On copie la liste dans le vecteur:
        p_chemin.resize(lChemin.size());
        typename std::list<Sommet*>::const_iterator lItChemin = lChemin.begin();
        const typename std::list<Sommet*>::const_iterator lItCheminFin = lChemin.end();
        typename std::vector<Objet>::iterator lItCheminRetour = p_chemin.begin();
        while (lItChemin != lItCheminFin)
        {
            //std::cout << "*lItChemin: " << *lItChemin <<std::endl;
            *lItCheminRetour = (*lItChemin)->m_etiquette;
            //std::cout << "*lItCheminRetour: " << *lItCheminRetour <<std::endl;
            ++lItCheminRetour;
            ++lItChemin;
        }
    }

}

/**
 * \brief Surcharge de l'op�rateur de flux de sortie.
 * \pre l'étiquette est un objet comparable, l'opérateur << y est surchargé
 * @param p_out Le flux de sortie.
 * @param p_graphe Le graphe à sérialiser.
 * \post Le nombre de sommets du graphe sera sérialisé
 * \post Pour chaque sommet, son num�ro, son étiquette seront sérialisés
 * \post Pour chaque sommet, tous ses liens, le numéro des voisins, seront sérialiés
 * @return
 */
template<typename Objet>
ostream& operator<< (ostream& p_out, const Graphe<Objet>& p_graphe)
{
    p_out << "Le graphe contient " << p_graphe.m_nbSommets << " sommet(s)" << std::endl;
    typename Graphe<Objet>::Sommet* vertex = p_graphe.m_listSommets;

    while (vertex != NULL)
    {
        p_out << "Sommet no " << vertex->m_numero << " " << vertex->m_etiquette << endl; // << std::endl;
        //Afficher les arcs.
        typename Graphe<Objet>::Arc* arc = vertex->m_listeDest;
        if (arc != NULL)
        {
            p_out << "Ce sommet a des liens vers le(s) sommet(s) : ";
            while (arc->m_suivDest != NULL)
            {
                p_out << arc->m_dest->m_numero << "(" << arc->m_cout << "), ";
                arc = arc->m_suivDest;
            }
            p_out << arc->m_dest->m_numero << "(" << arc->m_cout << ")";
        }
        p_out << std::endl;
        vertex = vertex->m_suivant;
    }

    return p_out;
}
/**
* \brief Vérifie si le graphe est fortement connexe.
* \pre Le graphe ne doit pas être vide.
* \post Le graphe reste inchangé.
* \return true: graphe est fortement connexe , false : sinon.
*/
template <typename Objet>
bool Graphe<Objet>::estFortementConnexe() const
{
    PRECONDITION(m_nbSommets > 0);

    std::vector<std::vector<Objet>> composantes;
    getComposantesFortementConnexes(composantes);

    return composantes.size() == 1;
}

/**
* \brief Implémente l'algorithme de Dijkstra pour trouver le plus court chemin entre deux sommets.
* \param[in] p_eOrigine L'étiquette du sommet d'origine.
* \param[in] p_eDestination L'étiquette du sommet de destination.
* \param[out] p_chemin Le vecteur contenant les étiquettes des sommets formant le chemin le plus court.
* \pre Les sommets d'origine et de destination doivent exister dans le graphe.
* \post Le graphe reste inchangé, et le chemin le plus court est retourné dans le vecteur p_chemin.
* \exception bad_alloc Si l'allocation mémoire pour construire le chemin échoue.
* \return Le coût total du chemin le plus court entre p_eOrigine et p_eDestination.
*/

template <typename Objet>
int Graphe<Objet>::dijkstra(const Objet& p_eOrigine, const Objet& p_eDestination, std::vector<Objet>& p_chemin)
{
    PRECONDITION(sommetExiste(getNumeroSommet(p_eOrigine)));
    PRECONDITION(sommetExiste(getNumeroSommet(p_eDestination)));

    Sommet* origine = nullptr;
    Sommet* destination = nullptr;

    // Initialize pathfinding data
    _initPathFinding(p_eOrigine, p_eDestination, origine, destination);

    auto compare = [](const Sommet* a, const Sommet* b) { return a->m_cout > b->m_cout; };
    priority_queue<Sommet*, std::vector<Sommet*>, decltype(compare)> filePriorite(compare);

    filePriorite.push(origine);

    while (!filePriorite.empty())
    {
        Sommet* current = filePriorite.top();
        filePriorite.pop();

        if (current == destination)
            break;

        Arc* arc = current->m_listeDest;
        while (arc)
        {
            Sommet* neighbor = arc->m_dest;
            int newCost = current->m_cout + arc->m_cout;

            if (newCost < neighbor->m_cout)
            {
                neighbor->m_cout = newCost;
                neighbor->m_predecesseur = current;
                filePriorite.push(neighbor);
            }

            arc = arc->m_suivDest;
        }
    }

    _recupererChemin(destination, p_chemin);

    return destination->m_cout;
}

/**
* \brief Trouve les sommets d’articulation du graphe orienté.
* \pre Le graphe ne doit pas être vide.
* \post Le graphe reste inchangé après l'exécution et Le chemin est retourné.
* \param[out] p_sommets Un vecteur contenant les étiquettes des sommets d’articulation.
* \exception bad_alloc Si une erreur de mémoire se produit lors de l'ajout de sommets au vecteur.
*/

template <typename Objet>
void Graphe<Objet>::getPointsArticulation(std::vector<Objet>& p_sommets)
{
    PRECONDITION(!estVide());

    p_sommets.clear();

    Sommet* current = m_listSommets;
    while (current)
    {

        std::vector<std::vector<Objet>> composantesBeforeRemoval;
        getComposantesFortementConnexes(composantesBeforeRemoval);


        int vertexToRemove = current->m_numero;
        Graphe<Objet> tempGraph(*this);
        tempGraph.enleverSommet(vertexToRemove);

        std::vector<std::vector<Objet>> composantesAfterRemoval;
        tempGraph.getComposantesFortementConnexes(composantesAfterRemoval);

        if (composantesAfterRemoval.size() > 1)
        {
            p_sommets.push_back(current->m_etiquette);
        }

        current = current->m_suivant;
    }
}
