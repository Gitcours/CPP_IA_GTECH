# **Pathfinding et Navigation**  

## **Objectifs du module**  
Dans ce module, nous allons explorer les concepts de **pathfinding** et de **navigation**, essentiels pour permettre aux PNJ de se déplacer intelligemment dans un environnement. Nous implémenterons l’algorithme **A*** (A-star), qui est une des méthodes les plus utilisées pour la recherche de chemins dans les jeux vidéo.

À la fin de ce module, vous serez capables de :  
- Comprendre les bases du pathfinding et son importance dans les jeux vidéo.  
- Implémenter l’algorithme **A*** en C++ avec SFML.  
- Faire naviguer un PNJ vers un objectif en évitant les obstacles.  

---

# **1. Introduction au Pathfinding**  

Le **pathfinding** permet de calculer un chemin entre un point de départ et un point d’arrivée, en prenant en compte les obstacles.  

Les méthodes classiques de pathfinding incluent :  
- **Parcours en ligne droite** (naïf, ne tient pas compte des obstacles).  
- **Parcours en grille** (divise l’espace en cases et calcule un chemin).  
- **Graphes de navigation** (représente les points importants sous forme de graphe).  
- **Algorithme A*** (utilisé dans les jeux pour un pathfinding efficace).  

---

# **2. Représentation de la Carte sous forme de Grille**  

Avant d’implémenter **A***, nous devons représenter l’environnement sous forme de **grille** où chaque case est soit traversable, soit un obstacle.

### **2.1 Structure de la grille**  

Fichier **`Grid.h`**  
```cpp
#ifndef GRID_H
#define GRID_H

#include <vector>
#include <SFML/Graphics.hpp>

const int GRID_WIDTH = 20;
const int GRID_HEIGHT = 15;
const int CELL_SIZE = 40;

class Grid {
public:
    std::vector<std::vector<int>> grid;

    Grid();
    void draw(sf::RenderWindow& window);
};

#endif
```

Fichier **`Grid.cpp`**  
```cpp
#include "Grid.h"

Grid::Grid() {
    grid.resize(GRID_HEIGHT, std::vector<int>(GRID_WIDTH, 0));

    // Définition de quelques obstacles
    for (int i = 5; i < 15; i++) {
        grid[7][i] = 1;
    }
}

void Grid::draw(sf::RenderWindow& window) {
    for (int y = 0; y < GRID_HEIGHT; y++) {
        for (int x = 0; x < GRID_WIDTH; x++) {
            sf::RectangleShape cell(sf::Vector2f(CELL_SIZE, CELL_SIZE));
            cell.setPosition(x * CELL_SIZE, y * CELL_SIZE);
            cell.setOutlineThickness(1);
            cell.setOutlineColor(sf::Color::Black);

            if (grid[y][x] == 1) {
                cell.setFillColor(sf::Color::Black);
            } else {
                cell.setFillColor(sf::Color::White);
            }

            window.draw(cell);
        }
    }
}
```

Fichier **`main.cpp`**  
Ajout de la grille dans le jeu :  
```cpp
#include <SFML/Graphics.hpp>
#include "Grid.h"

int main() {
    sf::RenderWindow window(sf::VideoMode(GRID_WIDTH * CELL_SIZE, GRID_HEIGHT * CELL_SIZE), "Pathfinding");

    Grid grid;

    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        window.clear();
        grid.draw(window);
        window.display();
    }

    return 0;
}
```

#### **Exercice 1 : Génération de niveaux dynamiques**  
Modifiez le programme pour **permettre au joueur de cliquer pour ajouter/supprimer des obstacles sur la grille**.

---

# **3. Implémentation de l'algorithme A\***  

L’algorithme A* (A Star) est l’un des algorithmes de pathfinding les plus efficaces utilisés en jeu vidéo. Il permet à un PNJ de trouver le chemin le plus court entre un point de départ et une destination tout en évitant les obstacles.  

---

## **3.1. Principe de fonctionnement**  

A* repose sur une combinaison de deux concepts :  
- **Recherche en profondeur** qui explore tous les chemins possibles.  
- **Heuristique** qui permet d’orienter la recherche vers l’objectif et d’éviter les chemins inutiles.  

L’algorithme utilise trois valeurs pour chaque case :  
- **G(n) :** Distance entre le point de départ et le nœud actuel.  
- **H(n) :** Distance estimée entre le nœud actuel et la destination (utilisation d’une heuristique).  
- **F(n) :** Somme des deux valeurs :  
  \[
  F(n) = G(n) + H(n)
  \]
  Cette valeur est utilisée pour prioriser les cases à explorer.  

A* fonctionne en explorant les cases avec la plus petite valeur de **F(n)**, ce qui permet de trouver un chemin optimal plus rapidement.  

---

## **3.2. Heuristiques utilisées**  

L’heuristique est une estimation de la distance entre un nœud et l’objectif. Différentes méthodes existent :  
- **Distance de Manhattan** (adaptée aux grilles sans diagonale) :  
  \[
  H(n) = |x_objectif - x_nœud| + |y_objectif - y_nœud|
  \]  
- **Distance Euclidienne** (adaptée aux déplacements diagonaux) :  
  \[
  H(n) = \sqrt{(x_{\text{objectif}} - x_{\text{nœud}})^2 + (y_{\text{objectif}} - y_{\text{nœud}})^2}
  \]  
- **Distance de Chebyshev** (lorsque les déplacements diagonaux coûtent le même prix que les orthogonaux) :  
  \[
  H(n) = \max(|x_{\text{objectif}} - x_{\text{nœud}}|, |y_{\text{objectif}} - y_{\text{nœud}}|)
  \]  

Dans un environnement en grille classique, on utilise souvent la distance de Manhattan.  

---

## **3.3. Fonctionnement détaillé de l’algorithme**  

### **Étapes principales de A***  
1. **Ajouter le nœud de départ à la liste ouverte (open list).**  
2. **Boucler tant que la liste ouverte n’est pas vide :**  
   - Récupérer le nœud avec le plus petit F(n) de la liste ouverte.  
   - Le déplacer vers la liste fermée (nœuds déjà explorés).  
   - Vérifier si ce nœud est la destination : si oui, reconstruire le chemin et terminer.  
   - Générer les nœuds voisins et pour chaque voisin :  
     - Ignorer les obstacles ou les nœuds déjà explorés.  
     - Calculer **G(n), H(n) et F(n)**.  
     - Si le voisin est déjà dans la liste ouverte avec un meilleur G(n), ignorer cette nouvelle valeur.  
     - Sinon, mettre à jour le nœud et l’ajouter à la liste ouverte.  

---

## **3.4. Implémentation complète en C++**  

Nous allons maintenant implémenter l’algorithme A* en C++.  

### **Définition d’un nœud pour A***  

#### **Fichier `Node.h`**
```cpp
#ifndef NODE_H
#define NODE_H

#include <SFML/System/Vector2.hpp>

struct Node {
    sf::Vector2i position;
    int gCost, hCost, fCost;
    Node* parent;

    Node(sf::Vector2i pos);
    void calculateCosts(Node* end, int newG);
};

#endif
```

#### **Fichier `Node.cpp`**
```cpp
#include "Node.h"
#include <cmath>

Node::Node(sf::Vector2i pos) : position(pos), gCost(0), hCost(0), fCost(0), parent(nullptr) {}

void Node::calculateCosts(Node* end, int newG) {
    gCost = newG;
    hCost = std::abs(position.x - end->position.x) + std::abs(position.y - end->position.y);
    fCost = gCost + hCost;
}
```

---

### **Implémentation de l’algorithme A***  

#### **Fichier `Pathfinding.h`**
```cpp
#ifndef PATHFINDING_H
#define PATHFINDING_H

#include "Grid.h"
#include "Node.h"
#include <vector>

class Pathfinding {
public:
    static std::vector<sf::Vector2i> findPath(Grid& grid, sf::Vector2i start, sf::Vector2i end);
};

#endif
```

#### **Fichier `Pathfinding.cpp`**
```cpp
#include "Pathfinding.h"
#include <queue>
#include <algorithm>

std::vector<sf::Vector2i> Pathfinding::findPath(Grid& grid, sf::Vector2i start, sf::Vector2i end) {
    std::vector<std::vector<bool>> visited(GRID_HEIGHT, std::vector<bool>(GRID_WIDTH, false));
    std::vector<Node*> openList;
    std::vector<Node*> allNodes;
    
    Node* startNode = new Node(start);
    Node* endNode = new Node(end);
    openList.push_back(startNode);
    allNodes.push_back(startNode);

    while (!openList.empty()) {
        std::sort(openList.begin(), openList.end(), [](Node* a, Node* b) { return a->fCost < b->fCost; });
        Node* current = openList.front();
        openList.erase(openList.begin());

        if (current->position == end) {
            std::vector<sf::Vector2i> path;
            while (current) {
                path.push_back(current->position);
                current = current->parent;
            }
            std::reverse(path.begin(), path.end());
            return path;
        }

        visited[current->position.y][current->position.x] = true;

        std::vector<sf::Vector2i> neighbors = {
            {current->position.x + 1, current->position.y},
            {current->position.x - 1, current->position.y},
            {current->position.x, current->position.y + 1},
            {current->position.x, current->position.y - 1}
        };

        for (sf::Vector2i& neighborPos : neighbors) {
            if (neighborPos.x < 0 || neighborPos.x >= GRID_WIDTH || neighborPos.y < 0 || neighborPos.y >= GRID_HEIGHT)
                continue;
            if (grid.grid[neighborPos.y][neighborPos.x] == 1 || visited[neighborPos.y][neighborPos.x])
                continue;

            Node* neighbor = new Node(neighborPos);
            neighbor->parent = current;
            neighbor->calculateCosts(endNode, current->gCost + 1);
            openList.push_back(neighbor);
            allNodes.push_back(neighbor);
        }
    }

    for (Node* node : allNodes)
        delete node;

    return {};
}
```

---

## **3.5. Intégration et Test**  

Ajoutez ce pathfinding dans un projet SFML pour afficher le chemin trouvé sur une grille et permettre au joueur de définir un point de départ et une destination en cliquant sur l’écran.  

---

## **3.6. Exercices**  

1. **Modifier l’heuristique** pour utiliser la distance euclidienne au lieu de la distance de Manhattan.  
2. **Ajouter des déplacements diagonaux** dans l’algorithme.  
3. **Faire en sorte qu’un PNJ suive en temps réel un chemin dynamique** (par exemple, si des obstacles sont ajoutés).  


---

# **4. Conclusion et Prochaines Étapes**  

Nous avons appris à :  
- Construire une grille de navigation.  
- Implémenter l’algorithme **A*** pour trouver un chemin.  
- Appliquer le pathfinding à un PNJ.  

Prochain module : **Évitement d’obstacles et déplacements avancés.**
