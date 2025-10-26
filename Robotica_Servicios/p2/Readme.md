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

Antes de detectar caras, filtro zonas azules (debido a las olas del mar) para evitar falsos positivos:

```python
mask_blue = cv.inRange(hsv, lower_blue, upper_blue)
filtered = cv.bitwise_and(rotated_frame, rotated_frame, mask_not_blue)
```

También rotamos la imagen varias veces para detectar caras aunque estén ligeramente inclinadas, esto debido a que el algoritmo solo detecta caras cuando estan en vertical:

```python
for angle in range(0, 360, 20):
rot_matrix = cv.getRotationMatrix2D(center, angle, 1.0)
```

## Conversión de píxeles a coordenadas reales

Detectar la cara es fácil. Lo difícil era convertir la posición del píxel donde se detecta la víctima a coordenadas reales del mapa. Para eso necesité conocer el campo de visión (FOV) real de la cámara ventral.

### Cálculo real del FOV

Probé hacerlo con datos teóricos de cámara, pero no coincidían con el simulador. Así que hice algo mejor:

    1. Subí el dron a una altura conocida (5.5 metros).
    
    2. Medí en el simulador cuántos metros veía en horizontal y vertical. Resultado:
    
        · Ancho visible ≈ 6 metros
        · Alto visible ≈ 4 metros
        
    3. Apliqué trigonometría inversa:

```math
    FOV = 2 \cdot \arctan\left(\frac{tamaño\_visible / 2}{altura}\right)
```

Asi obtuve unos valores para `CAMERA_FOV_X` y `CAMERA_FOV_Y`, sin embargo, al no ser exactos los valores de la anchura y altura que escogi, tuve que hacer algunas pruebas hasta dar con el mas adecuado.

## Gestión de batería

El dron nunca se queda tirado sin batería. Automáticamente regresa a base si baja del 20%:

```python
if battery_level < LOW_BATTERY_THRESHOLD:
    state = GO_BACK
```
Dependiendo del número de iteraciones del algoritmo, el nivel de batería debería disminuir progresivamente. Si la lancha está demasiado lejos o el dron requiere muchas iteraciones para alcanzar el objetivo, es posible que no llegue a aterrizar por falta de batería.

## Video de muestra

[ACCEDER AL VIDEO](https://drive.google.com/file/d/171781H9LIfBmLJR4g09K8_iJNTM9c5MS/view?usp=sharing)










