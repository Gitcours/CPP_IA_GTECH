
## **Exercice 1 : Implémentation d'un Behavior Tree en C++ pour un Ennemi**

Durée estimée : 1 heure


### **Objectif**  
Vous allez implémenter un **Behavior Tree (BT)** en C++ pour gérer l'intelligence artificielle d'un ennemi dans un jeu. L'ennemi doit adopter différents comportements en fonction de la situation :

- **Patrouiller** si le joueur est loin.  
- **Suivre** le joueur s'il est détecté.  
- **Attaquer** si le joueur est à portée.  
- **Fuir** si sa vie est faible.  

---

### **1. Définition des structures de base du Behavior Tree**  
Créez une hiérarchie de classes pour représenter les nœuds du **Behavior Tree**. Implémentez une classe de base `BTNode` et des classes dérivées pour les nœuds composites (**Séquence, Sélecteur**) et les feuilles (**actions spécifiques**).

---

### **2. Création des conditions et actions**
Implémentez des **conditions** et des **actions** pour les quatre comportements de l'ennemi :
1. **Condition** : Le joueur est détecté.  
2. **Condition** : Le joueur est à portée d’attaque.  
3. **Condition** : La vie est faible.  
4. **Action** : Patrouiller.  
5. **Action** : Suivre le joueur.  
6. **Action** : Attaquer.  
7. **Action** : Fuir.  

---

### **3. Assemblage du Behavior Tree**
Assemblez le Behavior Tree en utilisant une structure comme ceci (exemple):

```
        Sélecteur
       /    |    \
  Fuir   Sélecteur  Patrouille
        /        \
   Attaque    Suivre
```

---





## **Exercice 2 : Implémentation du GOAP en C++ pour un ennemi intelligent**

Durée estimée : 1 heure

### Objectif
L'objectif de cet exercice est d'implémenter un système de **GOAP (Goal-Oriented Action Planning)** pour un ennemi qui adopte un comportement adaptatif selon l'état du jeu.

### Comportement attendu de l'ennemi
L'ennemi doit adopter les comportements suivants en fonction de son état et de celui du joueur :
- **Patrouiller** : si le joueur n'est pas proche.
- **Suivre** : si le joueur est détecté.
- **Attaquer** : si le joueur est à portée.
- **Fuir** : si la vie de l'ennemi est faible.

### Instructions

### 1. Création des structures de base
Créez une classe *State* qui contient les informations suivantes :
```cpp
class State {
public:
    bool playerInSight;
    bool playerInRange;
    bool lowHealth;
};
```

Créez une classe **Action** qui représente une action GOAP. Chaque action doit avoir :
- Un coût (int)
- Une méthode `canExecute()` pour exécuter l'action
- Un effet sur le State
- Une méthode `execute()` pour exécuter l'action

```cpp
class Action {
public:
    int cost;
    virtual bool CanExecute(const State& state) = 0;
    virtual void Execute(State& state) = 0;
    virtual ~Action() {}
};
```

### 2. Implémentation des actions
Créez les classes suivantes qui héritent de `Action` :
- **PatrolAction** : l'ennemi patrouille (aucun pré-requis).
- **FollowAction** : l'ennemi suit le joueur si `playerInSight == true`.
- **AttackAction** : l'ennemi attaque si `playerInRange == true`.
- **FleeAction** : l'ennemi fuit si `lowHealth == true`.


### 3. Implémentation du planificateur GOAP
Créez une fonction `Plan()` qui :
- Reçoit un état actuel du jeu (State) et un objectif dépendant de l'état
- Retourne l'action applicable avec le coût le plus faible


### 4. Simulation
Initialisez un ennemi et testez différents états du jeu en appelant `planifier()` et en exécutant l'action retournée.


### Critères de validation
- Un système de GOAP fonctionnel avec au moins 4 actions
- Une planification déterminant la meilleure action
- Un ennemi qui change de comportement en fonction de l'état du monde


