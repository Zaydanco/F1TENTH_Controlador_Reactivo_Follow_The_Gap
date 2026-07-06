# Controlador Reactivo Follow the Gap - F1TENTH

## 1. Descripción del proyecto


Este proyecto implementa un controlador reactivo para el simulador F1TENTH en ROS 2 Humble. El objetivo principal es que el vehículo pueda recorrer el mapa BrandsHatch sin colisionar.

El controlador utiliza el enfoque **Follow the Gap**, que consiste en analizar las lecturas del LiDAR, detectar los espacios libres disponibles en la pista y dirigir el vehículo hacia el espacio más seguro.

Adicionalmente, el controlador cuenta con una función de contador tomando como inicio y fin el punto de partida al ejecutar el simulador. Un cronómetro por vuelta que indica el tiempo al momento de que el vehículo pase por el punto de inicio, un registro de cuál es la vuelta más rápida y por último una velocidad que cambia dependiendo de la situación del mapa.



## 2. Enfoque utilizado: Follow the Gap


El controlador hace uso del método Follow the Gap, el cual toma decisiones en tiempo real gracias a la información obtenida del sensor LIDAR y funciona de la siguiente manera:

1. El vehículo recibe datos del LiDAR desde el tópico `/scan`.
2. Se limpian las lecturas inválidas, como valores infinitos o NaN (no es un número por sus siglas en inglés).
3. Se analiza principalmente la zona frontal del vehículo.
4. Se identifica el obstáculo más cercano.
5. Se crea una burbuja de seguridad alrededor de ese obstáculo.
6. Se busca el espacio libre más grande disponible, lo que llamamos Gap.
7. Se selecciona un punto objetivo dentro del gap.
8. Se calcula el ángulo de dirección hacia ese punto.
9. Se ajusta la velocidad según el giro y el espacio libre disponible.
10. Se publica el comando de conducción en el tópico `/drive`.

Además, el controlador usa odometría mediante el tópico `/ego_racecar/odom` para contar vueltas y medir el tiempo de cada vuelta.



## 3. Tópicos utilizados


### Suscripciones

* `/scan`: recibe los datos del LiDAR.
* `/ego_racecar/odom`: recibe la posición del vehículo para contar vueltas.

### Publicación

* `/drive`: envía comandos de velocidad y dirección al vehículo usando `AckermannDriveStamped`.



## 4. Estructura del código


El archivo principal del controlador es:

```bash
gap_following/gap_following/gap_follower.py
```

Las partes principales del código son:

### `preprocess_lidar()`

Limpia las lecturas del LiDAR, reemplazando valores inválidos y suavizando los datos para evitar movimientos bruscos.

### `get_front_ranges()`

Selecciona solamente la zona frontal del LiDAR, ya que es la más importante para la conducción.

### `apply_safety_bubble()`

Crea una burbuja de seguridad alrededor del obstáculo más cercano para evitar que el vehículo pase demasiado cerca de las paredes.

### `find_max_gap()`

Busca el espacio libre más grande dentro de las lecturas frontales del LiDAR.

### `find_best_point()`

Selecciona el mejor punto dentro del gap. En este controlador se prioriza una zona cercana al centro del gap para evitar que el vehículo se cierre demasiado contra las paredes.

### `calculate_steering_angle()`

Convierte el punto objetivo en un ángulo de dirección. También suaviza el giro para evitar zigzags.

### `calculate_speed()`

Calcula una velocidad dinámica. El vehículo aumenta la velocidad en rectas y la reduce en curvas o cuando detecta poco espacio libre al frente.

### `odom_callback()`

Cuenta las vueltas usando la posición inicial del vehículo y registra el tiempo de cada vuelta.

### `print_final_results()`

Muestra el resumen final con el número de vueltas completadas y la vuelta más rápida.



## 5. Instrucciones de ejecución


Primero,antes que nada es necesario disponer de lo siguiente:
   
   -Ubuntu 22.04
   -ROS 2 Humble
   -Simulador F1TENTH instalado y configurado.
   -Mapa BrandsHatch.

Dentro del repositorio se cuenta únicamente con el paquete para el controlador reactivo. Por lo que el simulador debe estar previamente instalado. Para la instalación del controlador debemos copiar el paquete gap_following dentro de la carpeta src del workspace del simulador F1TENTH.

Luego, ya podemos entrar al repositorio:

```bash
cd ~/F1Tenth-Repository
source /opt/ros/humble/setup.bash
source install/setup.bash
```

Compilar el proyecto:

```bash
colcon build --symlink-install
source install/setup.bash
```

Ejecutar el simulador:

```bash
ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

En otra terminal, ejecutar el controlador:

```bash
cd ~/F1Tenth-Repository
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 run gap_following gap_following
```



## 6. Resultados obtenidos


El controlador logró completar las 10 vueltas consecutivas en el mapa BrandsHatch sin colisiones.

Durante la ejecución, el programa muestra en la terminal el número de vuelta, el tiempo de cada vuelta y la mejor vuelta registrada.

Ejemplo de salida:

```text
Vuelta 1 completada | Tiempo: 99.01 s | Mejor vuelta: 99.01 s
Vuelta 2 completada | Tiempo: 98.90 s | Mejor vuelta: 98.90 s
...
Vuelta 10 completada | Tiempo: 98.71 s | Mejor vuelta: 98.60 s
```

Al finalizar, se muestra un resumen:

```text
===== RESUMEN FINAL =====
Vueltas completadas: 10
Vuelta más rápida: vuelta 6 con 98.60 s
=========================
```



## 7. Conclusión


El controlador Follow the Gap permitió que el vehículo tomara decisiones de conducción en tiempo real usando únicamente la información del LiDAR. La burbuja de seguridad, la selección centrada del gap y la velocidad dinámica permitieron mejorar la estabilidad del vehículo y completar múltiples vueltas sin colisión. 
Así mismo, el uso de la odometría permitió implementar un sistema de cronometraje y conteo de vueltas que facilita el registro del desempeño del controlador.


## 8. Demostración en video.

En el siguiente enlace podrán visualizar el controlador puesto en práctica dentro del simulador.

https://youtu.be/4MRM1tIQsYI?si=UGL-mDDzQq9GBPyO
