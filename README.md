
/** \file Reseau.cpp
 * \brief Définition de l'interface pour un reseau informatique
 * \author TE
*/
#include <algorithm>
#include "Reseau.h"
#include "ContratException.h"
#include <queue>

using namespace std;

/**
 * \brief Constructeur par d�faut
 * \post Un r�seau vide est instanci�.
 */
Reseau::Reseau () { };

/**
 * \brief Constructeur à partir d'un fichier en entrée.
 * \param[in, out] p_fichierEntree fichierEntree un flux sur un fichier d'entrée
 * \pre Il y a assez de mémoire pour charger tous les routeur de la liste.
 * \pre	Le fichier p_fichierEntree est ouvert corectement.
 * \post Le fichier p_fichierEntree n'est pas fermé par la fonction.
 * Si les préconditions sont respectées, les données du réseau contenu
 * dans le fichier en entrée sont organis�es dans un graphe en m�moire.

 */
Reseau::Reseau(std::ifstream& p_fichierEntree)
{
    //Lecture des données sur les routeurs
    string nom; //Pour le nom des routeurs
    string adresseIP;
    string localisation;
    int numero = 1;
    vector<string> tabNomRouteurs;

    bool sentinelle = false;

    while (!p_fichierEntree.eof () && sentinelle == false)
    {
        getline(p_fichierEntree, nom);
        cout << nom << endl;
        if (nom == "$")//limite de la première partie du fichier
        {
            sentinelle = true;
        }
        else
        {
            getline(p_fichierEntree, adresseIP);
            getline(p_fichierEntree, localisation);
            Routeur lRouteur (nom, adresseIP, localisation);
            m_graphe.ajouterSommet(numero, lRouteur);
            ++numero;
            tabNomRouteurs.push_back(nom);
        }
    }

    int tempsTransmission;
    string nomD; //nom du routuer de destination
    int indiceSource;
    int indiceDestination;
    char buff[255];
    vector<string>::iterator location;
    while (!p_fichierEntree.eof())
    {
        p_fichierEntree.getline(buff, 100);
        nom = buff;
        location = find(tabNomRouteurs.begin (), tabNomRouteurs.end (), nom);
        indiceSource = location - tabNomRouteurs.begin ();
        p_fichierEntree.getline(buff, 100);
        nomD = buff;
        location = find(tabNomRouteurs.begin (), tabNomRouteurs.end (), nomD);
        indiceDestination = location - tabNomRouteurs.begin ();
        p_fichierEntree >> tempsTransmission;
        p_fichierEntree.ignore();

        m_graphe.ajouterArc(indiceSource + 1, indiceDestination + 1,
                             tempsTransmission);
    }

    cout << "********Graphe lu: ********" << std::endl;
    cout << m_graphe;
    cout << "***************************" << std::endl;

}

/**
 * \brief Affiche une liste de routeurs du réseau � l'ecran.
 * @param p_vRouteurs Une liste de routeurs dans un vector.
 * \post Le contenu de la liste v est affiché.
 */
void
Reseau::afficherRouteurs (std::vector<Routeur> & p_vRouteurs)
{
    if (p_vRouteurs.size () == 0)
    {
        std::cout << "La liste est vide\n";
        return;
    }

    for (unsigned int i = p_vRouteurs.size (); i > 0; i--)
    {
        std::cout << p_vRouteurs[i - 1] << std::endl;
    }
}

/**
 * \brief Affiche la liste de tous les routeurs du réseau � l'écran
 * \pre	Le reseau est non vide
 * \post La liste de tous les routeurs du r�seau sont affich�es.
 */
void
Reseau::afficherRouteurs()
{
    PRECONDITION(!m_graphe.estVide());
    std::vector<Routeur> lListeRouteur = m_graphe.listerEtiquetteSommets ();
    afficherRouteurs (lListeRouteur);
}

/**
 *
 * @param p_vRouteurs liste des routeurs enregistrés dans le réseau
 */
void
Reseau::getRouteurs (std::vector<Routeur>& p_vRouteurs) const
{
    p_vRouteurs = m_graphe.listerEtiquetteSommets ();
}
/**
* \brief Vérifie si tous les routeurs sont accessibles.
* \pre Le réseau ne doit pas être vide.
* \return true : les routeurs sont accessibles ,false : sinon.
*/
bool Reseau::routeursAccessibles() {
    PRECONDITION(!m_graphe.estVide());
    bool accessible = m_graphe.estFortementConnexe();
    POSTCONDITION(accessible == m_graphe.estFortementConnexe());
    return accessible;
}

/**
* \brief Identifie les routeurs critiques.
* \pre Le réseau ne doit pas être vide.
* \post Le réseau original reste inchangé.
* \exception bad_alloc S'il n'y a pas assez de mémoire pour retourner la liste des routeurs critiques.
* \return Un vecteur contenant les routeurs critiques.Sinon un vecteur vide.
*/
std::vector<Routeur> Reseau::routeursCritiques() {

    PRECONDITION(!m_graphe.estVide());

    const int nombreSommetsAvant = m_graphe.nombreSommets();

    std::vector<Routeur> routeursCritiques;

    m_graphe.getPointsArticulation(routeursCritiques);

    POSTCONDITION(nombreSommetsAvant == m_graphe.nombreSommets());

    return routeursCritiques;
}

/**
* \brief Trouve le plus court chemin entre deux routeurs en utilisant l'algorithme de Dijkstra.
* \pre Les routeurs d'origine et de destination doivent exister dans le réseau.
* \post Le graphe original reste inchangé.
* \post Le temps de transmission total est retourné, -1 s'il n'y a pas de chemin.
* \post La liste des routeurs à parcourir est retournée.
* \exception bad_alloc S'il n'y a pas assez de mémoire pour retourner la liste des routeurs.
* \param[in] p_routeurOrigine Le routeur d'origine.
* \param[in] p_routeurDestination Le routeur de destination.
* \param[out] p_nbSecondes Le temps de transmission total.
* \return Un vecteur contenant les routeurs du chemin minimal.Sinon un vecteur vide.
*/
std::vector<Routeur> Reseau::determinerMinParcours(const Routeur& p_routeurOrigine,
                                                   const Routeur& p_routeurDestination,
                                                   int& p_nbSecondes) {
    PRECONDITION(m_graphe.sommetExiste(m_graphe.getNumeroSommet(p_routeurOrigine)));
    PRECONDITION(m_graphe.sommetExiste(m_graphe.getNumeroSommet(p_routeurDestination)));

    const int nombreSommetsAvant = m_graphe.nombreSommets();
    std::vector<Routeur> chemin;
    int totalTime = m_graphe.dijkstra(p_routeurOrigine, p_routeurDestination, chemin);

    if (totalTime == std::numeric_limits<int>::max()) {
        p_nbSecondes = -1;
        POSTCONDITION(nombreSommetsAvant == m_graphe.nombreSommets());
        return {};
    }

    std::vector<Routeur> parcours = chemin;

    p_nbSecondes = totalTime;

    POSTCONDITION(nombreSommetsAvant == m_graphe.nombreSommets());
    return parcours;
}

