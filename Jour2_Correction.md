Voici la correction des exercices sans les émoticônes :  

---

## **Correction des exercices**

### **Exercice 1 : Génération de niveaux dynamiques**  
**Objectif** : Permettre au joueur d’ajouter/supprimer des obstacles en cliquant sur la grille.  

#### **Modification de `Grid.h`**  
Ajout d'une méthode pour gérer les clics.  

```cpp
void handleClick(int mouseX, int mouseY);
```

#### **Modification de `Grid.cpp`**  
Ajout de la gestion des clics.  

```cpp
void Grid::handleClick(int mouseX, int mouseY) {
    int x = mouseX / CELL_SIZE;
    int y = mouseY / CELL_SIZE;

    if (x >= 0 && x < GRID_WIDTH && y >= 0 && y < GRID_HEIGHT) {
        grid[y][x] = (grid[y][x] == 0) ? 1 : 0;  // Bascule entre obstacle et case vide
    }
}
```

#### **Modification de `main.cpp`**  
Gestion de l’événement de clic souris.  

```cpp
while (window.isOpen()) {
    sf::Event event;
    while (window.pollEvent(event)) {
        if (event.type == sf::Event::Closed)
            window.close();
        if (event.type == sf::Event::MouseButtonPressed) {
            if (event.mouseButton.button == sf::Mouse::Left) {
                grid.handleClick(event.mouseButton.x, event.mouseButton.y);
            }
        }
    }

    window.clear();
    grid.draw(window);
    window.display();
}
```

**Résultat** : On peut maintenant modifier la grille en cliquant.

---

### **Exercice 2 : Modifier l’heuristique pour utiliser la distance Euclidienne**  

#### **Modification de `Node.cpp`**
Changer la fonction `calculateCosts()` pour utiliser la distance Euclidienne :  

```cpp
void Node::calculateCosts(Node* end, int newG) {
    gCost = newG;
    int dx = position.x - end->position.x;
    int dy = position.y - end->position.y;
    hCost = std::sqrt(dx * dx + dy * dy);  // Distance Euclidienne
    fCost = gCost + hCost;
}
```

**Résultat** : L’algorithme utilise maintenant la distance Euclidienne, ce qui peut donner des chemins plus naturels.

---

### **Exercice 3 : Ajouter des déplacements diagonaux**  

#### **Modification de `Pathfinding.cpp`**  
Ajout des déplacements diagonaux dans la liste des voisins.  

```cpp
std::vector<sf::Vector2i> neighbors = {
    {current->position.x + 1, current->position.y},
    {current->position.x - 1, current->position.y},
    {current->position.x, current->position.y + 1},
    {current->position.x, current->position.y - 1},
    {current->position.x + 1, current->position.y + 1},  // Diagonal
    {current->position.x - 1, current->position.y - 1},  // Diagonal
    {current->position.x + 1, current->position.y - 1},  // Diagonal
    {current->position.x - 1, current->position.y + 1}   // Diagonal
};
```

---

### **Exercice 4 : Suivi en temps réel d’un chemin dynamique**  
**Objectif** : Faire en sorte qu’un PNJ suive un chemin même si des obstacles changent.  

#### **Ajout d'un PNJ (`PNJ.hpp`)**  
```cpp
#ifndef PNJ_H
#define PNJ_H

#include <SFML/Graphics.hpp>
#include <vector>
#include "Grid.hpp"

class PNJ {
public:
    sf::Vector2i position;
    std::vector<sf::Vector2i> path;
    int pathIndex;
    sf::Clock moveClock;
    bool needsRepath;

    PNJ(sf::Vector2i start);
    void update(Grid& grid, sf::Vector2i target);
    void draw(sf::RenderWindow& window);
};

#endif

```

#### **Implémentation (`PNJ.cpp`)**
```cpp
#include "PNJ.hpp"
#include "Pathfinding.hpp"

PNJ::PNJ(sf::Vector2i start) : position(start), pathIndex(0) {}

void PNJ::update(Grid& grid, sf::Vector2i target) {
    if (!path.empty() && grid.grid[path[pathIndex].y][path[pathIndex].x] == 1) {
        needsRepath = true;
    }

    if (needsRepath || path.empty()) {
        path = Pathfinding::findPath(grid, position, target);
        pathIndex = 0;
        needsRepath = false;
    }

    if (moveClock.getElapsedTime().asSeconds() < 0.5) {
        return;
    }
    moveClock.restart();

    if (!path.empty() && pathIndex < path.size()) {
        position = path[pathIndex];
        if(pathIndex != path.size()-1) 
            pathIndex++;
    }
}

void PNJ::draw(sf::RenderWindow& window) {
    sf::CircleShape shape(15);
    shape.setPosition(position.x * CELL_SIZE + 10, position.y * CELL_SIZE + 10);
    shape.setFillColor(sf::Color::Red);
    window.draw(shape);
}




```

#### **Modification de `main.cpp`**  
Ajout d’un PNJ qui suit un objectif.  

```cpp
#include <SFML/Graphics.hpp>
#include "PNJ.hpp"

int main() {
    sf::RenderWindow window(sf::VideoMode(GRID_WIDTH * CELL_SIZE, GRID_HEIGHT * CELL_SIZE), "Pathfinding");

    Grid grid;
    PNJ pnj({ 0, 0 });
    sf::Vector2i target(15, 10);

    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();

            if (event.type == sf::Event::MouseButtonPressed) {
                // Ajouter/Supprimer des obstacles avec clic gauche
                if (event.mouseButton.button == sf::Mouse::Left) {
                    grid.handleClick(event.mouseButton.x, event.mouseButton.y);
                    pnj.needsRepath = true; // Demander un recalcul du chemin
                }

                // Définir un nouvel objectif avec clic droit
                if (event.mouseButton.button == sf::Mouse::Right) {
                    target = { event.mouseButton.x / CELL_SIZE, event.mouseButton.y / CELL_SIZE };
                    pnj.needsRepath = true;
                }
            }
        }

        pnj.update(grid, target);

        window.clear();
        grid.draw(window);
        pnj.draw(window);
        window.display();
    }

    return 0;
}

```

**Résultat** : Le PNJ suit le chemin et change de direction si des obstacles apparaissent.

---
