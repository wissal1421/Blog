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

