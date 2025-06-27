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

