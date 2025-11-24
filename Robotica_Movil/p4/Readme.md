# Navegación global con algoritmo GPP, plan como recurso

## Navegación reactiva + planificación como recurso 

En este proyecto programé un coche autónomo, equipado con un sensor GPS y un mapa de la ciudad, para que ejerza de taxi autónomo siendo capaz de planificar su navegación desde el lugar en el que se encuentra hasta otro punto de la ciudad como destino utilizando el algoritmo de Gradient Path Planning, combinando:
 
-  Un campo de propagación hacia destino.

-  Un subcampo de penalización por obstáculos.

-  Un piloto reactivo que gobierna las velocidades lineal (V) y angular (W).

Grabé dos vídeos mostrando su ejecución real:

1)  Navegación con plan como recurso (utilizando el gradiente completo).

2)  Navegación puramente reactiva basada en costes locales sin plan global.


## 1. Preparación del mapa y ampliación de obstáculos

El robot recibe un mapa de la ciudad y lo binariza en obstáculos y espacios libres. Para evitar colisiones, genero una versión expandida del mapa donde cada obstáculo se agranda unos píxeles para representar el volumen del coche.

```python

city_map = np.where(city_map > 100, 255, 0)     # 255 = libre, 0 = obstáculo
OBSTACLE_EXPANSION_RADIUS = 4

# Ejemplo de expansión
for dr in range(-OBSTACLE_EXPANSION_RADIUS, OBSTACLE_EXPANSION_RADIUS + 1):
    for dc in range(...):
        expanded_cost_map[row+dr, col+dc] += CAR_MARGIN

```
Esto crea un “colchón” de seguridad para que el taxi no roce esquinas ni callejones estrechos.


## 2. Propagación de costes desde el destino

Lo mas importante del algoritmo es la creación de un campo escalar, cada celda del mapa almacena lo “costoso” que es llegar al destino desde ahí.

Se calcula desde la meta hacia afuera mediante una propagación tipo wavefront:

```python
cost_map[target_y, target_x] = 0
exploration_queue = deque([target_coords])

for neighbor in neighbors:
    cost_map[n_y, n_x] = current_cost + 1
```

El resultado es un mapa donde el robot solo debe seguir el descenso del gradiente para acercarse a su destino.


## 3. Reconstrucción del camino (plan como recurso)

Con el campo de costes listo, reconstruyo un camino aproximado tomando siempre el vecino con menor coste:

```python

while cost_map[cur_y, cur_x] > 0:
    best = get_best_neighbor(cur)
    path.append(best)
    cur = best

```

Este plan no es obligatorio (el robot puede navegar sin él), pero acelera la convergencia, reduce zigzags y evita zonas con penalización por obstáculos.

Mostré este plan visualmente sobre el mapa para depuración, marcando el camino con valores intermedios.


## 4. Pilotaje reactivo: convertir el gradiente en movimiento

Tanto si hay plan como si no, el robot finalmente avanza de forma reactiva:

1) Elige el siguiente punto a seguir (del plan o del gradiente local).

2) Convierte ese punto a coordenadas del mundo.

3) Calcula el ángulo deseado y ajusta velocidades lineales y angulares.

```python

desired_angle = atan2(goal_y - robot_y, goal_x - robot_x)
angle_diff = normalize(desired_angle - robot_yaw)

linear_speed = MAX_LINEAR_SPEED - abs(angle_diff)
angular_speed = 3.0 * angle_diff

HAL.setV(linear_speed)
HAL.setW(angular_speed)

```

## 5. Esperar al siguiente detino una vez alcanzado el primero

Una vez el coste es muy bajo (≈2), se considera alcanzado el objetivo. Entonces el robot se detiene y espera a que el usuario seleccione un nuevo punto del mapa.

```python

if cost_map[next_y, next_x] <= 2:
    HAL.setV(0); HAL.setW(0)
    esperar_nuevo_objetivo()

```

## 6. Vídeos demostrativos

Para validar todo el sistema grabé dos ejecuciones:

### 1. Navegación con planificación global

El coche utiliza todo el campo escalar y sigue el camino reconstruido. [VIDEO](https://drive.google.com/file/d/1fhoZH3SQvsm44yd5XmOb3KHp8C82zFnJ/view?usp=sharing)

### 2. Navegación reactiva sin plan

Solo observa costes locales y sigue gradiente inmediato; más sencilla pero menos óptima. [VIDEO](https://drive.google.com/file/d/1bRRTOWKjDn-67kcSKT89Lckop98U-QeE/view?usp=sharing)

Ambas muestran cómo el vehículo sortea obstáculos y toma calles hasta alcanzar el destino marcado.









