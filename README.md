[readme.md](https://github.com/user-attachments/files/35051957/readme.md)
# TPFinal_GrupoN_2026# 
1. Título y Descripción del Sistema

**Velador Inteligente con Regulador de Brillo (Dimmer) y Timer de Auto-Apagado Gradual**

El sistema consiste en una luminaria inteligente controlada por una placa **Arduino UNO** y programada en lenguaje C. Permite al usuario encender la lámpara, regular su intensidad lumínica en tiempo real mediante una perilla analógica (potenciómetro) y activar un modo de apagado automático temporizado (*Sleep Timer*). En este modo temporizado, la luz disminuye su brillo gradualmente utilizando temporizadores no bloqueantes (`millis()`) hasta apagarse por completo sin bloquear la lectura de las entradas del sistema.

# 2. Integrantes del Grupo

* María Constanza De Carlo
* Iván Saipert

# 3. Materia

* **Institución:** Universidad Nacional de San Martín (UNSAM) — Escuela de Ciencia y Tecnología (ECyT)
* **Materia:** Laboratorio de Computación I

# 4. Descripción General y Funcionalidades

El proyecto resuelve el control de iluminación de una habitación integrando control manual y automatización por tiempo:

* **Encendido y cambio de modos por un único botón:** Mediante detección de flancos ascendentes, un único pulsador permite ciclar prolijamente entre los estados del sistema (Apagado $\rightarrow$ Manual $\rightarrow$ Timer $\rightarrow$ Apagado).
* **Regulación de brillo en vivo (Dimmer PWM):** En el modo manual, el usuario gira el potenciómetro para ajustar el brillo del LED principal entre el $0\%$ y el $100\%$.
* **Apagado gradual temporizado (*Fade-Out*):** Al pasar al modo Timer, se enciende un LED piloto testigo y el sistema toma el nivel de brillo actual para reducirlo punto por punto a intervalos constantes de tiempo.
  * *Configuración Uso Real:* $5\text{ minutos}$ ($300.000\text{ ms}$, decrementando $1$ nivel cada $1176\text{ ms}$).
  * *Configuración Modo Demo (Laboratorio):* $30\text{ segundos}$ ($30.000\text{ ms}$, decrementando $1$ nivel cada $118\text{ ms}$).
* **Cancelación instantánea:** Al no utilizar `delay()` bloqueantes para el temporizador, el usuario puede presionar el botón en cualquier momento durante el conteo para apagar la lámpara inmediatamente.
* **Telemetría por Puerto Serie:** El sistema informa en tiempo real por el Monitor Serie (`9600 baudios`) la cantidad total de pulsaciones registradas, el brillo configurado, el brillo restante del timer y el estado actual.

# 5. Requisitos Funcionales Cumplidos

1. **Control de Entradas y Salidas (Digitales y Analógicas):**
   * *Entrada Digital:* Pulsador conectado al `Pin 4` (`digitalRead`).
   * *Entrada Analógica:* Potenciómetro conectado al `Pin A0` (`analogRead`, rango $0$ a $1023$).
   * *Salida Digital:* LED Piloto indicador de Timer en el `Pin 8` (`digitalWrite`).
   * *Salida Analógica (PWM):* LED Principal del velador en el `Pin 5` (`analogWrite`, rango $0$ a $255$).
2. **Contador de Flancos:**
   * Implementado sobre la entrada digital del pulsador (`Pin 4`). Compara el estado actual con el estado anterior (`estadoBoton == HIGH && estadoBotonAnterior == LOW`), incrementando la variable `contadorClicks` únicamente en el flanco de subida.
3. **Control Lógico por Tiempo:**
   * Implementado mediante la función `millis()` y la variable `millisAnteriores`. Evalúa la condición `(millisActuales - millisAnteriores) >= TIEMPODECREMENTO` para reducir progresivamente el brillo de la lámpara sin detener el ciclo de ejecución del microcontrolador.
4. **Control Lógico por Máquina de Estados:**
   * Estructurado mediante dos bloques `switch(estado)` independientes: uno para evaluar las transiciones de estado y otro para ejecutar las salidas correspondientes a cada estado (`ESTADOAPAGADO`, `ESTADOMANUAL`, `ESTADOTIMER`).

# 6. Componentes Utilizados

| Cantidad | Componente | Función en el Sistema |
| ----- | ----- | ----- |
| 1 | Placa Arduino UNO R3 | Unidad de control principal |
| 1 | Protoboard (Placa de pruebas) | Conexionado del circuito |
| 1 | Pulsador (*Pushbutton*) | Entrada digital para cambio de estado y contador de flancos |
| 1 | Potenciómetro ($10\text{ k}\Omega$) | Entrada analógica para regulación del nivel de brillo |
| 1 | LED Amarillo / Blanco | Salida analógica (PWM) - Luz principal del velador |
| 1 | LED Rojo | Salida digital - Luz testigo indicadora de Modo Timer activo |
| 2 | Resistencias de $220\ \Omega$ | Limitación de corriente para protección de ambos LEDs |
| 1 | Resistencia de $10\text{ k}\Omega$ | Configuración *Pull-Down* para la entrada del pulsador |
| Varios | Cables *Jumper* | Conexión entre placa y protoboard |

# 7. Esquema Eléctrico y Diagrama de Conexiones

### Tabla de Pinout (Conexiones al Arduino UNO)

* **Pin 4 (Digital IN):** Conectado al terminal de salida del Pulsador (con resistencia *pull-down* de $10\text{ k}\Omega$ a `GND` y el otro extremo a `5V`).
* **Pin 5 (Digital PWM OUT `~`):** Conectado al Ánodo del LED Principal (Cátodo conectado a `GND` mediante resistencia de $220\ \Omega$).
* **Pin 8 (Digital OUT):** Conectado al Ánodo del LED Piloto Rojo (Cátodo conectado a `GND` mediante resistencia de $220\ \Omega$).
* **Pin A0 (Analog IN):** Conectado al terminal central (cursor) del Potenciómetro (terminales extremos conectados a `5V` y `GND`).

# 8. Máquina de Estados

El software se divide en **3 estados principales** definidos mediante macros (`#define`):

```
                  [ Click en Botón (Flanco) ]
  +---------------------------------------------------------+
  |                                                         v
+-------------------+   Click (Flanco)   +-------------------+
|  0: ESTADOAPAGADO | <----------------- |   2: ESTADOTIMER  |
|  - Lámpara: 0     |   ó Brillo <= 0    |  - Lámpara: Baja  |
|  - Piloto: LOW    |                    |    1 pto c/ 118ms |
+-------------------+                    |  - Piloto: HIGH   |
          |                              +-------------------+
          | Click (Flanco)                         ^
          v                                        |
+-------------------+       Click (Flanco)         |
|  1: ESTADOMANUAL  | -----------------------------+
|  - Lámpara: Pote  |  (Copia brilloTimer = brilloManual
|  - Piloto: LOW    |   y reinicia millisAnteriores)
+-------------------+
```

1. **`ESTADOAPAGADO (0)`:** Estado inicial de reposo. El LED principal recibe un valor PWM de `0` y el LED piloto permanece en `LOW`. Al detectar un flanco ascendente en el pulsador (`clickBoton == 1`), transiciona a `ESTADOMANUAL`.
2. **`ESTADOMANUAL (1)`:** El LED piloto permanece en `LOW` y el LED principal actualiza su brillo en tiempo real según la lectura del potenciómetro (`lecturaPote / 4`). Al detectar un nuevo flanco en el botón, guarda el brillo actual en `brilloTimer`, reinicia el cronómetro (`millisAnteriores = millisActuales`) y transiciona a `ESTADOTIMER`.
3. **`ESTADOTIMER (2)`:** Se enciende el LED piloto (`HIGH`) y cada vez que transcurren `TIEMPODECREMENTO` milisegundos se descuenta una unidad a `brilloTimer`. Si `brilloTimer` llega a `0` o si el usuario presiona el botón nuevamente (`clickBoton == 1`), el sistema regresa a `ESTADOAPAGADO`.

# 9. Instrucciones de Uso y Ejecución

### En Simulador Tinkercad:

1. Abrir el circuito en Tinkercad y pegar el código fuente en la pestaña **Código (Texto)**.
2. Presionar **Iniciar Simulación** y abrir el **Monitor en Serie** para observar las variables.
3. Hacer **1 click** en el pulsador para encender la lámpara en Modo Manual y girar la perilla del potenciómetro para regular la intensidad del LED.
4. Hacer **1 click** adicional en el pulsador para activar el Modo Timer: se encenderá el LED rojo piloto y la luz principal comenzará a atenuarse gradualmente hasta apagarse a los 30 segundos (o presionar el botón una tercera vez para apagarla de inmediato).

### En Hardware Físico (Arduino IDE):

1. Conectar la placa Arduino UNO por USB a la computadora.
2. Abrir el archivo del proyecto en Arduino IDE.
3. Seleccionar la placa `Arduino Uno` y el puerto `COM` correspondiente en el menú *Herramientas*.
4. Presionar **Subir (Upload)** y abrir el **Monitor Serie** configurado a `9600 baudios`.

# 10. Imágenes y Demostración

Las capturas del circuito funcionando en sus distintos estados y el esquema eléctrico se encuentran alojados en la carpeta `/media` y en el informe técnico dentro de la carpeta `/doc`.
