# Navegación local esquivando obstáculos con algoritmo VFF

El objetivo de esta práctica fue programar un coche de Fórmula 1 autónomo, equipado con un sensor láser y un sensor GPS, para que pudiera recorrer un circuito de carreras delimitado por vallas y con otros coches como obstáculos. Para lograrlo, implementé un algoritmo de navegación local basado en el método VFF (Vector Field Force), que utiliza fuerzas virtuales para ajustar la velocidad lineal (V) y la velocidad angular (W) del coche en cada iteración.

El propósito principal era que el coche completara el circuito en el menor tiempo posible sin colisionar con ningún obstáculo. Además, el sistema de navegación global planificaba una serie de subobjetivos o puntos intermedios que, al alcanzarse uno tras otro, garantizaban que el coche diera la vuelta completa al circuito.

La aplicación se desarrolló como un bucle continuo que, en cada iteración, procesaba los datos del sensor láser, calculaba las fuerzas repulsivas y atractivas, combinaba dichas fuerzas y las traducía en decisiones de movimiento para los actuadores del vehículo.
