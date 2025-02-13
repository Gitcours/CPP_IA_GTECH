### **Goal-Oriented Action Planning** pour les jeux vidéo

---

## Introduction

**GOAP** (Goal-Oriented Action Planning) est une méthode de planification utilisée en intelligence artificielle pour permettre à un agent (par exemple, un personnage non-joueur, PNJ) de prendre des décisions de manière autonome en fonction de ses objectifs. Contrairement à des approches plus simples comme les arbres de comportement ou les machines à états, GOAP permet à l'agent de planifier une série d'actions complexes pour atteindre un objectif précis, tout en réévaluant sa stratégie à chaque étape en fonction de l'état actuel du monde.

---

## 1. Qu'est-ce que GOAP ?

### a. **Composants de GOAP**

GOAP repose sur plusieurs éléments essentiels qui permettent à l'agent de prendre des décisions basées sur des objectifs.

1. **Objectifs (Goals)** :
   - Un objectif est un état désiré que l'agent veut atteindre. Par exemple, "se nourrir" ou "se reposer". Les objectifs sont liés à des conditions d'achèvement, comme la réduction de la faim de l'agent ou l'augmentation de son niveau d'énergie.
   
2. **Actions** :
   - Une action représente un comportement que l'agent peut exécuter. Chaque action a :
     - **Préconditions** : Ce qui doit être vrai pour que l'action puisse être exécutée. Par exemple, l'action "manger" nécessite que l'agent ait de la nourriture.
     - **Effets** : Ce qui change dans l'état du monde après que l'action est effectuée. Par exemple, l'action "manger" pourrait réduire la faim de l'agent.

3. **Etat (State)** :
   - L'état est la représentation actuelle du monde du jeu. Cela inclut l'état des variables pertinentes pour l'agent, comme sa faim, sa santé, ses ressources, etc.

4. **Planificateur GOAP (GOAP Planner)** :
   - Le planificateur est responsable de générer un plan d'actions pour atteindre un objectif donné. Il examine l'état actuel et choisit les actions appropriées en fonction des préconditions, puis crée un ordre d'exécution.

---

### b. **Fonctionnement du GOAP**

L'agent suit généralement les étapes suivantes pour accomplir un objectif via GOAP :

1. **Sélection de l'objectif** :
   - L'agent choisit un objectif en fonction de son état actuel (par exemple, il choisit de "manger" si sa faim est trop élevée).

2. **Planification** :
   - Le planificateur génère un plan d'actions en fonction des préconditions des actions disponibles. Ce plan est une séquence d'actions qui, lorsqu'elles sont exécutées dans l'ordre, permettent d'atteindre l'objectif.

3. **Exécution** :
   - L'agent exécute les actions du plan, modifiant l'état du monde au fur et à mesure.

4. **Réévaluation** :
   - L'agent réévalue son plan après chaque action pour s'assurer que l'objectif est toujours atteignable et pour ajuster son plan en conséquence. Si l'état change de manière imprévu (par exemple, l'agent tombe malade), l'agent peut devoir changer d'objectif.

---

## 2. Implémentation de GOAP en C++

Voici un exemple complet en C++ qui montre comment créer un agent capable de **chercher de la nourriture** et de **manger** pour réduire sa faim.

### a. Structures de données de base

**State** : Représente l'état du monde.

```cpp
#include <iostream>
#include <vector>
#include <string>

class State {
public:
    bool hasFood = false;  // La nourriture est-elle disponible ?
    int hunger = 100;      // Le niveau de faim (100 = faim maximale)

    bool HasFood() const { return hasFood; }
    int GetHunger() const { return hunger; }
    
    void SetFood(bool food) { hasFood = food; }
    void ReduceHunger() { hunger = std::max(0, hunger - 50); }  // Réduit la faim de 50 points
    void SetHunger(int level) { hunger = level; }
};
```

**Action** : Représente une action que l'agent peut réaliser.

```cpp
class Action {
public:
    virtual bool CanExecute(const State& state) = 0;
    virtual void Execute(State& state) = 0;
    virtual ~Action() {}
};

class EatAction : public Action {
public:
    bool CanExecute(const State& state) override {
        return state.HasFood() && state.GetHunger() > 0;
    }

    void Execute(State& state) override {
        std::cout << "L'agent mange.\n";
        state.ReduceHunger();  // Réduit la faim après avoir mangé
        state.SetFood(false);   // Après avoir mangé, il n'y a plus de nourriture
    }
};

class SearchFoodAction : public Action {
public:
    bool CanExecute(const State& state) override {
        return !state.HasFood();  // Peut chercher de la nourriture si l'agent n'en a pas
    }

    void Execute(State& state) override {
        std::cout << "L'agent cherche de la nourriture.\n";
        state.SetFood(true);  // Trouve de la nourriture
    }
};
```

**GOAPPlanner** : Le planificateur génère un plan d'actions en fonction de l'état.

```cpp
enum class Goal {
    Manger,
    ChercherNourriture
};


class GOAPPlanner {
public:
    std::vector<Action*> Plan(const State& initialState, Goal goal) {
        std::vector<Action*> plan;

        if (goal == Goal::Manger) {
            if (initialState.GetHunger() > 0 && !initialState.HasFood()) {
                plan.push_back(new SearchFoodAction());
                plan.push_back(new EatAction());
            } 
            else if (initialState.HasFood()) {
                plan.push_back(new EatAction());
            }
        }

        return plan;
    }
};
```

**GOAPAgent** : L'agent exécute les actions générées par le planificateur.

```cpp
class GOAPAgent {
private:
    State state;
    GOAPPlanner planner;

public:
    GOAPAgent() {
        state.SetHunger(100);  // Initialement, l'agent a faim
    }

    void PerformActions() {
        Goal goal = Goal::Manger;  // L'objectif de l'agent est de manger

        std::vector<Action*> plan = planner.Plan(state, goal);

        for (auto action : plan) {
            if (action->CanExecute(state)) {
                action->Execute(state);  // Exécute l'action
            } else {
                std::cout << "Action impossible : " << typeid(*action).name() << "\n";
            }
            delete action;  // Libérer la mémoire
        }
    }

    void PrintState() {
        std::cout << "Faim: " << state.GetHunger() << "\n";
        std::cout << "Nourriture disponible: " << (state.HasFood() ? "Oui" : "Non") << "\n";
    }
};
```

### b. Fonction main()

```cpp
int main() {
    GOAPAgent agent;

    std::cout << "Etat initial de l'agent:\n";
    agent.PrintState();

    std::cout << "\nL'agent commence ses actions...\n";
    agent.PerformActions();  // L'agent va chercher de la nourriture, puis manger

    std::cout << "\nEtat de l'agent après avoir effectué les actions:\n";
    agent.PrintState();

    return 0;
}
```

---

### 3. Exercices

#### **Exercice 1** : **Ajout de nouveaux objectifs et actions**

Ajoutez un objectif supplémentaire pour l'agent : **"boire de l'eau"**. Implémentez une action qui permet à l'agent de boire de l'eau, et modifiez l'agent pour qu'il puisse choisir cet objectif si sa soif est élevée.

**Consignes** :
1. Créez une nouvelle variable d'état pour la **soif** de l'agent.
2. Ajoutez une action `DrinkWaterAction` qui diminue la soif de l'agent.
3. Modifiez la logique de planification pour permettre à l'agent de boire lorsqu'il a soif.

#### **Exercice 2** : **Réévaluation dynamique**

Implémentez une réévaluation dynamique des plans de l'agent. Si l'agent commence à se faire attaquer pendant qu'il mange, il devrait abandonner l'action "manger" et s'enfuir.

**Consignes** :
1. Ajoutez une nouvelle action `FleeAction` que l'agent peut exécuter lorsqu'il est attaqué.
2. Implémentez un mécanisme où, lorsqu'un événement (attaque) survient, l'agent abandonne son plan actuel pour en créer un nouveau.
   
#### **Exercice 3** (optionnel) : **Optimisation du planificateur**

Le planificateur actuel choisit un plan d'action simple. Cependant, dans un monde plus complexe, plusieurs plans pourraient être possibles. Implémentez un algorithme de recherche (par exemple, A*) pour choisir le plan le plus optimal en fonction des ressources disponibles.

---

## Conclusion

Le **Goal-Oriented Action Planning** (GOAP) est une approche puissante pour créer des agents capables de réaliser des comportements complexes et autonomes dans des jeux vidéo. En structurant les objectifs, actions et états, GOAP permet une grande flexibilité tout en offrant une approche systématique pour gérer les décisions des agents.
