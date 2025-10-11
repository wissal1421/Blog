# Algoritmo BSA de cobertura

Este proyecto implementa un **robot autónomo tipo aspiradora** capaz de recorrer un entorno desconocido, **planificar su trayectoria, detectar zonas críticas, y volver a celdas vecinas que no estuvieran visitadas** usando un sistema de navegación **BFS (Breadth-First Search)** y **controladores PID** para movimiento preciso.

---

## Objetivo del proyecto

El objetivo principal es crear un sistema que:
1. **Divida el mapa en celdas navegables.**
2. **Genere un plan de exploración completo**, marcando qué zonas ya fueron visitadas.
3. **Detecte zonas críticas o sin salida**.
4. **Use BFS para volver a puntos pendientes (return cells)** y seguir limpiando.
5. **Controle el movimiento del robot en el mundo real** mediante PIDs lineal y angular, para alcanzar cada celda sin desviaciones.

---

## Estructura general del código

El flujo del programa se divide en **cuatro estados principales**:

| Estado | Descripción |
|--------|--------------|
| `PLAN` | El robot planifica nuevas celdas a visitar y las marca. |
| `BFS_PLAN` | Si el robot llega a un callejón sin salida, usa BFS para volver a una celda de retorno pendiente. |
| `MOVE` | Ejecuta el plan ya definido, moviéndose de celda en celda. |
| `FINISHED` | Finaliza la limpieza cuando no quedan celdas libres ni caminos válidos. |

Estos estados se manejan dentro de un bucle `while True`, garantizando un comportamiento cíclico y continuo.

---

## Procesamiento del mapa

El mapa se obtiene mediante la interfaz de **WebGUI** y se convierte en una **imagen en escala de grises**:

```python
MAP_IMG = WebGUI.getMap('/resources/exercises/vacuum_cleaner_loc/images/mapgrannyannie.png')
gray = cv2.cvtColor(MAP_IMG, cv2.COLOR_BGR2GRAY)
```
Una vez se ha pasado a escala de grises se erosionan los obstáculos para que el robot no choque contra ellos:

```python
# --- Erosionar obstáculos para dejar más espacio de seguridad ---
# kernel define el "tamaño" de la erosión. Cuanto más grande, más se expanden los obstáculos.
kernel = np.ones((20, 20), np.uint8)
gray = cv2.erode(gray, kernel, iterations=1)
```

---

## Transformación de Coordenadas

Para convertir entre el mundo real y la imagen del mapa, se utiliza una transformación afín calculada mediante mínimos cuadrados:

```python
A = np.vstack([X, Y, np.ones(len(X))]).T
a_params, _, _, _ = np.linalg.lstsq(A, U, rcond=None)
b_params, _, _, _ = np.linalg.lstsq(A, V, rcond=None)
T = np.vstack([a_params, b_params])
```

`world_to_pixel(x, y)` convierte coordenadas del mundo real a píxeles:
```python
def world_to_pixel(x, y):
    vec = np.array([x, y, 1])
    return T.dot(vec)
```

`pixel_to_world(u, v)` busca las coordenadas del mundo teniendo los del pixel:
```python
def pixel_to_world(u, v):
    # Convertir T en 3x3
    T_ext = np.vstack([T, [0, 0, 1]])
    # Invertir
    T_inv = np.linalg.inv(T_ext)
    vec = np.array([u, v, 1])
    res = T_inv.dot(vec)
    return res[0], res[1]
```

---

## Estado PLAN
El estado PLAN es el encargado de construir el recorrido completo del robot por el entorno.
Su objetivo es explorar todas las celdas libres mientras va marcando su progreso, detectando posibles caminos de retorno y gestionando los puntos críticos.

La lógica general se puede dividir en tres grandes bloques:

### 1. Marcado de celdas visitadas

Cada vez que el robot entra en una nueva celda, la añade al plan y la marca como visitada:
```python
if cell not in plan:
    plan.append(cell)
    visited.add(cell)
    paint_cell(cell[0], cell[1], FREE)
```
De esta forma, el robot lleva un registro de por dónde ha pasado y evita volver a las mismas celdas salvo que sea necesario para retornar.

### 2. Expansión de vecinos

A partir de la celda actual, se obtienen los vecinos libres (arriba, abajo, izquierda y derecha) que todavía no se han visitado:
```python
neighbors = [
    (cell[0] + 1, cell[1]),
    (cell[0], cell[1] + 1),
    (cell[0] - 1, cell[1]),
    (cell[0], cell[1] - 1)
]

free_neighbors = [
    n for n in neighbors
    if 0 <= n[0] < rows and 0 <= n[1] < cols
    and not is_obstacle(n)
    and n not in visited
]
```

Si encuentra vecinos disponibles, avanza hacia el primero y guarda los demás como posibles celdas de retorno, marcándolas en azul:
```python
next_cell = free_neighbors[0]
cell = next_cell
plan.append(cell)
visited.add(cell)
paint_cell(cell[0], cell[1], FREE)

for n in free_neighbors[1:]:
    if n not in return_cells and n not in visited:
        return_cells.append(n)
        paint_cell(n[0], n[1], RETURN_CELL)

```

### 3. Manejo de retornos y puntos críticos
Cuando una celda no tiene vecinos libres, se considera crítica, y el sistema busca una celda de retorno desde el stack return_cells.
En ese caso, pasa al estado BFS_PLAN para planificar un camino hasta ella:
```python
paint_cell(cell[0], cell[1], CRITICAL_CELL)
critical_cells.add(cell)

if return_cells:
    ret = return_cells.pop()
    if ret not in visited:
        origin_cell = cell
        ret_target = ret
        q = deque([cell])
        parent = {cell: None}
        seen = {cell}
        state = BFS_PLAN
```
Si no quedan celdas de retorno, significa que el entorno ya se ha explorado completamente, y el plan pasa al estado MOVE para ejecutar el recorrido final.

---

## Estado BFS_PLAN
Cuando el robot se queda sin vecinos libres durante la fase de planificación (PLAN), necesita encontrar una ruta hacia alguna celda de retorno pendiente.
Ahí entra en juego BFS_PLAN, que usa una búsqueda en anchura (BFS) para construir el camino más corto posible hasta esa celda.

Buscamos el camino más corto desde la celda actual hasta ret_target, lo reconstruimos y lo insertamos directamente en el plan existente, de manera que el recorrido siga siendo coherente y sin saltos, el pseudocódigo quedaría así:

    1. Marcamos la celda objetivo (RETURN_CELL) en el mapa para visualizar hacia dónde queremos volver.
    2. Sacamos una celda de la cola (q.popleft()) y comprobamos si es el destino (ret).
    3. Si lo es, reconstruimos el camino usando el diccionario parent, lo invertimos y lo insertamos en el plan principal justo después de la celda desde la que veníamos (origin_cell).
    Finalmente, actualizamos cell = ret_target y volvemos al estado PLAN para seguir explorando.

---




