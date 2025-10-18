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




https://drive.google.com/file/d/1NV8W7KwN0Wq7s63KUYd_BSC5fKGeNKhO/view?usp=sharing
