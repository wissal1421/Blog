# Drone en misión de búsqueda y rescate

Este proyecto implementa un sistema autónomo de búsqueda y localización de víctimas utilizando un dron simulado. El dron recorre una zona marcada, detecta rostros humanos desde la cámara inferior (visión ventral) y calcula sus posiciones reales en el mapa usando trigonometría y el campo de visión (FOV) de la cámara.

## Generación del plan (zigzag)

Se generan puntos en zigzag con una función generate_matrix(x_min, y_min, x_max, y_max, step). La idea es recorrer filas y alternar dirección (→, ←) para cubrir la zona eficientemente.

Pequeño ejemplo de patrón:

```python
if left_to_right:
    plan_points.append((x_min, y))
    plan_points.append((x_max, y))
else:
    plan_points.append((x_max, y))
    plan_points.append((x_min, y))
```

Esta matriz evita zonas no exploradas y hace que el dron avance de forma ordenada, primero a la derecha, luego a la izquierda, bajando en cada iteración del patrón.

## Detección de víctimas

Para detectar rostros humanos usamos Haar Cascade de OpenCV:

```python
face_cascade = cv.CascadeClassifier('haarcascade_frontalface_default.xml')
faces = face_cascade.detectMultiScale(gray, scaleFactor=1.2, minNeighbors=8)
```
