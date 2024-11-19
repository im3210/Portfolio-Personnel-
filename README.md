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
 
