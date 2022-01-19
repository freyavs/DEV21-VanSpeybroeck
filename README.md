TODO: screenshot spel


Enlace al .exe (comprimido): 
https://drive.google.com/file/d/1-nsPY7wLYv1z1CdcQywhrtV9cdF517tH/view?usp=sharing

##### Importante: como probar el juego con varios clientes:
- Jugar desde el editor, abriendo el juego con "new editor window (PIE)". Puedes elegir tantos clientes como quieras, y seleccionas "Play as Client".
- Lo mismo, pero abriéndolo en "Standalone game". El problema aquí es que es mucho más pesado para tu ordenador que la primera opción. 
- Presionando "esc" o "p" puedes salir con tu cursor de la ventana y ir a otra. (Aunque "esc" no funcione en la opcion de "new editor window (PIE)".

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

Al fin el problema tenía mucho sentido. Al destruir el lingote, ese cliente ya no lo tiene, así que si otros jugadores tocan el lingote, ese "overlap event" ya no pasa en el cliente porque ya no tiene el lingote en su mundo. La solución era poner el lingote en invisible en vez de destruirlo, y asegurarnos de que los "overlap events" solo actualizan el tiempo de los jugadores que todavía no han terminado.

Cada vez que termina un jugador, usamos un "authority switch" y un evento que está llamado por el servidor y NO por los clientes, así que solamente se ejecuta en el servidor, y el servidor avisa a los clientes. (Esta es una opción de "replicate" llamado "run on server"). El evento dice a todos los controladores que tienen que actualizar su lista de jugadores terminados. Así que cada controlador va ordenando la nueva lista de jugadores con sus tiempos, borran el menú final actual, y lo muestran de nuevo con la nueva lista.

La lista de jugadores se encuentra en el GameState, y es un array que no se puede cambiar, así que primero tenemos que copiarlo a otro array. El ordenar se hace con un algoritmo que va comparando los tiempos y usa la función "swap", para intercambiar jugadores en la lista.

#### Mostrar el tiempo en HUD (Jan 4)
Aunque ya tenemos la lista de tiempos de los jugadores, para esos usamos el tiempo del mundo y ya no usamos un contador. La razón es porque a veces el contador se retrasa mucho en comparación con el tiempo del servidor, y era mejor usar el "worldTime". Así que tenemos que cambiar eso también en el HUD. Para esto, usamos un "Timer" de UE, que cada segundo actualiza el texto que está mostrado en el HUD. No utilizamos un "EventTick" porque eso actualizaría demasiadas veces el texto innecesariamente, porque solo mostramos los segundos y no los microsegundos, ...

Algo para notar es que aun los "worldTime" de los jugadores no son exactamente igual, y sería mejor que solamente el servidor guarda el tiempo. Pero al jugar el jugador no se va a dar cuenta, porque no va a estar con 3 ventanas jugando y ver que un cliente está retrasado 0,5 segundos. Por eso, al tener más tiempo sería algo para mejorar, pero no rompe el juego.

#### Actualizar las pócimas para multijugador (Jan 5)
Las pócimas tienen que funcionar igual que el lingote de oro. Ya que ahora se destruyen y después se aparecen de nuevo, ya sabemos que no va a funcionar por la misma razón que en el lingote de oro.

Aunque aquí parecía que al principio funciono con el destruir del objeto, al fin si había un problema. Para cambiar la velocidad y la altura de saltar el servidor tiene que darse cuenta de que ha cambiado. Porque si no, vemos que el jugador, cuando está corriendo, hay como "glitches" y no se puede mover más rápido porque el servidor no sabe que el jugador puede ir tan rápido.

Así que aquí también la solución es solo volver las pócimas invisibles, y hacer que no se puede aumentar más las habilidades si ya están aumentadas, porque aún habrá eventos de "overlap".  

Arreglar que funcionan las pócimas y el lingote de oro de para el multijugador era la parte más difícil de este proyecto. Ya que la siguiente parte tiene un tutorial muy bien explicado (para la mayoría de las cosas que necesitaba), pero para arreglar esto no podio encontrar algo similar y tenía que intentar mucho con prueba y error.

#### Menú principal (Jan 7)
Desde aquí, empecé a seguir el tutorial sobre sesiones que he puesto al principio. Hemos empezado con crear un nivel que se llama MainMenu donde entramos al abrir el juego. También necesitamos un OnlineGameInstance para poner algunas partes de la lógica del multijugador y las sesiones.

##### Los widgets
Para empezar, necesitamos los widgets del menú con botones para empezar o unirte a una sesión. 

El menu de empezar un sesion, es solo un boton. Pero para buscar sesiones necesitamos una forma de lista. Antes hemos usado el ListView para hacer una lista, pero el tutorial lo hizo con un VerticalBox. Me pareció mucho más fácil de esa manera que con el ListView. El VerticalBox lo llenamos con resultados de una búsqueda, y los resultados también lo ponemos en un widget separado que se llama SessionResult.

##### Empezar, buscar y unirte a sesiones de LAN
Ya que UE ya tiene funciones para todo esto, es únicamente usar StartSession, SearchSessions y JoinSession. Era mucho más fácil de lo que había esperado. Las funciones como "hostSession" después de presionar el botón "Host match" las ponemos en OnlineGameInstance. Al empezar una sesión el juego tiene que mandarte a otro nivel, el Lobby. Después los jugadores que se unen a tu sesión automáticamente se van al mismo Lobby.

Aquí tenía que decidir si quiero hacer las sesiones por LAN o por Steam. Pero, ya que no tengo dos laptops para probar el juego, decidí usar LAN. Tampoco importa mucho, porque no es muy difícil conectarlo con Steam y funciona de la misma manera que LAN, solamente que tienes que cambiar la configuración del fichero DefaultEngine.ini.

#### Menú Lobby (Jan 10)
Para entrar en el lobby, en OnlineGameInstance tenemos unas funciones hostSession y joinSession. El hostSession tiene que abrir el nivel de Lobby, y el joinSession automáticamente se une al servidor.

##### Diseño del nivel
Aquí he seguido el diseño del tutorial, a la derecha hay una venta donde entran los jugadores y vemos los personajes. A la izquierda hay el menú widget, donde los jugadores podrán ver la lista de jugadores. Es importante ponder la visibilidad del canvas en "visible" y no en "not hit-testable" para que el jugador no puede entrar en la ventana a la derecha.

Tuve que añadir un LobbyController y un LobbyCharacter para que no entra la lógica del juego en el nivel del Lobby. En el nivel ponemos 5 PlayerSpawns y una cámara fija desde donde los jugadores pueden ver el Lobby.

##### Botón para empezar el juego
Aquí necesitamos ejecutar un comando que hace un "servertravel" al nivel con el castillo.

##### Lista de jugadores
En el lobby también queremos mostrar la lista de jugadores que ha entrado. Para eso necesitamos guardar una lista de estructuras (PlayerInfo) que guardan alguna información sobre el jugador, como el color que ha elegido y su nombre.

Cuando entra un jugador, hay una función "onPostLogin" que lo detecta. Ahí tenemos que ir añadiendo el jugador a la lista de los jugadores conectados, y hacer una lista de PlayerInfo de los mismos jugadores conectados. Aquí también tenemos que usar comunicación adecuada entre los clientes y el servidor.

##### Elegir color
Ya que no hay un widget para elegir un color, puse 3 controles deslizantes para cada los canales RGB. Aquí también era un poco difícil hacer que estos eventos se comuniquen bien, porque al principio solamente cambiar el color del jugador servidor se mostraba en la lista. La solución era llamar a una función de "run on Server" desde el widget y no desde el controlador. Así que al cambiar el color, desde el widget llamamos a una función que actualiza el nuevo color en el servidor también.

Al salir del lobby y entrar el juego el personaje tiene que tener ese color. Eso lo hacemos por guardar el color en el GameState (ya que este se queda por todos los niveles). Al principio solamente el jugador local podía ver su cambio de color - lo que hacemos poniéndole un material al jugador su malla y el material tiene un parámetro de vector que indica el color. Lo que teníamos que hacer es desde el controlador, llamar a una función del ThirdPersonCharacter con la opción "run on Server" que asigna el color con un RepNotify para que todos los clientes lo "escuchan".

#### Jugador no puede entrar una sesión ya empezada (Jan 14)
Primero, ese jugador no debería encontrar la sesión, así que el jugador servidor tiene que destruir la sesión. Si alguien ha encontrado esta sesión antes, aún podrá unirse, así que tenemos que averiguar cuando un jugador entra, si no está entrando una sesión ya empezada.

El ThirdPersonGameMode se da cuenta de que alguien ha entrado, y ahí averiguamos si no hay demasiados jugadores (más de lo esperado). Al GameMode le puedes dar unos variables, así que al empezar le podemos pasar cuantos jugadores están en la sesión al principio. Cuando quiere un jugador más, veremos que es más del inicio y tenemos que echar el jugador del juego.

#### Mejorar los widgets (Jan 14)
Para finalizar los widgets, tuve que aun añadir el código para el botón de salir. También he querido limpiar un poco el diseño de los widgets, como darle colores a los controles deslizantes del lobby, jugar un poco con las proporciones de los otros menús, ...

Además, querría añadir unas pantallas mientras está cargando el juego. Primero, para mostrar que estamos buscando sesiones, ya que antes no mostraba que estaba buscando así que eso puede confundir los usuarios. 

Al mandar los jugadores al juego desde el lobby, el servidor puede ver los jugadores desapareciendo. Por eso añadí una pantalla de carga cuando el servidor presiona el botón para empezar el juego (y en los clientes no, aunque podría ser bien añadir eso también).

#### Botones para salir e ir al menú principal (Jan 18)
Implementar el botón de salir del juego era muy simple, pero el botón para volver al menú principal no tanto. 

Solamente abrir el nivel del menú principal no era suficiente, porque después el jugador cliente ya no podía empezar o unirse a otra sesión. Así que al entrar el menú principal, tenemos que llamar a "destroySession" para que salga de su sesión antigua.

Esto funciona para el jugador cliente, pero si sale el jugador servidor (al salir del juego o volver al menú principal), el juego se cierra para todos con un error. Por eso, antes de que el servidor se vaya, tenemos que mandar a todos los clientes al menú principal. En ese caso también queremos que salga un error "Host has left the game". 

Aquí otra vez tenemos que usar el AuthoritySwitch que nos facilita dar otro compartimiento al servidor que al cliente al presionar "salir" o "volver a jugar". En el caso del servidor, tiene que buscar todos los controladores (menos el controlador del servidor) y hacer que llaman a su "leaveSessionInProgress", pasando un "optionString" al abrir del nivel del menú principal. Usando en el MainMenuGameMode un "switchOnString" podemos hacer que salga un error cuando el "optionString" es igual a "hostLeft". 

Después de mandar todos los jugadores al menú principal, necesitamos un poco de retraso antes de que el servidor se vaya, porque si no, no hay suficiente tiempo para mandar todos los jugadores con éxito al menú principal.

Queda un único problema, cuando el jugador se va del juego, ya no está en la lista de jugadores y ya no sale en el menú final. Aunque tendría sentido, sería mejor guardarlo en una lista global. Aunque eso significa que tendríamos que cambiar mucho del sistema de como se hace ahora la lista final, así que lo dejamos para una versión futura.

#### Menú de pausa (Jan 18)
Al presionar "esc" o "p" el jugador puede ver los botones que hemos implementado arriba, "salir" y "volver a jugar". Aunque no realmente es un menú de pausa, porque estamos haciendo una carrera, así que el tiempo sigue contando.


## Postproducción (Jan 15)
#### Pruebas
Para probar el juego, he puesto un lingote de oro ya entes del laberinto para poder hacer las pruebas fácilmente. Las pruebas las hice igual a las pruebas que muestro en el video de la entrega.

En el menú principal, no me daba cuenta de como probar que sale el error cuando hay un problema con las sesiones en el menú principal. Porque aunque ponía el ordenador en modo, algunas cosas seguían funcionando porque estamos en LAN y no en una conexión de Steam. 

#### Botones para salir e ir al menú principal (Jan 18)
Implementar el botón de salir del juego era muy simple, pero el botón para volver al menú principal no tanto. 

Solamente abrir el nivel del menú principal no era suficiente, porque después el jugador cliente ya no podía empezar o unirse a otra sesión. Así que al entrar el menú principal, tenemos que llamar a "destroySession" para que salga de su sesión antigua.

Esto funciona para el jugador cliente, pero si sale el jugador servidor (al salir del juego o volver al menú principal), el juego se cierra para todos con un error. Por eso, antes de que el servidor se vaya, tenemos que mandar a todos los clientes al menú principal. En ese caso también queremos que salga un error "Host has left the game". 

Aquí otra vez tenemos que usar el AuthoritySwitch que nos facilita dar otro compartimiento al servidor que al cliente al presionar "salir" o "volver a jugar". En el caso del servidor, tiene que buscar todos los controladores (menos el controlador del servidor) y hacer que llaman a su "leaveSessionInProgress", pasando un "optionString" al abrir del nivel del menú principal. Usando en el MainMenuGameMode un "switchOnString" podemos hacer que salga un error cuando el "optionString" es igual a "hostLeft". 

Después de mandar todos los jugadores al menú principal, necesitamos un poco de retraso antes de que el servidor se vaya, porque si no, no hay suficiente tiempo para mandar todos los jugadores con éxito al menú principal.


Queda un único problema, cuando el jugador se va del juego, ya no está en la lista de jugadores y ya no sale en el menú final. Aunque tendría sentido, sería mejor guardarlo en una lista global. Aunque eso significa que tendríamos que cambiar mucho del sistema de como se hace ahora la lista final, así que lo dejamos para una versión futura.

#### Menú de pausa (Jan 18)
Al presionar "esc" o "p" el jugador puede ver los botones que hemos implementado arriba, "salir" y "volver a jugar". Aunque no realmente es un menú de pausa, porque estamos haciendo una carrera, así que el tiempo sigue contando.


#### Mejoras después de probar el juego

##### Cámara del lobby
La cámara en el lobby funcionaba para el servidor o el host, pero cuando entraba un cliente después de más o menos un segundo la cámara estaba yendo al cuerpo del jugador. Para hacer que no se mueva la cámara al entrar en el lobby, tuve que desactivar la opción "auto manage active camera" en el controlador.

#### Colores por defecto
Al probar un poco el lobby, me di cuenta de que cuando un cliente no elige un color, el servidor lo verá como negro. Por eso, al entrar en el lobby ya le tenemos que dar un color predeterminado a los jugadores - en este caso blanco/gris.

#### Problemas con los tiempos finales en el menú final
Primero, el tiempo final arriba en el menú y lo que sale en la lista no era igual. Eso fue porque estábamos usando el tiempo del mundo y no el tiempo que tenía el jugador en su PlayerState, así que podría ser que esos dos tiempos tenían un poquito de diferencia porque no se guardan al mismo tiempo.

Y de alguna forma no me di cuenta hasta muy tarde que los tiempos finales aún se estaban actualizando al tocar el oro invisible, así que tuve que arreglar eso también.

#### Cursor e input del menú final
Al llegar al fin, el jugador no puede tocar los botones porque su input sigue siendo solamente del juego. Así que al tocar el oro se tiene que mostrar el cursor y tener input del UI. También añadí que presionando el botón "esc" o "p" que puedes dejar aparecer y desaparecer el menú final.

#### Actualización de la lista de jugadores en el menú final
Al jugar el juego en con varios clientes de forma "Standalone game", el juego va muy lento porque abrir esas ventanas en un solo ordenador es muy pesado. Pero, por eso me di cuenta de que no siempre todos los jugadores que han terminado aparecen en la lista en el menú final. Pero, puede ser que al llamar la función que actualiza las listas, ese cliente todavía no se ha dado cuenta del nuevo jugador que haya terminado. Por eso, cuando actualizamos la lista, pero no añadimos un nuevo jugador, significa que aún estamos esperando a conseguir el tiempo final del jugador que acaba de terminar. En ese caso tenemos que actualizar la lista de nuevo hasta que tenemos más jugadores que la lista de antes.

La verdad es que esta parte, al fin, pienso que podría tener más sentido usar algo más optimal para multijugador. Pero al hacer esta parte todavía estaba experimentando mucho con como funcionaba todo en multijugador. Puede ser que lo he planeado mal y podría haber sido mejor primero jugar un poco con las sesiones en vez de convertir el juego.

----

# Diseño del juego
## Jugabilidad

### Mecánicas

**Salud**
- El jugador se muere con un solo golpe o toque de un objeto enemigo.
- Saltando en el agua matará al jugador.
- El jugador se muere si no llega suficiente rápido desde el inicio hasta el fin del laberinto.
- El jugador puede pasar un "checkpoint" para cambiar su ubicación de reaparición.

**Movimientos**
- El jugador puede soltar y maniobrarse para evitar objetos.
- El jugador puede soltar más alto si toma una pócima roja.
- El jugador puede correr más rápido si toma una pócima amarilla.

**Menús**
- Al entrar al juego, el jugador se encuentra en el menú inicial donde puede empezar o unirse a una sesión.
- Después de unirse a una sesión, hay un menú de lobby. Ahí el jugador puede cambiar el color de su personaje. El jugador servidor puede empezar la carrera, mandando los jugadores al nivel con el castillo.
- Presionando "esc" o "p" el jugador verá un menú pequeño donde puede ir de nuevo al menú principal o salir del juego.
- Al acabar el juego el jugador ve un menú final, con los tiempos de los otros jugadores que ya se han terminado la carrera. La lista de jugadores se va actualizando mientras más jugadores llegan al fin.
- Presionando "esc" o "p" en el menú final alterna si puede ver el menú o no.

### Dinámica
- El jugador necesita evitar o los objetos enemigos para que no se muera.
- El jugador necesita hacer sus movimientos (saltos, …) cuidadosamente para pasarlas pruebas y obstáculos del nivel.
- El jugador tiene que tomar la pócima roja para pasar la prueba de saltos (escalar la muralla hasta llegar adentro).
- El jugador necesita la pócima amarilla para llegar al fin del laberinto suficiente rápido.
- El jugador debería llegar al lingote de oro para acabar con el juego, intentando de acabar antes de los otros jugadores.
- El jugador puede intentar de llegar lo más rápido posible, tiene un temporizador para ver su tiempo.

### Estética
- El juego se ve muy real, usando paisajes, árboles, ... y fondos, para que el jugador se siente en un mundo que va más allá.
- Hay neblina y nieve en el entorno para qué se ve un poco más misterioso.
- El temporizador en el HUD debería darle un poco de estrés y sentimiento de prisa al jugador.
- Todos los jugadores pueden elegir un color para poder diferenciar entre los jugadores.

## Contenido

### Los enemigos
Los enemigos del jugador son objetos. Hay dos tipos:
- Objetos móviles: Hay rocas en forma de bolas que se caen por la muralla, y después ruedan hacia el agua. El jugador no puede tocarlas o se muere y va al último punto de control. También hay los cubos de piedras que se mueven de un lado al otro, que pueden matar al jugador y mandarle al último punto de control.
- Objetos estáticos: Son objetos que no se mueven, pero también pueden matar al jugador, como el agua o las cuerdas adentro del castillo.
- Los otros jugadores: aunque no se pueden matar el uno al otro, si deberías ver los otros jugadores como un enemigo porque quieres intentar de llegar al fin antes de ellos.

Además, hay una prueba especial que matara al jugador si no llega en un tiempo predeterminado. Aquí, el tiempo es el enemigo.

### Aumentación de las habilidades
- Las pócimas rojas: incrementan por unos segundos la altura de saltos del jugador
- Las pócimas amarillas: incrementan por unos segundos la velocidad del jugador

### Los niveles
Hay un nivel en donde los jugadores hacen una carrera, con varias pruebas, que se puede encontrar aquí: https://github.com/freyavs/DEV21-VanSpeybroeck/wiki/Las-pruebas

### Los menus
Los menus (Menu Principal y Lobby) los muestro aqui: https://github.com/freyavs/DEV21-VanSpeybroeck/wiki/Los-menus
