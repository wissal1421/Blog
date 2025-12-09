
# GridMap Probabilístico en un Almacén

Este ejercicio consiste en construir el Mapa de Ocupación Probabilístico (GridMap) de una nave industrial simulada. Este proyecto combina la navegación autónoma con la fusión sensorial avanzada, utilizando el sensor lidar del robot y su propia localización.

Los objetivos son:

1)  Exploración: Programar un algoritmo para que el robot se mueva deambulando por el entorno hasta cubrir su totalidad.

2)  Mapeo Probabilístico: Implementar el algoritmo de construcción del GridMap utilizando la Regla de Bayes y mecanismos de saturación para manejar la incertidumbre de los sensores.

## Estrategia de Exploración: Navegación Serpenteante Inteligente

Para garantizar que el robot cubriera toda la nave, descarté el movimiento aleatorio y opté por una estrategia serpenteante (Serpentine Exploration).

Mi robot opera mediante una Máquina de Estados simple que le permite avanzar, girar en las esquinas y moverse lateralmente:

### Estados (STATE_FORWARD, STATE_SIDE_MOVE, STATE_TURNING_180)

El robot sigue la pared hasta que choca con un obstáculo (FRONT_THRESHOLD). Luego, gira 90∘ y avanza lateralmente (SIDE_DISTANCE) para simular el barrido del área. Finalmente, gira 180∘ para volver en sentido contrario.

Snippets Clave:

-  Avance: El estado STATE_FORWARD simplemente establece la velocidad lineal.
```python
if current_state == STATE_FORWARD:
    # ...
    HAL.setV(FORWARD_SPEED)
    HAL.setW(0)
```

-  Movimiento Lateral: La distancia lateral es crucial. Tras muchas pruebas, descubrí que usar un valor SIDE_DISTANCE grande como 5.15 metros funcionaba mejor que valores pequeños (2 o 3 metros) para mi entorno, ya que permitía cubrir grandes tramos antes de dar la vuelta. Una cosa que seria interesante es colocar el valor de SIDE_DISTANCE igual a laser.maxRange ya que haria el codigo mucho mas eficiente al no pasar 2 veces por un sitio en el que ha dado toda probabilidad a las celdas.

### Refinamiento en Giros y Control

Para evitar que el robot se desviara en diagonales y acumulara error de rumbo, implementé un Control Proporcional (P-Control) en la velocidad angular (HAL.setW) en todos los estados de giro (STATE_TURNING_90, STATE_TURNING_180).

-  Utilizo el Error Angular (angle_error) para calcular el comando de velocidad.
```python

# Snippet de STATE_TURNING_90
if abs(angle_error) > ANGLE_TOLERANCE:
    w_command = KP_TURN * angle_error # Control Proporcional
    # ... limitar w_command ...
    HAL.setW(w_command)
# ...
```
### Detección Inteligente de Esquinas

En lugar de girar 180∘ al encontrar una esquina, la lógica identifica el camino abierto:

-  Si detecto obstáculo frontal y pared a la derecha (is_right_corner), fuerzo el giro 90∘ hacia la izquierda (turn_direction = 1).

-  Esto se gestiona en la prioridad más alta de la Máquina de Estados, para evitar que la lógica de serpenteo normal tome el control.

## Construcción del GridMap Probabilístico

Esta fue la parte más desafiante. En lugar de un mapa simple (0=Libre, 1=Ocupado), necesitábamos un mapa que almacenara la probabilidad de ocupación.

### La Regla de Bayes y Log-Odds

Para combinar la evidencia de cada rayo láser con la creencia previa del mapa, utilicé la Regla de Bayes en el espacio Log-Odds.

-  El mapa (MAP_LOG_ODDS) no almacena la probabilidad P∈[0,1], sino el Log-Odds L∈[−∞,∞]. Esto simplifica la actualización a una simple suma, haciéndola eficiente:
```
    L_new​=L_old​+L_sensor​
```

-  Definimos las ganancias sensoriales que representan la evidencia:
```python
L_OCC = 0.5   # Evidencia de Obstáculo (hace L más positivo)
L_FREE = -0.1 # Evidencia de Espacio Libre (hace L más negativo)
```

### Trazado de Rayos y Saturación

La función mapping() itera sobre cada rayo lidar:

1)  Trazado de Rayos: Usando las coordenadas del robot y del obstáculo (convertidas a píxeles usando WebGUI.poseToMap), trazamos la línea entre ambos puntos. Todos los píxeles intermedios se actualizan con LFREE​.

2)  Actualización del Obstáculo: El píxel final (donde choca el rayo) se actualiza con LOCC​.

3)  Saturación: Para evitar que unas pocas lecturas incorrectas dominen el mapa (el concepto de inercia excesiva), aplicamos saturación después de cada actualización. Esto limita los valores de Log-Odds a un rango seguro:
```python
# Aplicación de Saturación
MAP_LOG_ODDS[i, j] = np.clip(MAP_LOG_ODDS[i, j] + L_FREE, L_MIN, L_MAX)
```

### Visualización

Mi función update_probabilistic_gui() convierte los valores de Log-Odds a probabilidad, y luego a la escala de grises para enviarla a la interfaz de la WebGUI:

-  P = 0.5 (Desconocido) --> Gris (128)

-  P = 1.0 (Ocupado) --> Negro (0)

-  P = 0.0 (Libre) --> Blanco (255)

## Impacto de las distintas Odometrías

La prueba final del ejercicio era crucial: ver cómo la calidad del mapa se deteriora al usar fuentes de localización con mayor ruido.

-    Localización (HAL.getOdom()): Proporciona la mejor estimación de la pose del robot (posición x,y y orientación yaw), asumiendo un ruido mínimo.
      -    Resultado (Video 1 / Imagen 1): El mapa generado es muy preciso. Las paredes son líneas finas y paralelas, y el mapa se cierra correctamente.

-    Odometría con Ruido 1 (HAL.getOdom2()): Simula un sensor de odometría con una cantidad moderada de ruido.

      -    Resultado (Video 2 / Imagen 2): Se nota una deriva (drift) significativa. Las paredes paralelas se mapean como líneas dobles o "engordadas" porque el robot cree que está en un lugar cuando realmente está ligeramente en otro. El mapa se ve distorsionado y no se cierra correctamente en el origen.

-    Odometría con Ruido 2 (HAL.getOdom3()): Simula un alto nivel de ruido en la odometría.

      -    Resultado (Video 3 / Imagen 3): El mapa se deteriora rápidamente. Las líneas de las paredes se vuelven borrosas o se duplican drásticamente, haciendo que el mapa sea casi inútil para la navegación. Esto demuestra la extrema sensibilidad del mapeo de ocupación a la precisión de la localización. Si la pose es incorrecta, la evidencia del sensor se registra en el lugar equivocado, destruyendo la consistencia del mapa.

Este ejercicio ilustró perfectamente que "un buen mapa requiere una buena localización." Si la fuente de localización (en este caso, la odometría) es ruidosa, ningún algoritmo de mapeo, por sofisticado que sea (como el bayesiano que usamos), puede generar un mapa preciso.























