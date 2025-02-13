
## **Exercice 1 : Ajout de l'objectif "boire de l'eau"**  

Nous allons ajouter un nouvel objectif **"boire de l'eau"** en suivant ces étapes :  
1. Ajouter une variable `thirst` (soif) dans `State`.  
2. Créer une action `DrinkWaterAction`.  
3. Modifier le planificateur pour gérer cet objectif.  

### **Mise à jour du `State`**  

Ajoutons la gestion de la soif :  

```cpp
class State {
public:
    bool hasFood = false;
    bool hasWater = false;  
    int hunger = 100;
    int thirst = 100;  // Niveau de soif (100 = soif maximale)

    bool HasFood() const { return hasFood; }
    bool HasWater() const { return hasWater; }  
    int GetHunger() const { return hunger; }
    int GetThirst() const { return thirst; }  

    void SetFood(bool food) { hasFood = food; }
    void SetWater(bool water) { hasWater = water; }  
    void ReduceHunger() { hunger = std::max(0, hunger - 50); }
    void ReduceThirst() { thirst = std::max(0, thirst - 50); }  
};
```

---

### **Ajout de l'action "boire de l'eau"**  

Créons une action `DrinkWaterAction` qui permet à l'agent de boire.  

```cpp
class DrinkWaterAction : public Action {
public:
    bool CanExecute(const State& state) override {
        return state.HasWater() && state.GetThirst() > 0;
    }

    void Execute(State& state) override {
        std::cout << "L'agent boit de l'eau.\n";
        state.ReduceThirst();  
        state.SetWater(false);  
    }
};
```

Ajoutons aussi une action pour chercher de l'eau :  

```cpp
class SearchWaterAction : public Action {
public:
    bool CanExecute(const State& state) override {
        return !state.HasWater(); 
    }

    void Execute(State& state) override {
        std::cout << "L'agent cherche de l'eau.\n";
        state.SetWater(true);  
    }
};
```

---

### **Mise à jour du `GOAPPlanner`**  

Ajoutons le nouvel objectif **Boire** :  

```cpp
enum class Goal {
    Manger,
    ChercherNourriture,
    Boire
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
        else if (goal == Goal::Boire) {
            if (initialState.GetThirst() > 0 && !initialState.HasWater()) {
                plan.push_back(new SearchWaterAction());
                plan.push_back(new DrinkWaterAction());
            } 
            else if (initialState.HasWater()) {
                plan.push_back(new DrinkWaterAction());
            }
        }

        return plan;
    }
};
```

---

### **Mise à jour de l'agent pour gérer la soif**  

Modifions `GOAPAgent` pour choisir entre **manger** et **boire** selon ses besoins.  

```cpp
class GOAPAgent {
private:
    State state;
    GOAPPlanner planner;

public:
    GOAPAgent() {
        state.SetHunger(100);
        state.SetThirst(100);  
    }

    void PerformActions() {
        Goal goal;
        if (state.GetThirst() > state.GetHunger()) {
            goal = Goal::Boire;
        } else {
            goal = Goal::Manger;
        }

        std::vector<Action*> plan = planner.Plan(state, goal);

        for (auto action : plan) {
            if (action->CanExecute(state)) {
                action->Execute(state);
            } else {
                std::cout << "Action impossible : " << typeid(*action).name() << "\n";
            }
            delete action;
        }
    }

    void PrintState() {
        std::cout << "Faim: " << state.GetHunger() << "\n";
        std::cout << "Soif: " << state.GetThirst() << "\n";
        std::cout << "Nourriture disponible: " << (state.HasFood() ? "Oui" : "Non") << "\n";
        std::cout << "Eau disponible: " << (state.HasWater() ? "Oui" : "Non") << "\n";
    }
};
```

---

## **Exercice 2 : Réévaluation dynamique (fuite en cas d'attaque)**  

Nous allons ajouter une action pour **fuir** si l'agent est attaqué pendant qu'il mange.  

### **Ajout de la variable d'état pour le danger**  

```cpp
class State {
public:
    bool isUnderAttack = false;  
    bool IsUnderAttack() const { return isUnderAttack; }
    void SetUnderAttack(bool attack) { isUnderAttack = attack; }
};
```

---

### **Ajout de l'action "Fuir"**  

```cpp
class FleeAction : public Action {
public:
    bool CanExecute(const State& state) override {
        return state.IsUnderAttack();  
    }

    void Execute(State& state) override {
        std::cout << "L'agent fuit pour survivre !\n";
        state.SetUnderAttack(false);  
    }
};
```

---

### **Mise à jour de `GOAPPlanner` pour inclure la fuite**  

```cpp
class GOAPPlanner {
public:
    std::vector<Action*> Plan(const State& initialState, Goal goal) {
        std::vector<Action*> plan;

        if (initialState.IsUnderAttack()) {
            plan.push_back(new FleeAction());
        } 
        else if (goal == Goal::Manger) {
            if (initialState.GetHunger() > 0 && !initialState.HasFood()) {
                plan.push_back(new SearchFoodAction());
                plan.push_back(new EatAction());
            } 
            else if (initialState.HasFood()) {
                plan.push_back(new EatAction());
            }
        } 
        else if (goal == Goal::Boire) {
            if (initialState.GetThirst() > 0 && !initialState.HasWater()) {
                plan.push_back(new SearchWaterAction());
                plan.push_back(new DrinkWaterAction());
            } 
            else if (initialState.HasWater()) {
                plan.push_back(new DrinkWaterAction());
            }
        }

        return plan;
    }
};
```

---

### **Mise à jour de l'agent pour gérer une attaque**  

```cpp
void GOAPAgent::PerformActions() {

    if (rand() % 5 == 0) {
        state.SetUnderAttack(true);
        std::cout << "ALERTE : L'agent est attaqué !\n";
    }

    Goal goal;
    if (state.IsUnderAttack()) {
        goal = Goal::Manger; 
    } else if (state.GetThirst() > state.GetHunger()) {
        goal = Goal::Boire;
    } else {
        goal = Goal::Manger;
    }

    std::vector<Action*> plan = planner.Plan(state, goal);

    for (auto action : plan) {
        if (action->CanExecute(state)) {
            action->Execute(state);
        }
        delete action;
    }
}
```

