// C++ code
// Proyecto Final - Laboratorio de Computacion I (UNSAM)
// Velador Inteligente con Regulador de Brillo (Dimmer) y Timer de Auto-Apagado Gradual
// Integrantes: Maria Constanza De Carlo, Ivan Saipert

#define LEDLAMPARA 5      // Salida analogica (PWM) - Luz principal del velador
#define LEDPILOTO 8       // Salida digital - Luz testigo que indica que el timer esta activo

#define BOTON 4           // Entrada digital - Pulsador de cambio de modo
#define POTENCIOMETRO A0  // Entrada analogica - Perilla reguladora de brillo

#define ESTADOAPAGADO 0
#define ESTADOMANUAL 1
#define ESTADOTIMER 2

// --- CONFIGURACION DEL TIEMPO DEL TIMER (Control Logico por Tiempo) ---
// Modo Demo Laboratorio (30 segundos totales): 30.000 ms / 255 niveles = 118 ms por nivel
#define TIEMPODECREMENTO 118

// Modo Uso Real (5 minutos totales): 300.000 ms / 255 niveles = 1176 ms por nivel
// #define TIEMPODECREMENTO 1176

int estado = ESTADOAPAGADO;
int estadoBotonAnterior = LOW;
int contadorClicks = 0;   // Contador de flancos ascendentes registrados
int brilloTimer = 0;      // Nivel de brillo actual durante el apagado gradual

unsigned long millisAnteriores = 0;

void setup()
{
  pinMode(LEDLAMPARA, OUTPUT);
  pinMode(LEDPILOTO, OUTPUT);
  pinMode(BOTON, INPUT);
  pinMode(POTENCIOMETRO, INPUT);

  Serial.begin(9600);
}

void loop()
{
  unsigned long millisActuales = millis();

  // 1. Lectura de entradas (Digital y Analogica)
  int estadoBoton = digitalRead(BOTON);
  int lecturaPote = analogRead(POTENCIOMETRO);

  // Escalamos la lectura del potenciometro (0 a 1023) al rango PWM del LED (0 a 255)
  int brilloManual = lecturaPote / 4;

  // 2. Detector y Contador de Flancos Ascendentes (cambio de LOW a HIGH)
  int clickBoton = 0;
  if ((estadoBoton == HIGH) && (estadoBotonAnterior == LOW))
  {
    contadorClicks++;
    clickBoton = 1;
    estadoBotonAnterior = estadoBoton;
  }
  else
  {
    estadoBotonAnterior = estadoBoton;
  }

  // Monitoreo en tiempo real por Puerto Serie
  Serial.print("Clicks (Flancos): ");
  Serial.print(contadorClicks);
  Serial.print(" | Brillo Pote: ");
  Serial.print(brilloManual);
  Serial.print(" | Brillo Timer: ");
  Serial.print(brilloTimer);
  Serial.print(" | Estado: ");
  Serial.println(estado);

  // 3. Maquina de Estados: Cambios de estado (Transiciones)
  switch (estado)
  {
    case ESTADOAPAGADO:
      // Con 1 click (flanco) pasa al modo de encendido manual
      if (clickBoton == 1)
      {
        estado = ESTADOMANUAL;
      }
      break;

    case ESTADOMANUAL:
      // Con otro click activa el Timer de apagado gradual
      if (clickBoton == 1)
      {
        estado = ESTADOTIMER;
        brilloTimer = brilloManual; // Toma como punto de partida el brillo actual
        millisAnteriores = millisActuales;
      }
      break;

    case ESTADOTIMER:
      // Si el usuario hace click para cancelar o si el brillo llega a 0, se apaga
      if ((clickBoton == 1) || (brilloTimer <= 0))
      {
        estado = ESTADOAPAGADO;
      }
      break;
  }

  // 4. Maquina de Estados: Salidas por estado (Acciones)
  switch (estado)
  {
    case ESTADOAPAGADO:
      analogWrite(LEDLAMPARA, 0);
      digitalWrite(LEDPILOTO, LOW);
      break;

    case ESTADOMANUAL:
      analogWrite(LEDLAMPARA, brilloManual);
      digitalWrite(LEDPILOTO, LOW);
      break;

    case ESTADOTIMER:
      digitalWrite(LEDPILOTO, HIGH);
      analogWrite(LEDLAMPARA, brilloTimer);

      // Control logico por tiempo sin bloquear el microcontrolador
      if ((millisActuales - millisAnteriores) >= TIEMPODECREMENTO)
      {
        brilloTimer--;
        millisAnteriores = millisActuales;
      }
      break;
  }

  delay(20);
}
