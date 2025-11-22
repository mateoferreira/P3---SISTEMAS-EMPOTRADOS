# P3---SISTEMAS-EMPOTRADOS
Para esta práctica, he empleado como control de mi sistema una máquina de estados. 
Tengo una amplia colección de threads, y los estados básicamente van activando y desactivando esos threads.
Así mi loop principal queda muy reducido, aunque el programa como tal, no es muy fácil de depurar ya que un thread mal activado o desactivado puede romper totalmente la lógica.

Esto ha sido la mayor dificultad de la práctica, el ir solucionando estos errores de desactivaciones de threads o variables globales, y puedo decir que ha sido mi mayor problema porque son muy dificiles de detectar.

Para afrontarla, he ido primero desarrollando los estados incrementalmente, primero he empezado con el primer estado, el estado de cargando. Cuando este termina modifica una variable global pasamos al siguiente estado, lo desarrollo y lo pruebo, y así sucesivamente. Esto me ha ayudado mucho a simplificar y entender mi código, pero se ha ido complicando cuando he tenido que usar los mismos Threads o estados para distintas funciones (como mostrar el menú para preparar un café y mostrar el menú para modificar el precio de un café).

He elegido el uso de Threads para casi todo: leer sensores, desplazar el texto del LCD...
Para la lectura de sensores he optado por los threads y no por las interrupciones software por el hecho de que me parecía más cómodo, más o menos dominaba su uso y evitaba complicarme con el uso de Timers en los pines, ya que podría tener comportamientos indeseados y me parecía innecesario.

No he usado estas interrupciones software, pero sí las hardware para la lectura de botones. En el caso del botón, la interrupción es necesaria, y va a estar activa desde que se enciende la placa. Esta va a saltar cuando haya un cambio de estado, y mediante una variable global compruebo el tiempo entre el LOW y el HIGH, y va a ser en dos threads donde decido como tratarla. El thread de restart (vuelve al estado esperando cliente si el boton se pulsa entre 2-3 segundos) va a estar disponible desde que detecta a un cliente hasta que empieza a preparar el café; y el thread de admin (pasa al estado admin si el boton se pulsa durante += 5 segundos, o sale del estado admin si el boton se pulsa durante ese tiempo), va a estar disponible desde que la placa se inicia, ya que podemos entrar en cualquier momento.


## VIDEO DEMOSTRATIVO
https://youtu.be/eBv9IiNePuA
