Enlace al .exe (comprimido): 
todo

# El proyecto (proyecto final)
Esto es una practica libre, así que podemos elegir lo que queremos hacer. He decidido hacer un juego multijugador. Al pensar lo que querría hacer, me di cuenta de que ya tengo un juego de la tercera práctica donde el jugador tiene que pasar unas pruebas lo más rápido posible. Eso se podría convertir en un juego con más jugadores para hacer una carrera.

Para este proyecto, he usado la tercera práctica que hicimos en el curso de "desarrollo de videojuegos". Así que tiene el mismo mundo y las mismas pruebas: el jugador tiene que cruzar el agua saltando sobre troncos, escalar un castillo y al fin encontrar el lingote de oro. 

Pero ahora, el jugador podrá empezar su propia sesión o elegir una sesión de alguien diferente en el menú principal. Después, los jugadores se encuentran en el lobby, donde podrán elegir el color de su personaje. El hospedador de la sesión podrá decir cuando empieza el juego con un botón que manda todos los jugadores al nivel con las pruebas, donde hacen una carrera.

----

# Proceso

## Preproducción (Dec 21)

#### Planificación
Para ver si esta idea realmente funciona, tengo que convertir el juego de la practica 3 a un juego que funcione de forma multijugador. También tengo que añadir al fin del juego una lista de los jugadores que ya han terminado las pruebas, ordenado de más rápido a más lento.

Después, puedo ir añadiendo el menú principal para las sesiones, y el lobby para elegir un color y empezar el juego. Ya que en esta práctica no hay una lista con cosas que tiene que tener el juego, podemos ver a donde llegamos y si queda tiempo para añadir más, o para dejar unas cosas.

#### Estética
Nos quedamos con la estética del juego de la practica 3, con el bosque y la nieve. Ahora, porque vamos añadiendo algunos menús (menú principal y lobby), ahí también queremos que la estética se coincide con el juego mismo.  

Porque no sé que difícil será todo esto, me enfocaré en funcionamiento primero. Al sobrar tiempo, intentaré de ir mejorando la estética de los menús.

#### Prototipos
Para esto, había pensado empezar desde un proyecto de Unreal Engine, el [Multiplayer Shootout](https://docs.unrealengine.com/4.27/en-US/Resources/Showcases/BlueprintMultiplayer/). Pero, el proyecto está hecho en la versión 4.24. Por eso fui a buscar tutoriales sobre como hacer un juego multijugador en UE 4.26.

Después de buscar mucho, he encontrado alguien que explica lo que necesito. Tiene [una seria sobre como desarrollar un juego multijugador](https://www.youtube.com/watch?v=GcZQ2o6LpDI&ab_channel=RyanLaley), enseñando cosas como "replicate", la comunicación entre cliente y servidor, "RepNotify",... El mismo creador tiene [una serie sobre como hacer los menús de sesiones](https://www.youtube.com/watch?v=tcVEP2fqYmA).

Por eso, directamente he empezado desde la practica 3, quitándole los menús que ya estaban, para que solo hay el juego limpio para empezar desde cero. Fui intentando las cosas que él enseñaba en sus videos, a veces cosas que no realmente son necesarios para el proyecto, para intentar de entender la lógica de juegos multijugador.  
