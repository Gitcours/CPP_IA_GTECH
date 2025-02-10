Fichier **`PNJ.hpp`**  
```cpp
#ifndef PNJ_H
#define PNJ_H

#include <SFML/Graphics.hpp>
#include <cmath>

class PNJ {
public:
    sf::Vector2f position;
    float detectionRadius;
    sf::CircleShape circle;
	sf::Vector2f lastPlayerPosition;
    enum State { PATROL, CHASE, SEARCH };
    State currentState;



    PNJ(sf::Vector2f startPos, float radius, float radiusCircle);

    bool detectPlayer(sf::Vector2f playerPos);
    void patrol();
    void chase(sf::Vector2f playerPos);
    void update(sf::Vector2f playerPos, float deltaTime); 
    void search(sf::Vector2f lastPlayerPos, float deltaTime);
};

#endif
```



Fichier **`PNJ.cpp`**  
```cpp
#include "PNJ.hpp"

PNJ::PNJ(sf::Vector2f startPos, float radiusDetect, float radiusCircle) {
    circle.setRadius(radiusCircle);
    circle.setPosition(startPos);
    circle.setFillColor(sf::Color::Red);
    position = startPos;
    detectionRadius = radiusDetect;
	currentState = PATROL;
}

bool PNJ::detectPlayer(sf::Vector2f playerPos) {

    float distance = std::sqrt(std::pow(playerPos.x - position.x, 2) + std::pow(playerPos.y - position.y, 2));
    return (distance < detectionRadius);
}

void PNJ::patrol() {
    static int currentWaypoint = 0;
    static sf::Vector2f waypoints[4] = { sf::Vector2f(100, 300), sf::Vector2f(500, 100), sf::Vector2f(100, 300), sf::Vector2f(500, 300) };
    sf::Vector2f target = waypoints[currentWaypoint];
    sf::Vector2f direction = target - position;
    float distance = std::sqrt(direction.x * direction.x + direction.y * direction.y);

    if (distance < 5.0f) {
        currentWaypoint = (currentWaypoint + 1) % 4;
    }
    else {
        direction /= distance;
        position += direction * 0.2f;
    }
    circle.setPosition(position);
}

void PNJ::chase(sf::Vector2f playerPos) {
    sf::Vector2f direction = playerPos - position;
    float distance = std::sqrt(direction.x * direction.x + direction.y * direction.y);

    if (distance > 0) {
        direction /= distance;
        position += direction * 0.2f;
    }

	circle.setPosition(position);
}

void PNJ::search(sf::Vector2f lastPlayerPos, float deltaTime) {
    static float searchTimer = 0.0f;
    static sf::Vector2f searchDirection;

    if (searchTimer == 0.0f) {
        searchDirection = sf::Vector2f(rand() % 2 == 0 ? -1 : 1, rand() % 2 == 0 ? -1 : 1);
        searchDirection /= std::sqrt(searchDirection.x * searchDirection.x + searchDirection.y * searchDirection.y);
    }

    searchTimer += deltaTime;
    if (searchTimer < 10.0f) {
        position += searchDirection * 5.f * deltaTime;
    }
    else {
        searchTimer = 0.0f;
		currentState = PATROL;
    }

    float distance = std::sqrt((lastPlayerPos.x - position.x) * (lastPlayerPos.x - position.x) + (lastPlayerPos.y - position.y) * (lastPlayerPos.y - position.y));
    if (distance < detectionRadius) {
        searchTimer = 0.0f;
    }

    circle.setPosition(position);
}

void PNJ::update(sf::Vector2f playerPos, float deltaTime) {
    switch (currentState) {
    case PATROL:
        patrol();
        if (detectPlayer(playerPos)) currentState = CHASE;
        break;

    case CHASE:
        chase(playerPos);
        if (!detectPlayer(playerPos)) {
			lastPlayerPosition = playerPos;
            currentState = SEARCH;
        }
        break;

    case SEARCH:
        search(lastPlayerPosition, deltaTime);
        break;
    }
}
```
#include <SFML/Graphics.hpp>
#include <iostream>
#include "PNJ.hpp"

int main() {
    sf::RenderWindow window(sf::VideoMode(800, 600), "Détection PNJ");
    PNJ pnj(sf::Vector2f(400, 300), 100.0f, 10.f);
    sf::CircleShape player(10);
    player.setFillColor(sf::Color::Blue);
    sf::Vector2f playerPos(100, 100);
	sf::Clock clock;
	float deltaTime = 0.0f;

    while (window.isOpen()) {
		deltaTime = clock.restart().asMilliseconds();
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right)) playerPos.x += 0.2f;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left)) playerPos.x -= 0.2f;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down)) playerPos.y += 0.2f;
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up)) playerPos.y -= 0.2f;

        player.setPosition(playerPos);


		pnj.update(playerPos, deltaTime);

        window.clear();
        window.draw(player);
		window.draw(pnj.circle);
        window.display();
    }

    return 0;
}


Fichier **`main.cpp`**  

```cpp

```
