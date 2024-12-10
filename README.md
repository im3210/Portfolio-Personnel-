/**
 * \file FoncteurHachage.hpp
 * \brief Implémantation des foncteurs de hachage
 * \author Ludovic Trottier
 * \author Abder
 * \version 0.3
 * \date mai 2014
 *
 * Un foncteur n'est rien d'autre qu'un objet dont la classe définit l'opérateur fonctionnel '()'.
 * Les foncteurs, très utilsés dans la STL, ont la particularité de pouvoir être utilisés exactement
 * comme des fonctions puisqu'il est possible de leur appliquer leur opérateur fonctionnel selon une
 * écriture similaire à un appel de fonction. Cependant, ils sont un peu plus puissants que de simples fonctions,
 * car ils permettent de transporter, en plus du code de l'opérateur fonctionnel, des paramètres additionnels dans
 * leurs données membres. Les foncteurs constituent donc une fonctionnalité extrêmement puissante qui peut être
 * très pratique en de nombreux endroits.
 *
 */
#ifndef FONCTEURHACHAGE_H
#define FONCTEURHACHAGE_H

namespace labTableHachage {
/**
 * \class HString1
 * \brief Foncteur de hachage pour des string
 */
class HString1 {
public:
	size_t operator()(const std::string & p_clef) const {
		size_t total = 0;
		int nPremiers = 18;
		size_t premiers[18] = { 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53,
				59, 61, 67, 71, 73, 79 };
		for (size_t i = 0; i < p_clef.length(); i++) {
			total += premiers[i % nPremiers] * p_clef[i];
		}
		return total * GRAND_PREMIER;
	}
private:
	static const size_t GRAND_PREMIER = 1000003;
};

/**
 * \class HString2
 * \brief Foncteur de hachage pour des string
 */
class HString2 {
public:
	size_t operator()(const std::string & p_clef) const {
		size_t total = 5381;
		for (size_t i = 0; i < p_clef.size(); ++i) {
			total = total * 33 ^ p_clef[i];
		}
		return total;
	}
};

/**
 * \class HInt1
 * \brief Foncteur de hachage pour des int
 */
class HInt1 {
public:
	size_t operator()(const int & p_clef) const {
		return p_clef;
	}
};
/**
 * \class HInt2
 * \brief Foncteur de hachage pour des int
 */
class HInt2 {
public:
	size_t operator()(const int & p_clef) const {
		size_t total(p_clef);
		total = ((total >> 16) ^ total) * 0x45d9f3b;
		total = ((total >> 16) ^ total) * 0x45d9f3b;
		total = ((total >> 16) ^ total);
		return total;
	}
};
} // Fin namespace
#endif /* FONCTEURHACHAGE_H */
