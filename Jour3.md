# Behavior Trees pour l'IA en Jeu Vidéo

## 1. Introduction aux Behavior Trees (BT)
### 1.1 Qu'est-ce qu'un Behavior Tree ?
Un **Behavior Tree (BT)** est une structure de données hiérarchique utilisée pour modéliser le comportement d'une intelligence artificielle (IA) dans un jeu vidéo. Contrairement aux machines à états finis (FSM), qui deviennent rapidement complexes avec de nombreux états et transitions, les BT offrent une approche plus modulaire et extensible.

Les BT sont largement utilisés dans les jeux modernes pour gérer des PNJ (personnages non-joueurs), des ennemis et même des systèmes de dialogue.

### 1.2 Avantages des Behavior Trees
- **Lisibilité et modularité** : chaque nœud a une rôle bien défini.
- **Facilité d'extension** : permet d'ajouter facilement de nouveaux comportements.
- **Contrôle hiérarchique** : facilite l'organisation logique des actions de l'IA.
- **Réutilisation** : possibilité de créer des blocs de comportement réutilisables.

### 1.3 Structure d'un Behavior Tree
Un BT est composé de plusieurs types de nœuds :
1. **Nœud Racine** : Le point d'entrée du BT.
2. **Nœuds Composites** : permettent de combiner plusieurs nœuds enfants.
   - **Sequence** : exécute les enfants en séquence jusqu'à échec ou succès.
   - **Selector** : teste les enfants un par un et s'arrête dès qu'un succès est obtenu.
3. **Nœuds Décorateurs** : modifient le comportement d'un sous-arbre (ex: inverser un résultat, ajouter un délai).
   - **Invert** : Inverse le résultat d'un nœud enfant. Si l'enfant retourne `SUCCESS`, il retourne `FAILURE` et inversement. Ce type de décorateur est utile pour vérifier si une condition **n'est pas** remplie.
   - **Repeat** : Répète l'exécution d'un nœud enfant un certain nombre de fois ou indéfiniment jusqu'à succès.
   - **Limiter** : Contrôle combien de fois un enfant peut être exécuté dans une période donnée.
4. **Nœuds Feuilles** : représentent des actions concrètes ou des conditions.
   - **Actions** : effectuent une tâche (ex: attaquer un joueur).
   - **Conditions** : vérifient un état avant d'exécuter une action.

### 1.4 Utilisation d'un Blackboard
Le **Blackboard** est une structure de données partagée entre plusieurs nœuds du BT. Il permet aux nœuds d'échanger des informations sans dépendre directement les uns des autres. 

#### Avantages du Blackboard :
- **Centralisation des informations** : les données de l'IA sont stockées à un seul endroit.
- **Flexibilité** : les nœuds peuvent accéder aux informations sans couplage direct.
- **Évolutivité** : permet d'ajouter de nouvelles interactions facilement.

#### Exemple d'utilisation du Blackboard :
Un garde pourrait stocker la position du joueur dans le blackboard lorsqu'il le détecte. Plus tard, d'autres nœuds comme "Se diriger vers la position du joueur" peuvent récupérer cette donnée pour réagir en conséquence.

## 2. Implémentation d'un Behavior Tree en C++

### 2.1 Base du Behavior Tree
Nous allons implémenter un système de Behavior Tree en C++ en définissant une structure de base avec des nœuds et un arbre principal.

#### Définition des classes de base
```cpp
#include <iostream>
#include <vector>
#include <memory>

enum class NodeState { SUCCESS, FAILURE, RUNNING };

class BTNode {
public:
    virtual ~BTNode() = default;
    virtual NodeState execute() = 0;
};
```

### 2.2 Implémentation des nœuds composites

#### SequenceNode (exécute les enfants en séquence)
```cpp
class SequenceNode : public BTNode {
private:
    std::vector<std::unique_ptr<BTNode>> children;
public:
    void AddChild(std::unique_ptr<BTNode> child) {
        children.push_back(std::move(child));
    }
    NodeState execute() override {
        for (auto& child : children) {
            if (child->Execute() == NodeState::FAILURE) {
                return NodeState::FAILURE;
            }
        }
        return NodeState::SUCCESS;
    }
};
```

#### SelectorNode (sélectionne un enfant qui réussit)
```cpp
class SelectorNode : public BTNode {
private:
    std::vector<std::unique_ptr<BTNode>> children;
public:
    void AddChild(std::unique_ptr<BTNode> child) {
        children.push_back(std::move(child));
    }
    NodeState execute() override {
        for (auto& child : children) {
            if (child->Execute() == NodeState::SUCCESS) {
                return NodeState::SUCCESS;
            }
        }
        return NodeState::FAILURE;
    }
};
```

### 2.3 Implémentation d'un Blackboard
```cpp
#include <unordered_map>
#include <string>

class Blackboard {
private:
    std::unordered_map<std::string, int> data;
public:
    void SetValue(const std::string& key, int value) {
        data[key] = value;
    }
    int GetValue(const std::string& key) {
        return data[key];
    }
};
```

### 2.4 Exemple d'utilisation avec une IA de garde
```cpp
class ConditionNode : public BTNode {
private:
    Blackboard& blackboard;
    std::string key;
    int expectedValue;
public:
    ConditionNode(Blackboard& bb, const std::string& key, int value) : blackboard(bb), key(key), expectedValue(value) {}
    NodeState execute() override {
        return (blackboard.GetValue(key) == expectedValue) ? NodeState::SUCCESS : NodeState::FAILURE;
    }
};

class ActionNode : public BTNode {
private:
    std::string actionName;
public:
    ActionNode(std::string name) : actionName(name) {}
    NodeState execute() override {
        std::cout << "Action: " << actionName << std::endl;
        return NodeState::SUCCESS;
    }
};
```

### 2.5 Construction du BT
```cpp
int main() {
    Blackboard blackboard;
    blackboard.SetValue("PlayerDetected", 0);

    auto root = std::make_unique<SelectorNode>();
    auto sequence = std::make_unique<SequenceNode>();
    sequence->AddChild(std::make_unique<ConditionNode>(blackboard, "PlayerDetected", 1));
	sequence->AddChild(std::make_unique<ActionNode>("Attaquer"));

    root->AddChild(std::move(sequence));
    root->AddChild(std::make_unique<ActionNode>("Patrouiller"));

    root->execute();
    return 0;
}
```

## 3. Exercices Pratiques

### Exercice 1 : Création d'un nœud d'action personnalisé

Créez un nœud PrintMessageNode qui affiche un message personnalisé lorsqu'il est exécuté.

### Exercice 2 : Ajout d'un décorateur "Invert"

Ajoutez un nœud décorateur InvertNode qui inverse le résultat d'un nœud enfant (SUCCESS devient FAILURE et vice versa).

### Exercice 3 : Intégration d'un blackboard

Modifiez l'exemple pour stocker une variable EnnemiProche dans un Blackboard.

Ajoutez un nœud CheckEnemyProximityNode qui vérifie la valeur de EnnemiProche et agit en conséquence.

### Exercice 4 : Construction d'un comportement de patrouille

Construisez un BT qui implémente un comportement de patrouille.

Si un ennemi est détecté, il doit se mettre en mode "Attaque".

S'il ne détecte rien, il doit continuer à patrouiller.

## 4. Conclusion
Les Behavior Trees sont une solution puissante et flexible pour la conception d'IA en jeu vidéo. Avec une bonne implémentation en C++, ils permettent de créer des comportements avancés et facilement modifiables.
