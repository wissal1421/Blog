
# Robot que se autolocaliza con balizas visuales

Este blog contiene la implementación de un sistema de localización visual basada en marcadores (AprilTags) y una lógica de navegación reactiva para un robot móvil. El sistema combina la estimación de pose por visión (PnP) con la integración de incrementos de odometría para mitigar el error acumulado (drift).

## Arquitectura del Sistema de Localización

La localización se basa en el cálculo de la transformación entre el sistema de coordenadas del robot y un marco de referencia global definido por la posición conocida de las etiquetas AprilTag.

### Estimación de Pose (Perspective-n-Point)

Utilizamos el algoritmo SolvePnP con el flag `SOLVEPNP_IPPE_SQUARE` (específico para marcadores planos) para obtener los vectores de rotación (rvec) y traslación (tvec).

```python
# Extracción de la matriz de transformación Cámara -> Marcador
c2m = np.eye(4)
c2m[:3, :3] = cv2.Rodrigues(rvec)[0]
c2m[:3, 3] = tvec.ravel()
```

### Transformación de Coordenadas y Ajuste de Ejes

Para obtener la pose global, se invierte la relación cámara-marcador y se multiplica por la posición conocida del tag en el mundo (w2m​). Se aplica una matriz de ajuste (ROT_ADJUST) para alinear el sistema de coordenadas de la cámara (Eje Z hacia adelante) con el sistema del mundo (Eje Z hacia arriba).

```python
# Inversión y ajuste de ejes para obtener Marcador -> Cámara
m2c = np.linalg.inv(c2m)
m2c_adj = np.dot(ROT_ADJUST, m2c)

# Transformación final a coordenadas globales
w2c = np.dot(w2m, m2c_adj)
yaw_est = float(np.arctan2(w2c[1, 0], w2c[0, 0]))
```

## Fusión de Datos

Debido a que la odometría absoluta de `HAL.getOdom()` presenta un ruido gaussiano acumulativo, se implementó una lógica de actualización por incrementos.

-  Corrección: Cuando hay una detección válida, la pose estimada se sobrescribe con el cálculo visual.

-  Predicción: En ausencia de etiquetas, se calcula el diferencial (Δx,Δy,Δθ) entre el estado de odometría actual y el anterior, sumándolo a la última pose estimada válida.

```python
# Cálculo de incrementos (Deltas)
dx = odom.x - x_l
dy = odom.y - y_l
dyaw = normalize_angle(odom.yaw - yaw_l)

# Integración sobre la última estimación conocida
x_e += dx
y_e += dy
yaw_e = normalize_angle(yaw_e + dyaw)
```

## Control de Navegación Reactiva

El robot opera bajo una Máquina de Estados Finitos (FSM) con dos estados principales:

-  FORWARD: Control de velocidad mediante un PID lineal que toma como error la distancia medida por el sensor LIDAR frente al `THRESHOLD_WALL`.

-  TURN: Activado por proximidad a obstáculos o por un temporizador (random.uniform). La dirección de giro se determina de forma aleatoria para maximizar la exploración del área.

```python
# Control de velocidad lineal mediante PID
dist_error = front_dist - THRESHOLD_WALL
lin_speed = pid_lineal(dist_error)
HAL.setV(lin_speed)
```

## Resultados y Demostración

El sistema demuestra robustez frente a la pérdida temporal de contacto visual con las balizas, manteniendo una estimación de pose coherente mediante la integración de deltas de odometría. La precisión aumenta significativamente al re-entrar en el campo de visión de un AprilTag, donde el error de deriva se resetea a cero.

A continuación, se adjunta una demostración en vídeo del sistema en ejecución, donde se observa la convergencia de la pose estimada (marcador rojo) con la posición real del robot en el entorno de simulación:

[Vídeo](https://drive.google.com/file/d/1S63AcW_s-0kkW183hWTw397k0h4xhMRA/view?usp=sharing)
