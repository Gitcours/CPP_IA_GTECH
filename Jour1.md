# **Comportements Basiques des PNJ**  

## **Objectifs du module**  
Dans ce module, nous allons explorer les comportements fondamentaux des PNJ (personnages non-joueurs) dans les jeux vidéo. Nous implémenterons les bases de la perception et des réactions des PNJ, ainsi que les comportements de patrouille et de poursuite, en utilisant une **Machine à États Finis (FSM)** pour structurer la logique d’IA.  

À la fin de ce module, vous serez capables de :  
- Comprendre comment les PNJ perçoivent leur environnement.  
- Implémenter des comportements de patrouille et de poursuite en C++ avec SFML.  
- Construire une **FSM** pour gérer les états d’un PNJ.  

---

# **1. Perception et Réaction des PNJ**  

Un PNJ doit être capable de détecter son environnement avant d’adopter un comportement. Il peut utiliser différents types de perception comme la vision et l’audition.  

### **1.1 Détection du joueur par la distance**  

Le moyen le plus simple de détecter un joueur est de vérifier s’il se trouve dans un rayon donné.  

#### **Code : Détection de proximité**
Fichier **`PNJ.h`**  
```cpp
#ifndef PNJ_H
#define PNJ_H

#include <SFML/Graphics.hpp>
#include <cmath>

class PNJ {
public:
    sf::Vector2f position;
    float detectionRadius;

    PNJ(sf::Vector2f startPos, float radius);

    bool detectPlayer(sf::Vector2f playerPos);
};

#endif
```

Fichier **`PNJ.cpp`**  
```cpp
#include "PNJ.h"

PNJ::PNJ(sf::Vector2f startPos, float radius) {
    position = startPos;
    detectionRadius = radius;
}

bool PNJ::detectPlayer(sf::Vector2f playerPos) {
    float distance = std::sqrt(std::pow(playerPos.x - position.x, 2) + std::pow(playerPos.y - position.y, 2));
    return (distance < detectionRadius);
}
```

Fichier **`main.cpp`**  
```cpp
#include <SFML/Graphics.hpp>
#include <iostream>
#include "PNJ.h"

int main() {
    sf::RenderWindow window(sf::VideoMode(800, 600), "Détection PNJ");
    PNJ pnj(sf::Vector2f(400, 300), 100.0f);
    sf::CircleShape player(10);
    player.setFillColor(sf::Color::Blue);
    sf::Vector2f playerPos(100, 100);

    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right)) playerPos.x += 2;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left)) playerPos.x -= 2;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down)) playerPos.y += 2;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up)) playerPos.y -= 2;

        player.setPosition(playerPos);

        if (pnj.detectPlayer(playerPos)) {
            std::cout << "Joueur détecté !" << std::endl;
        }

        window.clear();
        window.draw(player);
        window.display();
    }

    return 0;
}
```

#### **Exercice 1 : Développement d'un PNJ en sentinelle**  
Modifiez le PNJ pour qu’il **reste immobile** et change de couleur lorsqu'il détecte le joueur.  

---

# **2. Comportements de Base des PNJ**  

### **2.1 Comportement de Patrouille**  
Un PNJ en patrouille suit un chemin prédéfini et change de direction à des points clés.

Fichier **`PNJ.h`**  
Ajout d'une méthode de patrouille :  
```cpp
void patrol();
```

Fichier **`PNJ.cpp`**  
Ajout de l’implémentation de la patrouille :  
```cpp
void PNJ::patrol() {
    static int currentWaypoint = 0;
    static sf::Vector2f waypoints[2] = { sf::Vector2f(100, 300), sf::Vector2f(500, 300) };
    sf::Vector2f target = waypoints[currentWaypoint];
    sf::Vector2f direction = target - position;
    float distance = std::sqrt(direction.x * direction.x + direction.y * direction.y);

    if (distance < 5.0f) { 
        currentWaypoint = (currentWaypoint + 1) % 2;
    } else {
        direction /= distance;
        position += direction * 2.0f;
    }
}
```

#### **Exercice 2 : Développement d'un PNJ avec plusieurs points de patrouille**  
Modifiez le PNJ pour qu’il **puisse patrouiller sur un chemin défini par plus de deux points**.  

---

### **2.2 Comportement de Poursuite**  

Si un PNJ détecte le joueur, il doit cesser de patrouiller et le poursuivre.

Fichier **`PNJ.h`**  
Ajout d'une méthode de poursuite :  
```cpp
void chase(sf::Vector2f playerPos);
```

Fichier **`PNJ.cpp`**  
Ajout de l’implémentation de la poursuite :  
```cpp
void PNJ::chase(sf::Vector2f playerPos) {
    sf::Vector2f direction = playerPos - position;
    float distance = std::sqrt(direction.x * direction.x + direction.y * direction.y);

    if (distance > 0) {
        direction /= distance; 
        position += direction * 2.0f;
    }
}
```

#### **Exercice 3 : Développement d'un PNJ qui alterne entre patrouille et poursuite**  
Créez un PNJ qui **patrouille en temps normal**, mais qui **poursuit le joueur lorsqu’il le détecte**.  

---

# **3. Machines à États Finis (FSM) pour les PNJ**  

Une **FSM (Finite State Machine)** permet de structurer les comportements des PNJ en différents **états**.

Fichier **`PNJ.h`**  
Ajout de l’énumération des états et de la gestion de l’état actuel :  
```cpp
enum State { PATROL, CHASE, SEARCH };
State currentState;

void update(sf::Vector2f playerPos);
```

Fichier **`PNJ.cpp`**  
Implémentation de la FSM :  
```cpp
void PNJ::update(sf::Vector2f playerPos) {
    switch (currentState) {
        case PATROL:
            patrol();
            if (detectPlayer(playerPos)) currentState = CHASE;
            break;

        case CHASE:
            chase(playerPos);
            if (!detectPlayer(playerPos)) {
                currentState = SEARCH;
            }
            break;

        case SEARCH:
            break;
    }
}
```

#### **Exercice 4 : Développement d'un PNJ qui cherche après avoir perdu le joueur**  
Modifiez le PNJ pour qu’après avoir perdu le joueur, **il cherche dans la zone pendant quelques secondes avant de reprendre sa patrouille**.  

---

# **4. Conclusion et Prochaines Étapes**  

Nous avons appris à :  
- Comprendre la perception des PNJ.  
- Implémenter des comportements de patrouille et de poursuite.  
- Structurer l’IA d’un PNJ avec une **FSM**.  

Prochain module :  
Nous verrons **les comportements avancés**, notamment :  
- L’évitement d’obstacles.  
- Les tactiques d’attaque en groupe.  
- Les stratégies de contournement et d’embuscade.
