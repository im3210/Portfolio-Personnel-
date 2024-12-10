/**
 * \file TableHachage.hpp
 * \brief Implémantation des opérateurs de la classe table de hachage
 * \author Ludovic Trottier
 * \author Abder
 * \version 0.3
 *
 */
#include <cmath>
#include "ContratException.h"

/**
 * \namespace labTableHachage
 * \brief Namespace du laboratoire contenant la classe TableHachage
 */
namespace labTableHachage
{

/**
 * \brief Constructeur
 *
 * Prépare une table vide de taille nombre premier.
 *
 * \pre Il faut qu'il y ait suffisamment de mémoire
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
TableHachage<TypeClef, TypeElement, FoncteurHachage>::TableHachage(size_t size) :
m_tab(_prochainPremier(size)), m_cardinalite(0), m_nInsertions(0), m_nCollisions(0)
{
    vider();
}

/**
 * \brief Insertion d'une paire (clef, valeur) dans la table de dispersion
 *
 * Une redispersion quadratique sera utilisée en cas de collision
 *
 * Si, après une insertion, le taux de remplissage atteint le maximum, on double la taille
 * de la table et on prend le nombre premier suivant.
 * @param p_clef
 * @param p_el valeur
 * \pre Il faut qu'il y ait assez de mémoire
 * \pre La clef à insérer n'est pas déjà présente dans la table
 * \post La clef est ajoutée avec sa valeur
 */
    template<typename TypeClef, typename TypeElement, class FoncteurHachage>
    void TableHachage<TypeClef, TypeElement, FoncteurHachage>::inserer(
        const TypeClef& p_clef, const TypeElement& p_el) {
    PRECONDITION(!contient(p_clef));

    size_t valeurHachee = _distribution(p_clef);
    size_t position = valeurHachee;
    long i = 0;  // Collision counter for this insertion
    size_t collisionsPourCetteInsertion = 0;

    while (!_estVacante(position) && !_estEffacee(position)) {
        collisionsPourCetteInsertion++;
        position = (valeurHachee + static_cast<long>(std::pow(i++, 2))) % m_tab.size();
    }

    ASSERTION(_estVacante(position) || _estEffacee(position));

    m_tab[position] = EntreeHachage(p_clef, p_el, OCCUPE);
    m_cardinalite++;
    m_nInsertions++;  // Augmenter le compteur d'insertions
    m_nCollisions += collisionsPourCetteInsertion;

    // Mise à jour du nombre maximal de collisions pour une insertion
    if (collisionsPourCetteInsertion > m_maximumCollisionUneInsertion) {
        m_maximumCollisionUneInsertion = collisionsPourCetteInsertion;
    }

    POSTCONDITION(m_tab[position].m_clef == p_clef);
    POSTCONDITION(m_tab[position].m_el == p_el);
    POSTCONDITION(m_tab[position].m_info == OCCUPE);

    if (_doitEtreRehachee()) {
        rehacher();
    }
}


/**
 * \brief Trouver une position libre pour la clef. Une position libre est une position vacante ou
 *        une position effacée.
 *
 * On applique premièrement la fonction de hachage sur la clef qui retourne la
 * position initiale. Si cette position n'est pas libre, on applique la redistribution
 * quadratique. La redistribution s'arrête lorsqu'une position vacante est trouvée ou
 * lorsqu'une position contenant la clef (qui devrait être effacé dans ce cas) est trouvée.
 *
 * \param[in] p_clef La clef laquelle il faut trouver une position libre
 * \return La position libre de la clef
 *
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
size_t TableHachage<TypeClef, TypeElement, FoncteurHachage>::_trouverPositionLibre(
                                                                                   const TypeClef & p_clef)
{

    size_t valeurHachee = _distribution(p_clef);
    size_t position = valeurHachee;
    long i(0);

    while (!_estVacante(position) && !_estEffacee(position))
    {
        position = (valeurHachee + (long) pow(i, 2)) % m_tab.size();
        i++;

    }
    ASSERTION(_estVacante(position) || _estEffacee(position));


    return position;
}

/**
 * \brief Déterminer si une clef est présente dans la table
 *
 * \param[in] p_clef La clef laquelle il faut chercher
 * \return Bool indiquant si la clef est dans la table
 * \post La table est inchangée.
 *
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::contient(
                                                                    const TypeClef & p_clef) const
{
    size_t position = _trouverPositionClef(p_clef);
    return _clefExiste(position, p_clef) && _estOccupee(position);
}

/**
 * \brief Retourner l'élément associé à une clef
 *
 * \param[in] p_clef La clef laquelle il faut chercher l'élément associé
 * \pre La clef est dans la table
 * \return L'élément associé à la clef
 * \post La table est inchangée
 *
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
TypeElement TableHachage<TypeClef, TypeElement, FoncteurHachage>::element(
                                                                          const TypeClef & p_clef) const
{
    size_t position = _trouverPositionClef(p_clef);

    PRECONDITION(_clefExiste(position, p_clef));
    PRECONDITION(_estOccupee(position));

    return m_tab[position].m_el;
}

/**
 * \brief Retourner le nombre d'éléments dans la table
 * \post La table est inchangée
 * \return Le nombre d'éléments dans la table
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
int TableHachage<TypeClef, TypeElement, FoncteurHachage>::taille() const
{
    return m_cardinalite;
}

/**
 * \brief Calcule les statistiques du nombre moyen de collisions par insertion.
 * \pre L'objet doit avoir ajouter au moins un élément
 * \return Le nombre de collisions moyen par insertion : le nombre de collisions / le nombre d'insertions
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
void TableHachage<TypeClef, TypeElement, FoncteurHachage>::statistiques(
        double& p_ratio, int& p_nbCollisions, int& p_maximumCollisionUneInsertion) const {
    PRECONDITION(m_nInsertions > 0);

    p_ratio = static_cast<double>(m_nCollisions) / static_cast<double>(m_nInsertions);
    p_nbCollisions = m_nCollisions;
    p_maximumCollisionUneInsertion = m_maximumCollisionUneInsertion;
}



/**
 * \brief Afficher la table
 * \post La table est inchangée
 * \param[out] p_out Le ostream vers lequel afficher
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
void TableHachage<TypeClef, TypeElement, FoncteurHachage>::afficher(
                                                                    std::ostream & p_out) const
{
    p_out << "{";
    for (size_t i = 0; i < m_tab.size(); ++i)
    {
        if (m_tab[i].m_info == OCCUPE)
        {
            p_out << m_tab[i] << ",";
        }
    }
    p_out << "}";
}

/**
 * \brief Vider la table de dispersion
 * \post La table est vide
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
void TableHachage<TypeClef, TypeElement, FoncteurHachage>::vider()
{
    m_cardinalite = 0;
    for (size_t i = 0; i < m_tab.size(); ++i)
    {
        m_tab[i].m_info = VACANT;
    }
}

/**
 * \brief Déterminer si la table doit être rehacher
 * \return Bool indiquant si la table doit être rehacher
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::_doitEtreRehachee() const
{
    return ((m_cardinalite * 100) >= (TAUX_MAX * m_tab.size()));
}

/**
 * \brief Supprimer un élément de la table.
 *
 * On utilise le "lazy deletion". L'état de l'entrée est simplement mis à EFFACE.
 *
 * \pre La clé à supprimer doit être présente dans la table
 * \param[in] p_clef La clef à supprimer
 * \post La table comprend un élément de moins
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
void TableHachage<TypeClef, TypeElement, FoncteurHachage>::enlever(
                                                                   const TypeClef & p_clef)
{

    PRECONDITION(contient(p_clef));

    size_t position = _trouverPositionClef(p_clef);

    ASSERTION(_clefExiste(position, p_clef));
    ASSERTION(_estOccupee(position));

    m_tab[position].m_info = EFFACE;
    --m_cardinalite;

    POSTCONDITION(!contient(p_clef));
}

/**
 * \brief Trouver la position d'une clé dans la table.
 *
 * On applique premièrement la fonction de hachage sur la clef qui retourne la
 * position initiale. Si la clef ne se trouve pas à cette position, on applique la redistribution
 * quadratique. La redistribution s'arrête lorsqu'une position vacante est trouvée ou
 * lorsqu'une position contenant la clef est trouvée.
 *
 * \param[in] p_clef La clef laquelle il faut trouver sa position
 * \return La position trouvée
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
size_t TableHachage<TypeClef, TypeElement, FoncteurHachage>::_trouverPositionClef(
                                                                                  const TypeClef & p_clef) const
{
    size_t valeurHachee = _distribution(p_clef);
    size_t position = valeurHachee;
    long i(0);

    while (!_estVacante(position) && !_clefExiste(position, p_clef))
    {
        position = (valeurHachee + (long) pow(i, 2)) % m_tab.size();
        i++;
    }
    return position;
}

/**
 * \brief Application du foncteur de hachage sur une clef,
 * modulo la taille de la table
 *
 * \param[in] p_clef La clef à hacher
 * \post La table est inchangée
 * \return La position de la clef
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
size_t TableHachage<TypeClef, TypeElement, FoncteurHachage>::_distribution(
                                                                           const TypeClef & p_clef) const
{
    return m_hachage(p_clef) % m_tab.size();
}

/**
 * \brief Déterminer si une position est vacante
 * \return Bool indiquant si une position est vacante
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::_estVacante(
                                                                       size_t p_position) const
{
    return m_tab[p_position].m_info == VACANT;
}

/**
 * \brief Déterminer si une position est effacée
 * \return Bool indiquant si une position est effacée
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::_estEffacee(
                                                                       size_t p_position) const
{
    return m_tab[p_position].m_info == EFFACE;
}

/**
 * \brief Déterminer si une position est occupée
 * \return Bool indiquant si une position est occupée
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::_estOccupee(
                                                                       size_t currentPos) const
{
    return m_tab[currentPos].m_info == OCCUPE;
}

/**
 * \brief Déterminer si la clef est à la position choisie
 * \return Bool indiquant si la clef est à la position choisie
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::_clefExiste(
                                                                       const size_t & p_position, const TypeClef & p_clef) const
{
    return m_tab[p_position].m_clef == p_clef;
}

/**
 * \brief Rehacher la table.
 *
 * La taille est rehachée quand le taux maximum d'occupation
 * est dépassé.
 *
 * La taille est doublée et le premier nombre premier suivant
 * ce nombre est choisie comme nouvelle taille.
 *
 * \post La table est rehachée avec la nouvelle taille
 *
 */
    template<typename TypeClef, typename TypeElement, class FoncteurHachage>
    void TableHachage<TypeClef, TypeElement, FoncteurHachage>::rehacher() {
    std::vector<EntreeHachage> entreesActives;
    _reqEntreesActives(entreesActives);

    // Réinitialisation des compteurs
    m_nInsertions = 0;
    m_nCollisions = 0;
    m_maximumCollisionUneInsertion = 0;

    _redimensionner();
    vider();

    for (const auto& entree : entreesActives) {
        inserer(entree.m_clef, entree.m_el);
    }
}


/**
 * \brief Redimensionner la taille de la table de dispersion.
 *
 * La taille est doublée et le premier nombre premier suivant
 * ce nombre est choisie comme nouvelle taille.
 *
 * \post La table à la nouvelle taille
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
void TableHachage<TypeClef, TypeElement, FoncteurHachage>::_redimensionner()
{
    m_tab.resize(_prochainPremier(2 * m_tab.size()));
}

/**
 * \brief Retourner les entrées de la table qui sont actives
 * \param[out] p_v Un vecteur qui contiendra les entrées actives de la table
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
void TableHachage<TypeClef, TypeElement, FoncteurHachage>::_reqEntreesActives(
                                                                              std::vector<EntreeHachage> & p_v) const
{
    for (size_t i = 0; i < m_tab.size(); ++i)
    {
        if (m_tab[i].m_info == OCCUPE)
        {
            p_v.push_back(m_tab[i]);
        }
    }
}

/**
 * \brief Retourner le prochain nombre premier à partir
 * d'un certain entier
 * \param[in] p_entier Entier de départ
 * \return Le nombre premier suivant p_entier
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
int TableHachage<TypeClef, TypeElement, FoncteurHachage>::_prochainPremier(
                                                                           int p_entier) const
{
    if (p_entier % 2 == 0)
    {
        p_entier++;
    }
    while (!_estPremier(p_entier))
    {
        p_entier += 2;
    }
    return p_entier;
}

/**
 * \brief Déterminer si un entier est premier
 * \param[in] p_entier L'entier à tester
 * \return Bool indiquant si l'entier est premier
 * \post La table est inchangée
 */
template<typename TypeClef, typename TypeElement, class FoncteurHachage>
bool TableHachage<TypeClef, TypeElement, FoncteurHachage>::_estPremier(
                                                                       int p_entier) const
{
    if (p_entier <= 1)
    {
        return false;
    }
    if (p_entier == 2)
    { // le seul nombre premier pair
        return true;
    }
    if (p_entier % 2 == 0)
    { // sinon, ce n'est pas un nombre premier
        return false;
    }

    int divisor = 3;
    int upperLimit = static_cast<int> (sqrt((float) p_entier) + 1);

    while (divisor <= upperLimit)
    {
        if (p_entier % divisor == 0)
        {
            return false;
        }
        divisor += 2;
    }
    return true;
}

/**
 * \brief Surcharge de l'opérateur <<
 * \param[out] p_out Le ostream vers lequel afficher
 * \param[in] p_source La table à afficher
 * \return p_out
 */
template<typename TClef, typename TElement, class FHachage>
std::ostream& operator<<(std::ostream& p_out,
        const TableHachage<TClef, TElement, FHachage> & p_source)
{
    p_source.afficher(p_out);
    return p_out;
}

} //Fin du namespace
