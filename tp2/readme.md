
## Partie 1 : fonctions fournies

Pour commencer, nous allons copier/coller les fonctions qui ont été fournies dans un fichier à part.

J'ai choisi le nom `added` par simplicité. A nouveau, je distingue bien les `.h` des `.c` :

### added.h

```c
#pragma once

int* AJOUTTAB(int* tab, int actualSize, int x);
int* LSQUESTIONS();
char** UNEQUESTION(int k);
```

Les questions sont ajoutées directement dans le `.c` pour différentes raisons :
- nous n'avons besoin de cette variable QUE dans ces fonctions
- définir une variable dans un `.h` va aboutir à des duplicats de déclaration, donc à des crashs

### added.c

```c
#include "added.h"

char* ALLQUESTIONS[20][6] = {
  {"Quelle est la capitale de l'Italie ?", "Madrid", "Rome", "Athènes", "Lisbonne", "B"},
  {"Qui a peint la Joconde ?", "Léonard de Vinci", "Michel-Ange", "Raphaël", "Donatello", "A"},
  {"Quel est le symbole chimique de l'or ?", "Ag", "Au", "Fe", "Pt", "B"},
  {"En quelle année a eu lieu la chute du mur de Berlin ?", "1987", "1989", "1991", "1993", "B"},
  {"Qui a écrit 'Les Misérables' ?", "Victor Hugo", "Émile Zola", "Molière", "Balzac", "A"},
  {"Quel est le plus grand océan du monde ?", "Atlantique", "Pacifique", "Arctique", "Indien", "B"},
  {"Combien y a-t-il de planètes dans le système solaire ?", "7", "8", "9", "10", "B"},
  {"Quelle est la monnaie officielle du Japon ?", "Yuan", "Yen", "Won", "Ringgit", "B"},
  {"Qui a découvert la pénicilline ?", "Louis Pasteur", "Alexander Fleming", "Marie Curie", "Joseph Lister", "B"},
  {"Quel est le plus long fleuve du monde ?", "Nil", "Amazon", "Yangtsé", "Mississippi", "B"},
  {"Quelle est la langue la plus parlée dans le monde ?", "Anglais", "Espagnol", "Mandarin", "Hindi", "C"},
  {"Qui est l'auteur de '1984' ?", "George Orwell", "Aldous Huxley", "Jules Verne", "Isaac Asimov", "A"},
  {"Quel pays a gagné la Coupe du monde de football 2018 ?", "Brésil", "France", "Allemagne", "Argentine", "B"},
  {"Quelle planète est surnommée la planète rouge ?", "Vénus", "Mars", "Jupiter", "Mercure", "B"},
  {"Quel est le nombre de côtés d'un hexagone ?", "5", "6", "7", "8", "B"},
  {"Qui a formulé la théorie de la relativité ?", "Newton", "Einstein", "Galilée", "Copernic", "B"},
  {"Quel est le plus grand désert du monde ?", "Sahara", "Gobi", "Kalahari", "Antarctique", "D"},
  {"Quel métal est liquide à température ambiante ?", "Fer", "Mercure", "Plomb", "Aluminium", "B"},
  {"Quelle est la capitale du Canada ?", "Toronto", "Montréal", "Ottawa", "Vancouver", "C"},
  {"Combien de cordes a un violon ?", "4", "5", "6", "7", "A"}
};

int* AJOUTTAB(int* tab, int actualSize, int x) {
	int* r = realloc(tab, (1 + actualSize) * sizeof(int));
	r[actualSize] = x;
	return r;
}

int* LSQUESTIONS() {
	int* r = malloc(20 * sizeof(int));
	for (int i = 0; i < 20; i++) { r[i] = i; }
	return r;
}

char** UNEQUESTION(int k) {
	return ALLQUESTIONS[k];
}

```

> [!Caution]
> Ce code ne fonctionne pas tel quel, et c'est "normal".
> J'ai laissé un piège dedans, dont je parle plus tard.

## Partie 2 : Fonctions de base à créer

Il est temps de créer nos fonctions.
Pour chaque fonction, je vais créer un `.c` et un `.h`, afin de rester propre et simple dans la conception.

### Fonction EGAL

La fonction EGAL est assez simple.
Elle prend en paramètre deux chaînes, et utilise la [fonction présentée dans le TP](https://github.com/esirem-chassel/3aa-algo/blob/main/TP/tp2.md#11-egal) pour effectuer une comparaison insensible à la casse.

```c
#pragma once
#include <string.h>

int EGAL(char* left, char* right);
```

```c
#include "egal.h"

int EGAL(char* left, char* right) {
	return (0 == _stricmp(left, right));
}
```

Testons notre fonction dans notre main :

```c
#include <stdio.h>
#include "added.h"
#include "egal.h"

int main() {
    char* a = "a";
    char* b = "b";
    char* c = "A";
    printf("%d %d\n", EGAL(a, b), EGAL(a, c));
    return 0;
}
```

### Fonction DANSLISTE

Il n'existe pas de manière simple de réaliser des arguments facultatifs en C, et c'est pour cela que je vous ai précisé à l'oral de systématiquement placer cet argument, tout simplement.

```c
#pragma once
#include "egal.h"

int DANSLISTE(char val, char liste[], int ignorerCasse);
```

La fonction commence de manière relativement simple, mais il y a un twist. Quelqu'un de naïf écrirait un code similaire au suivant :

```c
#include "dansliste.h"

int DANSLISTE(char val, char liste[], int ignorerCasse) {
	int r = 0;
	for (int i = 0; (r == 0) && (i < sizeof(liste)); i++) {
		if (val == liste[i]) {
			r = 1;
		} else if ((ignorerCasse == 1) && EGAL(val, liste[i])) {
			r = 1;
		}
	}
	return r;
}
```

Ce qui aboutirait certainement à une erreur à l'exécution (au moins), à cause de l'appel à `EGAL`, car cette fonction attend une chaîne, ou plutôt un pointeur sur caractère.

Si on veut faire les choses bien, on pourrait créer une chaîne, en initialisant une variable :
```c
char s1[] = {val, '\0'};
char s2[] = {liste[i], '\0'};
```

Pour rester simple, nous allons simplement pointer vers l'adresse du caractère en question :

```c
#include "dansliste.h"

int DANSLISTE(char val, char liste[], int ignorerCasse) {
	int r = 0;
	for (int i = 0; (r == 0) && (i < sizeof(liste)); i++) {
		if (val == liste[i]) {
			r = 1;
		}
		else if ((ignorerCasse == 1) && EGAL(&val, &liste[i])) {
			r = 1;
		}
	}
	return r;
}
```

> [!Warning]
> Ceci est possible et cohérent UNIQUEMENT parce que nous traitons des chaînes composées d'un seul caractère.

### Fonction POGNON

Cette fonction est d'une grande simplicité, mais elle est __récursive__ et renvoie un `double`.

```c
#pragma once

double POGNON(int tour);
```

```c
#include "pognon.h"

double POGNON(int tour) {
	return (tour > 1) ? (2.16 * POGNON(tour - 1)) : 1000.;
}
```

On remarquera la **notation ternaire**, qui nous permet de l'écrire simplement et de manière compacte tout en restant lisible.

## Partie 3 : jeu principale

### Menu principal

L'objectif du menu principal est simplement d'afficher les choix de jeu, mais il utilise déjà certaines fonctions.

```c
#include <stdio.h>
#include <stdlib.h>
#include "added.h"
#include "egal.h"

int main() {
	int stopIt = 0;
	while(!stopIt) {
		char saisie[2];
		printf("Voulez-vous [J]ouer ou [Q]uitter ?\n");
		scanf_s(" %1s", saisie, 2);
		if(EGAL(saisie, "j")) {
			// on va jouer !
            printf("On joue !");
			stopIt = 0;
		}
		else if (EGAL(saisie, "q")) {
			stopIt = 1;
		}
	}
	return 0;
}
```

J'utilise un booléen `stopIt` qui me permet de vérifier à la fois la saisie et le souhait de continuer ou non.
La variable `saisie` est initialisée avec une longueur à 2 pour stocker également le caractère `'\0'`, attention à ne pas l'oublier !
Sur la même idée, le `scanf_s` utilise le format `" %1s"`.
Quelques notes :
- l'espace du début permet d'ignorer tout caractère blanc avant la saisie, comme des espaces résiduels
- le 1 permet d'indiquer que la chaîne à saisir est d'une longueur de 1
- cette longueur de 1 est pour le caractère à saisir; mais la chaîne, elle, est d'une longueur de 2, d'où le troisième argument !

### Fonction TOUR

Le gros morceau !
Cette fonction n'est pas si longue, mais il convient de comprendre quelques éléments :
- cette fonction va utiliser les fonctions de `added` pour interagir avec la liste des questions
- cette fonction va devoir maintenir la liste des questions déjà posées
- cette fonction va sans doute devoir se rappeller elle-même (être récursive)

Etant donné qu'on parle de "choisir" une question parmi plusieurs, je vais également créer une fonction `rnd`, qui renverra un nombre entier aléatoire entre 0 et 20 (sur `[0,20[`).

```c
#pragma once
#include <stdio.h>
#include <stdlib.h>
#include "added.h"

double TOUR(int numero, int* questions);
int RND();
```

Le contenu de la fonction `rnd` est simpliste :

```c
int RND() {
	return rand() % 20;
}
```

> [!Note]
> On pourrait améliorer cette fonction en indiquant un nombre autre que 20 fourni en argument de la fonction. Faites-vous plaisir ! C'est une petite modification.

La première partie de `TOUR` va consister en la recherche d'une question non-déjà posée.

```c
int r = -1;
do {
	r = RND();
} while ((questions != NULL) && DANSLISTE(r, questions, 0));
```

On va donc appeller `RND` afin d'obtenir un nombre aléatoire, et s'assurer que ce nombre n'est pas dans la liste déjà utilisée.

> [!Note]
> Les plus curieux·ses auront noté qu'on appelle `DANSLISTE` avec une liste d'entiers, alors qu'elle est sensée recevoir une liste de char.
> Concrètement, C va *cast* l'entier en caractère, selon la table ASCII.
> Cela fonctionne correctement car nous utilisons des entiers entre 0 et 2<sup>7</sup>-1, qui est la plage ASCII de base.
> Sans cela, nous devrions créer une seconde fonction proche de `DANSLISTE` mais pour des entiers.
> C'est également la raison du troisième paramètre à 0 : nul besoin de comparer la casse, bien au contraire.

Ensuite, nous stockons la question choisie dans le tableau de questions, et nous obtenons les détails de la question :

```c
questions = AJOUTTAB(questions, numero - 1, r);
char** uq = UNEQUESTION(r);
```

> [!Caution]
> Comme l'ajout de la question dans le tableau va agrandir le tableau, on a ici une réallocation dynamique.
> Cette fois, j'ai été gentil envers vous, et ai fourni la fonction !
> Néanmoins, la fonction realloc dépend de `malloc.h` ou de `stdlib.h` selon les systèmes, pensons bien à les inclure dans `added.h` !

L'étape suivante est l'affichage de la question et de ses réponses.

```c
printf("%s\n", uq[0]);
int ss = 'A';
for (int as = 1; as <= 4; as++) {
	printf("%c : %s\n", ((ss - 1) + as), uq[as]);
}
```

J'utilise ici une astuce assez logique. Après avoir affiché la question, j'initialise une variable entière en me basant sur la valeur ASCII du caractère `'A'`. J'itère ensuite sur 4 éléments, afin d'afficher les quatre réponses possibles.
Parce que le premier caractère est stocké dans la variable `ss`, je peux incrémenter d'autant cette valeur pour obtenir le caractère correspondant, dans l'ordre de la table ASCII, en partant de A : A, puis B, C et D !

Traitons maintenant la saisie de l'utilisateur.
C'est assez simple, comme pour les choix du menu :

```c
char saisie[2];
char ok[] = "ABCD";
do {
	scanf_s(" %1s", saisie, 2);
} while (!DANSLISTE(saisie[0], ok, 1));
```

On utilise ici le fait que la chaîne `ok` n'est finalement qu'un tableau de caractères !

Maintenant, reste la dernière partie :
- si l'utilisateur a bien trouvé, on affiche la cagnotte
- - si le tour est le dixième, on termine le jeu
- - sinon, on lui propose de continuer ou de s'arrêter
- - - si l'utilisateur arrête, il empoche la cagnotte et revient au menu
- - - si l'utilisateur continue, on passe au tour suivant
- sinon, on arrête tout et il "perd" sa cagnotte

Initialisons d'abord, tout en haut de notre fonction, notre variable de cagnotte.
Celle-ci nous servira de valeur de retour (d'où le double dans la signature de TOUR !).

```c
if (EGAL(saisie, uq[5])) {
	pognon = POGNON(numero);
	printf("Bonne reponse !\n");
	printf("Votre cagnotte : %f\n", pognon);
	if (numero < 10) {
		printf("Souhaitez-vous [C]ontinuer ou [A]rreter ?");
		int nextValide = 0;
		while(!nextValide) {
			scanf_s(" %1s", saisie, 2);
			if (EGAL(saisie, "C")) {
				nextValide = 1;
				TOUR(numero + 1, questions);
			} else if (EGAL(saisie, "A")) {
				nextValide = 1;
			}
		}
	}
} else {
	printf("Mauvaise reponse !\n");
	pognon = 0;
}
return pognon;
```

Ici, rien de particulier ou d'impressionnant.
Le seul point remarquable est l'appel récursif à `TOUR`, en incrémentant le numéro du tour.

La fonction `TOUR` est donc un peu plus importante que les autres, mais reste raisonnable :

```c
double TOUR(int numero, int* questions) {
	double pognon = 0.;
	int r = -1;
	do {
		r = RND();
	} while ((questions != NULL) && DANSLISTE(r, questions, 0));
	questions = AJOUTTAB(questions, numero - 1, r);
	char** uq = UNEQUESTION(r);
	printf("%s\n", uq[0]);
	int ss = 'A';
	for (int as = 1; as <= 4; as++) {
		printf("%c : %s\n", ((ss - 1) + as), uq[as]);
	}
	char saisie[2];
	char ok[] = "ABCD";
	do {
		scanf_s(" %1s", saisie, 2);
	} while (!DANSLISTE(saisie[0], ok, 1));
	if (EGAL(saisie, uq[5])) {
		pognon = POGNON(numero);
		printf("Bonne reponse !\n");
		printf("Votre cagnotte : %f\n", pognon);
		if (numero < 10) {
			printf("Souhaitez-vous [C]ontinuer ou [A]rreter ?");
			int nextValide = 0;
			while(!nextValide) {
				scanf_s(" %1s", saisie, 2);
				if (EGAL(saisie, "C")) {
					nextValide = 1;
					TOUR(numero + 1, questions);
				} else if (EGAL(saisie, "A")) {
					nextValide = 1;
				}
			}
		}
	} else {
		printf("Mauvaise reponse !\n");
		pognon = 0;
	}
	return pognon;
}
```

Deux points restent en suspens, tous deux résolus dans le main :
- il faut initialiser le générateur de nombres aléatoires, UNE fois (sinon son résultat sera toujours le même)
- il faut appeller `TOUR` pour la première fois. Pour cela, nous allons initialiser un pointeur sur entier vide, c'est à dire à `NULL`.

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include "added.h"
#include "egal.h"

int main() {
	srand(time(NULL));
	int stopIt = 0;
	while(!stopIt) {
		char saisie[2];
		printf("Voulez-vous [J]ouer ou [Q]uitter ?\n");
		scanf_s(" %1s", saisie, 2);
		if(EGAL(saisie, "j")) {
			int* qs = NULL;
			TOUR(1, qs);
			stopIt = 0;
		}
		else if (EGAL(saisie, "q")) {
			stopIt = 1;
		}
	}
	return 0;
}
```

Voilà pour la partie principale !
Il existe bien des variations, et il nous reste encore à traiter cette fameuse cagnotte, qu'on renvoie, sans l'utiliser réellement.

On remarque également qu'un nombre assez important revient régulièrement : le 20, qui correspond au nombre de questions. Ce nombre devenant dynamique sur la partie Modularité du TP, il faudra certainement créer de nouvelles fonctions dans `added` et modifier les fonctions existantes, afin de procéder à l'allocation dynamique des chaînes de caractères nécessaire pour stocker ces éléments... Peut-être comme structures ?
