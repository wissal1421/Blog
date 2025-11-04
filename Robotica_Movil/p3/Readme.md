# Navegación local esquivando obstáculos con algoritmo VFF

El objetivo de esta práctica fue programar un coche de Fórmula 1 autónomo, equipado con un sensor láser y un sensor GPS, para que pudiera recorrer un circuito de carreras delimitado por vallas y con otros coches como obstáculos. Para lograrlo, implementé un algoritmo de navegación local basado en el método VFF (Vector Field Force), que utiliza fuerzas virtuales para ajustar la velocidad lineal (V) y la velocidad angular (W) del coche en cada iteración.

El propósito principal era que el coche completara el circuito en el menor tiempo posible sin colisionar con ningún obstáculo. Además, el sistema de navegación global planificaba una serie de subobjetivos o puntos intermedios que, al alcanzarse uno tras otro, garantizaban que el coche diera la vuelta completa al circuito.

La aplicación se desarrolló como un bucle continuo que, en cada iteración, procesaba los datos del sensor láser, calculaba las fuerzas repulsivas y atractivas, combinaba dichas fuerzas y las traducía en decisiones de movimiento para los actuadores del vehículo.

## Diseño del algoritmo

Empecé definiendo unas ganancias para ajustar la intensidad de las fuerzas atractiva y repulsiva:

```python
AIN_ATTRACTIVE = 4.00
GAIN_REPULSIVE = 5.00
THERESHOLD_DIST = 1.25  # Distancia mínima a partir de la cual un obstáculo comienza a repeler al coche
```

Luego, escribí una función para convertir coordenadas absolutas a relativas, porque el coche necesita saber dónde están los objetos con respecto a su propia posición:

```python
def absolute2relative(x_abs, y_abs, robot_x, robot_y, robot_yaw):
    dx = x_abs - robot_x
    dy = y_abs - robot_y

    x_rel = dx * math.cos(robot_yaw) + dy * math.sin(robot_yaw)
    y_rel = -dx * math.sin(robot_yaw) + dy * math.cos(robot_yaw)

    return x_rel, y_rel
```

También procesé los datos del sensor láser para conocer la posición de los obstáculos cercanos. Cada lectura del láser me da una distancia a un obstáculo en cierto ángulo, así que convertí esas lecturas en coordenadas cartesianas:

```python
for i in range(180):
    dist = laser_data.values[i]

    angle = math.radians(i - 90)

    x = dist * math.cos(angle)
    y = dist * math.sin(angle)
```
