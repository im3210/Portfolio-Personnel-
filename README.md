private:
    std::vector<Entree> m_tableauDesEntrees; // Le tableau des entrées

    //Vous devez ajouter d'autres paramètres et/ou classes privés pour compléter le modèle proposé dans l'énoncé
    //Vous pouvez ajouter également des méthodes privées.
    // Tables de hachage pour indexer les données
    labTableHachage::TableHachage<std::string, int, labTableHachage::HString1> m_tableParTelephone;
    labTableHachage::TableHachage<std::string, int, labTableHachage::HString1> m_tableParNomPrenom;

    // Statistiques sur les collisions
    size_t m_nombreCollisionsTelephone = 0;
    size_t m_nombreCollisionsNomPrenom = 0;
    size_t m_maxCollisionsTelephone = 0;
    size_t m_maxCollisionsNomPrenom = 0;


    
};
