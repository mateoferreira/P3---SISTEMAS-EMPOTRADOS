# P3---SISTEMAS-EMPOTRADOS
Para esta práctica, he empleado como control de mi sistema una máquina de estados. 
Tengo una amplia colección de threads, y los estados básicamente van activando y desactivando esos threads.
Así mi loop principal queda muy reducido, aunque el programa como tal, no es muy fácil de depurar ya que un thread mal activado o desactivado puede romper totalmente la lógica.

Esto ha sido la mayor dificultad de la práctica, el ir solucionando estos errores de desactivaciones de threads o variables globales, y puedo decir que ha sido mi mayor problema porque son muy dificiles de detectar.

Para afrontarla, he ido primero desarrollando los estados incrementalmente, primero he empezado con el primer estado, el estado de cargando. Cuando este termina modifica una variable global, pasamos al siguiente estado, lo desarrollo y lo pruebo, y así sucesivamente. Esto me ha ayudado mucho a simplificar y entender mi código, pero se ha ido complicando cuando he tenido que usar los mismos Threads o estados para distintas funciones (como mostrar el menú para preparar un café y mostrar el menú para modificar el precio de un café).

He elegido el uso de Threads para casi todo: leer sensores, desplazar el texto del LCD...
Para la lectura de sensores he optado por los threads y no por las interrupciones software por el hecho de que me parecía más cómodo, más o menos dominaba su uso y evitaba complicarme con el uso de Timers en los pines, ya que podría tener comportamientos indeseados y me parecía innecesario.

No he usado estas interrupciones software, pero sí las hardware para la lectura de botones. En el caso del botón, la interrupción es necesaria, y va a estar activa desde que se enciende la placa. Esta va a saltar cuando haya un cambio de estado, y mediante una variable global compruebo el tiempo entre el LOW y el HIGH, y va a ser en dos threads donde decido como tratarla. El thread de restart (vuelve al estado esperando cliente si el boton se pulsa entre 2-3 segundos) va a estar disponible desde que detecta a un cliente hasta que empieza a preparar el café; y el thread de admin (pasa al estado admin si el boton se pulsa durante >= 5 segundos, o sale del estado admin si el boton se pulsa durante ese tiempo), va a estar disponible desde que la placa se inicia, ya que podemos entrar en cualquier momento.

El botón no tiene tiempo de rebote, ya que el ISR lo manejo de una forma que siempre se resetea correctamente cuando dejamos de pulsar. El único problema es que a veces no se pulsa correctamente el botón, y aunque calcules bien el tiempo de pulsado, puede medirse mal ya que detecta el inicio más tarde y tienes que volver a repetir la pulsación. Esto no me ha pasado en muchas ocasiones, pero por ejemplo, si me ha pasado en el video de muestra. Podría haberlo repetido, pero he decidido dejarlo así para mostrar que puede haber pequeños errores.

Por otro lado, he usado el otro pin de interrupción hardware para el joystick. A diferencia del botón, esta interrupción va a estar disponible solo en determinadas ocasiones, es decir, solo cuando tengamos que seleccionar algo en el menú de productos o en el de admin. Podría haber tratado el botón como los ejes x e y del joystick, es decir, como un thread que mire el estado del botón, pero me ha resultado más visual tratarlo como interrupción que modifica una variable global. Como inconveniente, el joystick si que tiene rebote, y he tenido que tratarlo ya que la máquina me funcionaba incorrectamente en la selección de productos y no identificaba el por qué.

El resto del código ha sido seguir las indicaciones del enunciado, en todo momento menos en una parte. Por una equivocación al leer el enunciado, cuando acaba la preparación del café, se queda esperando a que el cliente retire su bebida hasta que el ultrasonidos detecta que hay cliente. En el enunciado pone que hay que esperar 3 segundos, pero como ya he demostrado que sé calcular el tiempo entre el inicio y el fin de un estado (por ejemplo, en la muestra de temperatura y humedad antes de mostrar el menú), he decidido mantener mi minúscula modificación, ya que, en mi opinion personal, quedaba bien.

Para evitar posibles bloqueos uso un watchdog de 1 segundo que reseteo en cada iteracion del loop principal. Este está solo como último recurso, ya que en mi código a primera vista no pueden ocurrir bloqueos (no hay delays, ni dentro ni fuera de los Threads)

Por último, voy a hablar de los intervalos de activación de los Threads. Estos los he escogido bastante a ojo. Excepto el contador, que por cierto, es un thread que se ejecuta cada 1 segundo y solo se muestra por pantalla cuando una variable global es activada por la pulsaciñon del botón del joystick en el momento oportuno; y el Thread que lleva el parpadeo del led inicial, los Threads tienen unos intervalos de activación menores a 1 segundo. La mayoría estan entorno al medio segundo, en especial, menciono al Thread de preparación que va aumentando la intensidad del led2 a medida que el café se prepara. Defino este intervalo como variable global, ya que es más fácil cambiarlo dependiendo de si quieres ver el progreso de la intensidad más o menos claro.
Los Threads del joystick son los que tienen un intervalo menor, ya que hay que intentar pillar el movimiento cuando el usuario mueve el joystick, pero tampoco puede ser muy pequeño, ya que seguramente en vez de moverse de 1 en 1 posición, podría moverse de 2 en 2 o incluso más 

## MONTAJE DEL CIRCUITO
Los componentes los he ido probando de uno en uno para ver sus respectivos funcionamientos. El manejo del sensor de Temperatura, junto con el del joystick han sido los más complicados.
El de temperatura ya que en las trasparencias tenía los pines cambiados y me costó mucho encontrar el fallo de las lecturas, y el joystick ya que la orientación es importante. Imprimiendo los valores por el puerto serie, descubrí que los movimientos arriba y abajo se detectaban por el eje X, y derecha izquierda por el del Y, lo que resulta poco intuitivo.

Por lo demás ha sido sencillo el montaje, con ayuda de ChatGPT y de las trasparencias. Un componente que ha sido fácil de montar es el LCD. Sé que algunos compañeros lo montaron con potenciómetro para modular el contraste de la pantalla, pero yo lo monté tal cual las traspas, cambiando únicamente la resistencia hasta que se viesen las letras como yo quería, y el resultado ha sido impecable.

Como comentarios, a falta de pines digitales tuve que conectar los ejes x e y del joystick a pines analógicos, y el ultrasonidos lo he puesto mirando opuestamente al usuario para que pueda probarlo mejor, ya que sino, siempre iba a dectarme a mí.
Otra cosa, es que el esquema de circuitos lo hice inicialmente con Tinkercad. El esquema se me borró de un día para otro aunque estuviese guardado y menos mal que hice una captura antes de perderlo. También en esta aplicación no había el sensor de temperatura ni el joystick. Para simular el de temperatura he puesto un sensor con 3 pines, donde están en la misma posición que en el circuito real, y para el joystick, ya que perdí mi esquema, lo he representado aparte con la app Fritzing, con el único inconveniente de que los pines del joystick que tenemos en esa app están en distinto orden que los reales, pero los he conectado como si estuviesen en el mismo orden.
Por último, en mi esquema he puesto el ultrasonidos invertido respecto al real para una mayor claridad.

Los esquemas de mi circuito son, el general:
<img width="722" height="531" alt="circuito" src="https://github.com/user-attachments/assets/cc4a47e5-03c3-4c86-98e6-7b7fe3ddf81d" />

Y el del joystick:

<img width="821" height="574" alt="Captura desde 2025-11-22 20-01-29" src="https://github.com/user-attachments/assets/c10a688d-dff5-40a0-a9a0-9b9c0d05a633" />

Y mi circuito real: 

![image](https://github.com/user-attachments/assets/b2ca0f17-0fc5-4059-a60d-fc5faece1190)


## VIDEO DEMOSTRATIVO
Por último, adjunto el enlace de mi video. He probado el circuito en numerosas ocasiones, provocando comportamientos indeseados intentando buscar errores y los he minimizado. No quiero decir que haya acabado con todos pero si puedo asegurar que mi programa cumple a la perfección las especificaciones y a prueba de errores.

https://youtu.be/eBv9IiNePuA?si=lyyHkiabI4-EKQQB
