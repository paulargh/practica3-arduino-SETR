# practica3-arduino-SETR
# Controlador Máquina Expendedora
## Conexión y pines del circuito:
El circuito está montado sobre una placa Arduino Mega 2560:

- **LED1 (Rojo):** Conectado al pin digital 24 (con una resistencia a GND), encargado de parpadear 3 veces (1s) en el arranque y mantenerse fijo en el modo administración.

- **LED2 (Azul):** Conectado al pin PWM 3 (con una resistencia a GND), encargado de aumentar su brillo progresivamente mientras se prepara el café, además de mantenerse fijo en el modo administración.

- **Botón externo:** Conectado al pin digital 2, se utiliza este pin porque soporta interrupciones hardware, para medir pulsaciones largas (de 2–3 (s) o ≥ 5 (s)).

- **Sensor de ultrasonidos:** El pin TRIG va conectado al pin digital 9 y el pin ECHO al pin digital 10, además de su alimentación a 5V y GND.

- **Pantalla LCD:** Configurada con RS en el pin 12, E en el pin 11 y los pines de datos D4 a D7 conectados a los pines digitales 4, 5, 6 y 7, con RW a GND (escribir texto).

- **Joystick:** Los ejes analógicos VRx y VRy van conectados a A0 y A1 respectivamente para moverse por los menús. El pulsador central (SW) va al pin digital 8, configurado con resistencia pull-up interna (INPUT_PULLUP), para fijar la lectura en reposo a HIGH (5V) y al pulsar pasa a LOW (0V), evitando falsas pulsaciones.

- **Sensor de temperatura y humedad (DHT11):** Conectado a 5V, GND y el pin de datos (DATA) al pin A5. El sensor contiene internamente su propia resistencia.

## Descripción de la solución y los recursos utilizados:

Para conseguir que el sistema responda rápido, no se quede bloqueado y gestione a la vez los sensores, los botones y la pantalla, combino el uso de threads, interrupciones hardware y watchdog.

### 1. Uso de hilos para organizar las tareas (ArduinoThread)
Para no llenar el bucle principal (loop) de temporizadores o usar funciones bloqueantes (delay()), uso la librería ArduinoThread junto con un controlador ThreadController. Cada componente se ha separado en su propia clase heredada de Thread, ejecutándose cada una con su propio intervalo de tiempo.

- **HiloLed:** Controla el parpadeo de 3 segundos del LED rojo en el arranque y el aumento progresivo de brillo por PWM del LED azul mientras se prepara el café. Al gestionarlo en este thread, evito usar interrupciones como TimerOne.

- **HiloLcd:** Gestiona la actualización de la pantalla LCD y la lectura del joystick cada 80 ms. Solo actualiza la pantalla cuando cambia el texto para evitar parpadeos, e incluye un bloqueo por software en el joystick para obligar a soltarlo antes de registrar una nueva pulsación.

- **HiloUltrasonidos:** Mide la distancia con el sensor cada 200 ms usando un tiempo límite (timeout) de 30 ms para evitar que el programa se quede colgado si no rebota el eco. Además, necesita 2 lecturas seguidas válidas (< 100 cm) antes de confirmar que hay un cliente delante.

- **HiloTemperatura:** Lee la temperatura y la humedad del sensor DHT11 cada 2 segundos, respetando el tiempo mínimo de lectura del propio sensor y filtrando posibles lecturas erróneas.

### 2. Interrupción hardware para el botón externo (attachInterrupt)
Para el botón externo (pin 2) uso una interrupción hardware por cambio de flanco (CHANGE). Como el sistema tiene que medir si se mantiene pulsado entre 2 y 3 segundos (para reiniciar el servicio) o 5 segundos o más (para entrar o salir del modo administración) en cualquier momento, leerlo en el loop haría que se perdieran pulsaciones, por eso uso interrupciones. Dentro de la interrupción aplico un filtro antirrebotes de 50 ms, guardo los tiempos de pulsación y activo un flag, dejando la comprobación de si la pulsación es válida o no para el loop.

### 3. Watchdog (avr/wdt.h)
Como medida de seguridad frente a bloqueos, uso Watchdog con un tiempo límite de 8 segundos (WDTO_8S) justo al terminar el setup(), desactivándolo primero con wdt_disable() para evitar reinicios durante el arranque. En cada vuelta del loop reinicio el contador con wdt_reset(), asegurando que todo funciona correctamente. Pongo 8 segundos para dar margen a las lecturas de los sensores antes de reiniciar la placa.

Finalmente, todo esto se coordina en el loop() (arranque, espera de cliente, temperatura, selección de productos, preparación, modo administración), permitiendo un funcionamiento fluido y sin bloqueos.

## Esquemático
<img width="881" height="628" alt="image" src="https://github.com/user-attachments/assets/60e0fe5a-e750-42ec-9b06-119f5d958cda" />

