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

## Producción

#### El menú final con lista de jugadores (Dec 22 - Jan 3)
Para lograr esto, al acabar el juego (y tocar el lingote de oro) tenemos que guardar el tiempo que ha necesitado el jugador en el PlayerState. Además, también he tenido que mover algunas partes de la lógica que tenía en el ThirdPersonCharacter al PlayerController, porque si no, no funcionaba el código en multijugador. Mover el código al PlayerController es necesario porque solo el cliente local tiene su propio controlador, así que solo ejecutara el código en su controlador (aunque tenemos que tener en cuenta que el servidor tiene acceso a todos los controladores).

Para ver el menú final, también teníamos que cambiar la lógica del lingote de oro, porque ahora si un jugador termina, para todos aparece el menú final. La solución aquí también era mover código, más especifico el evento LevelCompleted, al controlador. Porque el jugador que toca el lingote solamente tiene su propio controlador, y por eso solamente él se podrá ver el menú final. Para hacer que el servidor no lo ve tampoco, controlamos si el controlador es del cliente local con IsLocalPlayerController. 

Además, ya que el lingote se destruye después de tocarlo, por mover el código al controlador, únicamente se destruirá en el mundo del jugador que lo ha tocado. Los demás aún pueden ver el lingote y aún pueden tocarlo para acabar con su juego también. 

Así que por ahora, usando un ListView (lo cual era más complicado de lo que esperaba), el menú final tiene una lista de los jugadores que ya han pasado todas las pruebas.

#### Resolver problemas y acabar con el menú final (Jan 4)
Después de los primeros días trabajando en el menú final, ya está funcionando muy bien aunque aún hay unos problemas causados por el hecho de que estamos en el modo multijugador.

Primero, los tiempos de los otros jugadores siempre están en 0, y solo vemos nuestro tiempo de forma correcta.  Por eso tuve que actualizar el PlayerState en el ThirdPersonCharacter, y llamar el widget en el controlador. Así se muestran todos los tiempos de los jugadores.

Pero aún hay otro problema. Lo que pasa ahora es que desde el momento que se destruye el lingote de oro, ese jugador ya no recibirá las actualizaciones de los otros jugadores que terminan después de él. Para encontrar el problema he tenido que utilizar muchos sprints en todos lados para ver cuál es el problema, si hay algo con la comunicación entre cliente-servidor, ... En multijugador a veces no es tan fácil entender por qué algo no funciona.

[todo: insert screenshot van prints]

Al fin el problema tenía mucho sentido. Al destruir el lingote, ese cliente ya no lo tiene, así que si otros jugadores tocan el lingote, ese "overlap event" ya no pasa en el cliente porque ya no tiene el lingote en su mundo. La solución era poner el lingote en invisible en vez de destruirlo, y asegurarnos de que los "overlap events" solo actualizan el tiempo de los jugadores que todavía no han terminado.

Cada vez que termina un jugador, usamos un "authority switch" y un evento que está llamado por el servidor y NO por los clientes, así que solamente se ejecuta en el servidor, y el servidor avisa a los clientes. (Esta es una opción de "replicate" llamado "run on server"). El evento dice a todos los controladores que tienen que actualizar su lista de jugadores terminados. Así que cada controlador va ordenando la nueva lista de jugadores con sus tiempos, borran el menú final actual, y lo muestran de nuevo con la nueva lista.

La lista de jugadores se encuentra en el GameState, y es un array que no se puede cambiar, así que primero tenemos que copiarlo a otro array. El ordenar se hace con un algoritmo que va comparando los tiempos y usa la función "swap", para intercambiar jugadores en la lista.

#### Mostrar el tiempo en HUD (Jan 4)
Aunque ya tenemos la lista de tiempos de los jugadores, para esos usamos el tiempo del mundo y ya no usamos un contador. La razón es porque a veces el contador se retrasa mucho en comparación con el tiempo del servidor, y era mejor usar el "worldTime". Así que tenemos que cambiar eso también en el HUD. Para esto, usamos un "Timer" de UE, que cada segundo actualiza el texto que está mostrado en el HUD. No utilizamos un "EventTick" porque eso actualizaría demasiadas veces el texto innecesariamente, porque solo mostramos los segundos y no los microsegundos, ...

Algo para notar es que aun los "worldTime" de los jugadores no son exactamente igual, y sería mejor que solamente el servidor guarda el tiempo. Pero al jugar el jugador no se va a dar cuenta, porque no va a estar con 3 ventanas jugando y ver que un cliente está retrasado 0,5 segundos. Por eso, al tener más tiempo sería algo para mejorar, pero no rompe el juego.
