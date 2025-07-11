# Examen-electronica
desarrollo de circuito con arduino y motor paso a paso

## Pruebas con motor paso a paso y sensor LDR
~~~
#include <Stepper.h>

const int pinSensor = A0;
const int umbralSombra = 800;

const int pasosPorVuelta = 2048; 
Stepper motor(pasosPorVuelta, 8, 10, 9, 11);

void setup() {
  Serial.begin(9600);
  randomSeed(analogRead(1));  // Para que los movimientos sean aleatorios
  motor.setSpeed(10);  // Velocidad inicial
}

void loop() {
  int valorLuz = analogRead(pinSensor);
  Serial.println(valorLuz);

  if (valorLuz < umbralSombra) {
    Serial.println("Latido");

    // Latido aleatorio
    int pasosMax = random(100, 300); 
    int velocidadContraccion = random(15, 40);  // rapido
    int velocidadRelajacion = random(5, 20);    // lento
    int repeticiones = (random(0, 100) < 20) ? 2 : 1;  // Latido doble aleatorio

    for (int i = 0; i < repeticiones; i++) {
      // Contracción (adelante)
      motor.setSpeed(velocidadContraccion);
      motor.step(pasosMax);

      // Relajación (atrás)
      motor.setSpeed(velocidadRelajacion);
      motor.step(-pasosMax);

      delay(random(80, 500));  // Pausa aleatoria entre repeticiones
    }

    delay(random(700, 1400));  // Pausa final antes del siguiente latido
  } else {
    // Motor en reposo
    delay(50);
  }

  delay(10);  // Para estabilizar lecturas
}
~~~

# Programacion del motor paso a paso solo
~~~
#include <Stepper.h>

const int stepsPerRevolution = 2048;
Stepper motor(stepsPerRevolution, 8, 10, 9, 11);

// Fases: 0 = respiración, 1 = inquietud, 2 = fuga
int fase = 0;
unsigned long tiempoInicioFase = 0;

void setup() {
  randomSeed(analogRead(A0));
  tiempoInicioFase = millis();
}

// ---- FASE DURACIONES (en milisegundos) ----
const unsigned long duracionRespiracion = 25000;
const unsigned long duracionInquietud   = 35000;
const unsigned long duracionFuga        = 40000;

// ---- LOOP PRINCIPAL ----
void loop() {
  unsigned long ahora = millis();

  // Cambiar de fase cuando se cumple el tiempo
  if (fase == 0 && ahora - tiempoInicioFase > duracionRespiracion) {
    fase = 1;
    tiempoInicioFase = ahora;
  } else if (fase == 1 && ahora - tiempoInicioFase > duracionInquietud) {
    fase = 2;
    tiempoInicioFase = ahora;
  } else if (fase == 2 && ahora - tiempoInicioFase > duracionFuga) {
    fase = 0;
    tiempoInicioFase = ahora;
  }

  // Ejecutar comportamiento según la fase
  if (fase == 0) {
    respirar();
  } else if (fase == 1) {
    moverseInquieto();
  } else if (fase == 2) {
    intentarEscapar();
  }
}

// ---- FASE 1: RESPIRACIÓN ORGÁNICA ----
void respirar() {
  motor.setSpeed(5);      // muy lento
  motor.step(500);        // inhala
  delay(2000);
  motor.step(-500);       // exhala
  delay(2000);
}

// ---- FASE 2: INQUIETUD IRREGULAR ----
void moverseInquieto() {
  int dir = random(0, 2) == 0 ? -1 : 1;
  int pasos = random(300, 1000);     // desplazamiento visible
  int velocidad = random(8, 20);     // cambia el "ánimo"
  motor.setSpeed(velocidad);
  motor.step(dir * pasos);
  delay(random(4000, 12000));        // pausa larga, desconcertante
}

// ---- FASE 3: FUGA REPETITIVA ----
void intentarEscapar() {
  for (int i = 0; i < 4; i++) {
    motor.setSpeed(25);
    motor.step(700);                // siempre hacia adelante
    delay(5000);                   // pausa larga entre impulsos

    // De vez en cuando, sobresalto:
    if (random(0, 3) == 0) {
      motor.setSpeed(30);
      motor.step(300);             // como un espasmo
      delay(300);
    }
  }

  // Al terminar este ciclo de fuga, pasar a la siguiente fase
  fase = 0;
  tiempoInicioFase = millis();
}
~~~
~~~
#include <Stepper.h>

const int stepsPerRevolution = 2048;
Stepper motor(stepsPerRevolution, 8, 10, 9, 11);

// Fases: 0 = respiración, 1 = inquietud, 2 = fuga
int fase = 0;
unsigned long tiempoInicioFase = 0;

// ---- Duraciones por fase ----
const unsigned long duracionRespiracion = 25000;
const unsigned long duracionInquietud   = 35000;
const unsigned long duracionFuga        = 40000;

// ---- Movimiento general ----
unsigned long ultimoMovimiento = 0;

// ---- FUGA: variables internas ----
bool escapeEnPausa = false;
unsigned long tiempoUltimoEscape = 0;
int escapePasosRestantes = 4;
unsigned long duracionPausaActual = 0;

// ---- LÍMITES DE SEGURIDAD ----
const int VELOCIDAD_MAX = 20;
const int VELOCIDAD_MIN = 3;
const int PASOS_MAX = 1200;

void setup() {
  randomSeed(analogRead(A0));
  tiempoInicioFase = millis();
}

// ---- Función segura para mover el motor ----
void moverSeguro(int velocidad, int pasos) {
  velocidad = constrain(velocidad, VELOCIDAD_MIN, VELOCIDAD_MAX);
  pasos = constrain(pasos, -PASOS_MAX, PASOS_MAX);
  motor.setSpeed(velocidad);
  motor.step(pasos);
}

// ---- LOOP PRINCIPAL ----
void loop() {
  unsigned long ahora = millis();

  // Cambio de fase por duración
  if (fase == 0 && ahora - tiempoInicioFase > duracionRespiracion) {
    fase = 1;
    tiempoInicioFase = ahora;
  } else if (fase == 1 && ahora - tiempoInicioFase > duracionInquietud) {
    fase = 2;
    tiempoInicioFase = ahora;
    escapePasosRestantes = 4;
    escapeEnPausa = false;
    tiempoUltimoEscape = ahora;
    duracionPausaActual = random(8000, 15000);  // pausa inicial aleatoria
  } else if (fase == 2 && ahora - tiempoInicioFase > duracionFuga) {
    fase = 0;
    tiempoInicioFase = ahora;
  }

  // Ejecutar comportamiento por fase
  switch (fase) {
    case 0: respirar(); break;
    case 1: moverseInquieto(); break;
    case 2: intentarEscapar(); break;
  }
}

// ---------------------------
// FASE 0: Respiración
void respirar() {
  if (millis() - ultimoMovimiento >= 4000) {
    moverSeguro(5, 500);   // inhalar
    delay(2000);
    moverSeguro(5, -500);  // exhalar
    delay(2000);
    ultimoMovimiento = millis();
  }
}

// ---------------------------
// FASE 1: Inquietud
void moverseInquieto() {
  if (millis() - ultimoMovimiento >= random(4000, 12000)) {
    int dir = random(0, 2) == 0 ? -1 : 1;
    int pasos = random(300, 1000);
    int velocidad = random(5, 18);
    moverSeguro(velocidad, dir * pasos);
    ultimoMovimiento = millis();
  }
}

// ---------------------------
// FASE 2: Fuga con pausas aleatorias y dirección variable
void intentarEscapar() {
  unsigned long ahora = millis();

  if (escapePasosRestantes <= 0) {
    fase = 0;
    tiempoInicioFase = ahora;
    return;
  }

  if (escapeEnPausa) {
    if (ahora - tiempoUltimoEscape >= duracionPausaActual) {
      // Espasmo inesperado
      if (random(0, 3) == 0) {
        moverSeguro(18, 300);
        delay(300);
      }
      escapeEnPausa = false;
    }
  } else {
    int direccion = random(0, 2) == 0 ? -1 : 1;  // nueva: dirección aleatoria
    moverSeguro(12, direccion * 700);           // velocidad más suave
    escapePasosRestantes--;
    escapeEnPausa = true;
    tiempoUltimoEscape = ahora;
    duracionPausaActual = random(8000, 15000);  // pausa aleatoria prolongada
  }
}
~~~
