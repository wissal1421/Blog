
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








https://drive.google.com/file/d/1S63AcW_s-0kkW183hWTw397k0h4xhMRA/view?usp=sharing
