# practica3-arduino-SETR
# Controlador Máquina Expendedora
### Conexión y pines del circuito:
El circuito está montado sobre una placa Arduino Mega 2560:

- LED1 (Rojo): Conectado al pin digital 24 (con una resistencia a GND), encargado de parpadear 3 veces (1s) en el arranque y mantenerse fijo en el modo administración.

- LED2 (Azul): Conectado al pin PWM 3 (con una resistencia a GND), encargado de aumentar su brillo progresivamente mientras se prepara el café.

- Botón externo: Conectado al pin digital 2. Se utiliza este pin porque soporta interrupciones hardware, para medir pulsaciones largas (de 2–3 s o ≥ 5 s).

- Sensor de ultrasonidos: El pin TRIG va conectado al pin digital 9 y el pin ECHO al pin digital 10, además de su alimentación a 5V y GND.

- Pantalla LCD: Configurada con RS en el pin 12, E en el pin 11 y los pines de datos D4 a D7 conectados a los pines digitales 4, 5, 6 y 7, con RW a GND (escribir texto).

- Joystick: Los ejes analógicos VRx y VRy van conectados a A0 y A1 respectivamente para moverse por los menús. El pulsador central (SW) va al pin digital 8, configurado con resistencia pull-up interna (INPUT_PULLUP), para fijar la lectura en reposo a HIGH (5V) y al pulsar pasa a LOW (0V), evitando falsas pulsaciones.

- Sensor de temperatura y humedad (DHT11): Conectado a 5V, GND y el pin de datos (DATA) al pin A5. El sensor contiene internamente su propia resistencia.
