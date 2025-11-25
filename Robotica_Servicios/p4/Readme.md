
# Robot que mueve estanterías en un almacén (logística)

Durante esta práctica trabajé con dos robots muy diferentes: un holonómico y un Ackermann. Mi objetivo era que cada robot recogiera estanterias dentro del almacén y los llevara a unos puntos que defini como DROP OFFS, planificando caminos con OMPL y siguiendo las trayectorias con PID.

## Preparacón del mapa

El mapa está en píxeles, mientras que el robot trabaja en metros, así que primero tuve que relacionar ambos sistemas con una transformación lineal ajustada por mínimos cuadrados.

```python
# puntos en pixeles y metros (ejemplo ilustrativo)
pix = np.array([[120, 350], [820, 360], [130, 920]])
mt  = np.array([[0.5, 0.2], [2.4, 0.22], [0.55, 1.8]])

# estimar transformación afín
A, _, _, _ = np.linalg.lstsq(
    np.hstack([pix, np.ones((pix.shape[0],1))]),
    mt,
    rcond=None
)

def pix_to_m(p):
    p = np.array([p[0], p[1], 1.0])
    return A.T @ p
```

Esto me permitió pasar cualquier punto del mapa a coordenadas reales antes de planificar.

## El robot holonómico

Para el holonómico usé un espacio de estados en SE(2) normal, ya que no tiene restricciones de movimiento lateral.

### Planificador RRT* con OMPL

Ejemplo ilustrativo del setup:

```python

space = ob.SE2StateSpace()
bounds = ob.RealVectorBounds(2)
bounds.setLow(0)
bounds.setHigh(warehouse_size)
space.setBounds(bounds)

si = ob.SpaceInformation(space)
si.setStateValidityChecker(is_valid_state)

problem = ob.ProblemDefinition(si)
problem.setStartAndGoalStates(start, goal)

planner = og.RRTstar(si)
planner.setRange(0.4)
planner.setGoalBias(0.05)
planner.setProblemDefinition(problem)
planner.setup()

```

### Control del movimiento con PID

El holonómico requiere control independiente de orientación y posición.

Un ejemplo muy simplificado de mi ciclo de control:

```python
err_pos  = np.hypot(goal_x - x, goal_y - y)
err_yaw  = normalize_angle(goal_yaw - yaw)

v  = kp_v  * err_pos
w  = kp_w  * err_yaw

send_velocity(vx=v, vy=0, w=w)

```














