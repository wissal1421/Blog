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















