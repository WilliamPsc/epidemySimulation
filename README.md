/**
* @author William PENSEC
* @version 5.0
* @date 22/04/2020
* @logiciel http://virtulab.univ-brest.fr/matrix-studio.html
* @description Simulation d'épidémie en OpenCL.
* 				Le programme permet de simuler une progression d'épidémie selon plusieurs paramètres paramétrables dans l'onglet Librairie
**/


 ================ PROJET LIBRE OPENCL ================
 
 ===== Simulation d'épidémie avec Matrix Studio (voir @logiciel) =====
 
 1/ Conception générale
 
	- a) Matrices
	
		- 1) Resultat
		
La matrice contenant l'addition de toutes les autres matrices précédentes afin d'afficher une vue globale de la simulation

		- 2) Random
		
La matrice générant aléatoirement des nombres. Elle est utilisée tout le temps à chaque fois qu'il y a un nombre aléatoire qui doit être généré (déplacement, % de risques, ...)

		- 3) Sain, Malade, Immunise
		
Les matrices Sain1, Malade1 et Immunise1 sont des matrices principales. Elles permettent d'écrire les résultats à chaque cycle des valeurs calculées.
Alors que les matrices Sain2, Malade2, Immunise2 sont des matrices de valeurs temporaires qui ne font que stocker les valeurs calculées avant que le cycle ne se finissent et que les valeurs soient inscrites dans les matrices principales.
				
	- b) Scheduler
	
L'ordre d'exécution du programme se déroule par à l'étape 0, une initialisation des matrices principales, puis après on cycle sur le calcul et l'écriture de la simulation.
	
	- c) Kernels
	
		- 1) KernelInit
		
Ce kernel permet l'initialisation des matrices Resultat, Sain1, Malade1. On place un malade dans un endroit de la matrice selon une valeur paramétrable (voir partie 1/d)1) Paramètres) nommées MALINIT. Ensuite, pour toutes les autres cases on calcule un nombre aléatoire modulo 5 puis, on place, si ce nombre est inférieur à une probabilité calculée par (taille de la population / 128x128), un individu sain sinon la case est coloriée en noir.

		- 2) Kernel1
		
			Partie 1 : Transformations
			
La première partie de ce kernel est juste là afin de transformer un individu malade en individu immunisé (le guérir). Ce cycle se fait via une variable paramétrable qui chaque 1000 cycles double. Et on efface le pixel guéri des matrices Malade.
On calcule également un pourcentage afin de déterminer si l'individu meurt ou pas. La variable déterminante est encore ici paramétrable et le traitement fait est juste de mettre la couleur à noir dans la matrice Malade2.
La dernière transformation faite est la transformation d'un individu sain en un individu malade. Pour chaque individu sain, on regarde s'il existe un individu malade sur une case de différence (Est, Ouest, Nord, Sud) et si tel est le cas on calcule un nombre et si ce nombre est inférieur à une valeur définie dans les paramètres alors l'individu devient malade sinon il reste sain.
			
			Partie 2 : Déplacements
			
Cette seconde partie est la plus simple et la plus répétitive car elle ne fait que générer un nombre aléatoire et selon ce chiffre modulo 5 on détermine la position vers laquelle cet individu va se déplacer (0 - reste à sa place / 1 - Nord / 2 - Sud / 3 - Est / 4 - Ouest). Et on fait ça pour les individus malades, sains, et immunisés.
		
		- 3) KernelCopie
		
Ce kernel est juste là pour récupérer les valeurs temporaires et les inscrire dans les matrices principales à chaque cycle.
On recopie tout d'abord l'intégralité des matrices temporaires (pixels noirs, et pixels coloriés).
Enfin on recopie les matrices principales dans la matrice Resultat en gardant seulement les pixels de couleurs.		
		
	- d) Librairie
	
 		- 1) Paramètres
		
Il y a 5 paramètres principaux, et 3 autres paramètres secondaires qui sont les couleurs des individus. Tous sont modifiables dans la partie Libraries/Library.
Plus de détails sont disponibles dans la partie 2/
				
		- 2) Fonction alea
		
Cette fonction est seulement la fonction calculant les nombres aléatoire selon une seed (chiffre généré dans la matrice Random). Elle retourne enfin la valeur générée.
 
 
 2/ Paramètres modifiables
 
	- a) MALINIT : Position initiale du malade (chiffre entre 0 et 8)
	- b) POPULATION : Valeur de population. Ce chiffre doit être très élevé car on applique une division dessus.
	- c) RND_MAX : Valeur d'aléatoire. Fixée à 16 384. Si la valeur est trop élevée cela peut faire des problèmes dans l'aléatoire
	- d) DEATH : Pourcentage de risque d'avoir la mort de l'individu malade
	- e) GUERISON : Valeur sur dix milles chance de guérir de la maladie.
	- f) RO : Pourcentage de transmission de la maladie. 0 % aucune transmission / 100% la maladie se répend très vite
	
	
3/ Problèmes persistants

	Le plus gros problème qu'il me reste mais que je n'arrive pas à résoudre c'est la disparition des individus de même type lorsqu'ils se chevauchent.
	Par exemple, Un individu de type X rencontre un individu de type Y rien ne bouge.
	Par contre si un individu de type X rencontre un individu de type X alors un des 2 individus va disparaitre et donc on perd 1 individu à chaque fois que cela se produit.
	Le phénomène est visible notamment avec les personnes saines; au départ on a un nombre très important d'individus sains, et plus le temps passe et plus le nombre d'individus diminue.
