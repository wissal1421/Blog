# Formula1 sigue líneas, control reactivo PID

En esta práctica implementé un sistema de **seguimiento de línea roja** usando visión por computador y control PID. El objetivo era que el robot siguiera una línea roja de forma fluida, corrigiendo su trayectoria según la posición de la línea en la imagen.

## Detección del Color Rojo

En HSV, el color rojo está partido en dos rangos (alrededor de 0° y 360°).
Por eso defino dos máscaras para detectar correctamente todos los tonos de rojo:
```python
LOWER_RED_1 = np.array([0, 100, 100])
UPPER_RED_1 = np.array([10, 255, 255])
LOWER_RED_2 = np.array([160, 100, 100])
UPPER_RED_2 = np.array([180, 255, 255])
```

## Control PID

El control PID se encarga de corregir el error entre el centro de la cámara y la posición real de la línea.
Ajusté las constantes para equilibrar estabilidad y respuesta rápida:
```python
Kp = 0.02  # Proportional gain
Ki = 0.0001  # Integral gain
Kd = 2.00  # Derivative gain
```

El control proporcional corrige la dirección según el error actual, el integral acumula errores pequeños a lo largo del tiempo, y el derivativo suaviza los cambios bruscos evitando oscilaciones.Además, limito la velocidad angular para que el robot no gire demasiado rápido:
```python
ANGULAR_VELOCITY_LIMIT = 4
```

## Análisis de la imagen
```python
def analyze_image():
    image = HAL.getImage()
    hsv_image = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

    mask1 = cv2.inRange(hsv_image, LOWER_RED_1, UPPER_RED_1)
    mask2 = cv2.inRange(hsv_image, LOWER_RED_2, UPPER_RED_2)
    red_mask = cv2.bitwise_or(mask1, mask2)

    red_filtered = cv2.bitwise_and(image, image, mask=red_mask)
    WebGUI.showImage(red_filtered)

    return red_mask, image.shape
```
Aquí convierto la imagen de la cámara a HSV, aplico las dos máscaras de rojo, y muestro una imagen de depuración para comprobar visualmente qué está detectando el robot.

## Extracción del punto de referencia

En lugar de analizar toda la imagen (que es costoso), busco la línea roja en una fila horizontal cercana al centro. Eso me da un punto de referencia para calcular el error respecto al centro:
```python
target_row = int(height / 2) + 10
# Recorro columnas buscando píxeles rojos
```
Si no detecto línea en esa fila, activo una búsqueda profunda en toda la imagen como fallback.

## Ajuste inteligente de velocidad

Algo importante: no sirve de nada girar bien si el robot entra pasado de velocidad en una curva. Así que reduje la velocidad lineal según el error:
```python
v = MAX_V / (1 + 0.15 * abs(error))
```

## Lógica del bucle principal
```python
1. Capturar cámara
2. Detectar rojo
3. Si no hay rojo → buscar
4. Calcular error
5. Aplicar PID
6. Ajustar velocidades
```

## Video de muestra

[Accede para ver como se ejecuta el codigo :)](https://drive.google.com/file/d/1NV8W7KwN0Wq7s63KUYd_BSC5fKGeNKhO/view?usp=sharing)
