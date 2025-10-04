# Aspiradora de gama baja con Autómata de Estados Finito reactivo
Esta primera práctica implementa un autómata de estados finitos (FSM) que controla el comportamiento de una aspiradora de gama baja que usa sus sensores para evitar obstaculos.

El objetivo de esta practica es que la aspiradora aspire el mayor área de la casa posible, lo recorre en espiral, retrocede cuando ve obstaculos y avanza a una nueva zona en la que no hay obstaculos cerca

La demo de la práctica la podeis encontrar en el siguiente enlace(el video esta en x3.5 ya que su duracion era muy larga): [Ver video de la demo](https://drive.google.com/file/d/1RL-5eyy3iSCq4FYbWVSv6drl4xO3okxa/view?usp=sharing)

## Estructura general del sistema

El programa sigue una máquina de estados finitos (FSM) con los siguientes estados:
  - TURNING: Estado inicial del robot y en el cual realiza las espirales
  - BACKWARD: Estado al que pasa tras encontrar un objeto con el laser, consiste en poner una velocidad negativa(para retroceder) hasta que el obstaculo este lo suficientemente lejos
  - AVOIDING_OBSTACLE: Una vez a retrocedido lo suficiente, gira hacia el lado donde la media de obstaculos esta mas lejos, es decir, girara a la derecha si los obstaculos estan mas lejos a ese lado y viceversa
  - FORWARD: Este es el ultimo estado al que pasa y lo hace cuando ha girado lo suficiente, consiste en ir hacia delante 5 segundos, durante este tiempo no debe encontrar nngun obstaculo y si lo hace volvera al estado BACKWARD, de no haber objeto, comenzara a hacer espirales nuevamente

## Lógica del autómata










