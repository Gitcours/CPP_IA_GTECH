# **Introduction à l'IA dans les Jeux Vidéo**  

## **1. Définition de l’IA dans les Jeux Vidéo**  

L'intelligence artificielle dans les jeux vidéo désigne l’ensemble des techniques permettant de simuler un comportement intelligent pour des entités contrôlées par la machine. Il peut s’agir d’ennemis, d’alliés, de créatures, de foules, ou même de systèmes générant du contenu procédural.  

---

## **2. Rôle de l’IA dans les Jeux Vidéo**  

L’IA joue un rôle fondamental dans l’expérience du joueur. Elle intervient dans plusieurs aspects d’un jeu :  

### **2.1 Comportement des PNJ**  
L'IA permet de simuler des ennemis ou alliés réactifs, capables de prendre des décisions et d’agir en fonction du joueur. Exemples :  
- **Ennemis de Metal Gear Solid** : ils patrouillent et réagissent au bruit ou à la vision du joueur.  
- **Bots de F.E.A.R.** : ils utilisent la couverture et se coordonnent pour attaquer.  

### **2.2 Navigation et Déplacement**  
Un bon système de déplacement est essentiel pour éviter que les PNJ ne restent bloqués contre des obstacles. Exemples :  
- **Pathfinding (A\*) dans Age of Empires** : permet aux unités de se déplacer intelligemment sur la carte.  
- **NavMesh dans Unity** : utilisé pour la navigation des personnages en 3D.  

### **2.3 Stratégie et Prise de Décision**  
Certains jeux nécessitent une IA capable de planifier ses actions en fonction des objectifs et des ressources disponibles. Exemples :  
- **Les adversaires de StarCraft** : prennent en compte l’économie et la gestion des troupes.  
- **Les PNJ de The Sims** : suivent des règles de satisfaction et de besoins pour agir.  

### **2.4 Génération de Contenu**  
L’IA est parfois utilisée pour générer des mondes dynamiques et variés. Exemples :  
- **Minecraft** : génération procédurale des terrains.  
- **Left 4 Dead** : système d’IA Director qui ajuste la difficulté en fonction du joueur.  

### **2.5 Interaction avec le Joueur**  
Certains jeux intègrent une IA capable de s’adapter aux actions du joueur. Exemples :  
- **Alien: Isolation** : l’Alien apprend du comportement du joueur pour le traquer.  
- **Left 4 Dead** : ajuste le nombre d’ennemis selon la performance des joueurs.  

---

## **3. Principales Techniques d’IA en Jeu Vidéo**  

Les jeux vidéo utilisent plusieurs techniques pour modéliser le comportement de l’IA. Ces techniques sont souvent combinées pour créer des comportements plus réalistes.

### **3.1 Machines à États Finis (Finite State Machines - FSM)**  
Les FSM sont une méthode simple et efficace pour gérer les comportements des PNJ en définissant des états et des transitions.  

#### **Exemple d’une IA de garde** :  
1. **Patrouille** → Si le joueur est visible, passer à **Chasse**.  
2. **Chasse** → Si le joueur est hors de vue, passer à **Recherche**.  
3. **Recherche** → Après un temps donné, revenir à **Patrouille**.  

**Avantages :** Simple, efficace, facile à implémenter.  
**Inconvénients :** Difficulté à gérer des comportements complexes sans explosion du nombre d’états.  

---

### **3.2 Algorithmes de Pathfinding**  
Le pathfinding permet de calculer un chemin entre un point A et un point B en évitant les obstacles.  

- **A*** (A-star) : utilisé dans la majorité des jeux pour optimiser les déplacements.  
- **Dijkstra** : plus lent, mais utile pour certaines applications.  
- **NavMesh** : représentation plus efficace pour le déplacement en 3D.  

---

### **3.3 Behavior Trees (Arbres de Comportement)**  
Les behavior trees permettent d’organiser des décisions complexes sous forme d’arborescence.  

Exemple d’un **ennemi qui attaque** :  
- **Condition** : Le joueur est visible ?  
    - Oui → **Tirer sur le joueur**.  
    - Non → **Chercher une couverture**.  

**Avantages :** Plus modulable que les FSM, utilisé dans de nombreux jeux modernes.  
**Inconvénients :** Complexité accrue.  

---

### **3.4 Systèmes Basés sur les Règles (Rule-Based AI)**  
Utilise des ensembles de règles du type **Si condition, alors action**.  
Exemple :  
- Si le joueur a peu de vie → **L’ennemi devient plus agressif**.  
- Si l’ennemi est blessé → **Il fuit vers un allié**.  

---

### **3.5 Intelligence Artificielle Apprenante**  
Certaines IA sont capables de s’adapter grâce à l’apprentissage automatique. Exemples :  
- **Apprentissage par renforcement** : Un agent apprend en recevant des récompenses (utilisé dans OpenAI Five pour Dota 2).  
- **Réseaux de neurones** : Utilisés pour analyser et s’adapter au comportement du joueur (Alien: Isolation).  

**Limites :**  
- Complexité de mise en œuvre.  
- Besoin d’un grand volume de données d’entraînement.  

---

## **4. Étude de Cas : IA dans des Jeux Célèbres**  

| **Jeu** | **Technique IA utilisée** |
|---------|---------------------------|
| **Pac-Man (1980)** | FSM pour les fantômes (chasse, fuite, patrouille) |
| **Half-Life (1998)** | Pathfinding et tactiques d’escouade |
| **F.E.A.R. (2005)** | Behavior Trees pour des ennemis réactifs |
| **StarCraft (1998-2020)** | IA stratégique avec gestion des ressources |
| **Alien: Isolation (2014)** | IA hybride avec apprentissage et anticipation |

---

## **Conclusion**  

L’intelligence artificielle joue un rôle essentiel dans les jeux vidéo en simulant des comportements crédibles et en améliorant l’expérience de jeu. Elle repose sur différentes techniques, allant des FSM aux algorithmes plus avancés comme les Behavior Trees et l’apprentissage automatique.  

Dans le prochain module, nous allons implémenter une **FSM en C++** pour gérer le comportement d’un PNJ.
