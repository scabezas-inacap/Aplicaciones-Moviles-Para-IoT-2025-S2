# Control de temperatura Ideal

Haremos un código en Arduino para controlar la temperatura.

Tendremos indicadores LED que nos digan como un semáforo si estamos dentro de una temperatura ideal. Tendremos márgenes para FRIO y para CALOR.

Simule el sensor de temperatura con un potenciometro.

LEDS:
* Rojo-Frío Crítico
* Amarillo-Alerta de Frío
* Verde-Temperatura Ideal
* Verde-Temperatura Ideal
* Amarillo-Alerta de Calor
* Rojo-Calor Crítico

Condiciones:
* Si la temperatura está entre 15° y 20° -> Temperatura Ideal
* Si la temperatura está entre 10° y 14° -> Alerta de Frío
* Si la temperatura es menor a 10° -> Frío Crítico
* Si la temperatura está entre 21° a 25° -> Alerta de Calor
* Si la temperatura es mayor a 25° -> Calor Crítico

Código en Arduino
```ino
int izqLR = 10;
int izqLY = 9;
int izqLG = 8;

int izqRG = 7;
int izqRY = 6;
int izqRR = 5;

int analogoPotenciometro = A0;

void setup() {
  pinMode(izqLR, OUTPUT);
  pinMode(izqLY, OUTPUT);
  pinMode(izqLG, OUTPUT);

  pinMode(izqRG, OUTPUT);
  pinMode(izqRY, OUTPUT);
  pinMode(izqRR, OUTPUT);
  Serial.begin(9600); // Salida Serial
}

void loop() {
  int potenciometro;
  potenciometro = analogRead(analogoPotenciometro);
  //Serial.print("Lectura Potenciometro: ");
  //Serial.println(potenciometro);
  int temperatura = map(potenciometro, 0, 1023, 1,43);
  //delay(1000);
  Serial.println(temperatura);
  if (temperatura < 10){
    digitalWrite(izqLR, HIGH);
    digitalWrite(izqLY, LOW);
    digitalWrite(izqLG, LOW);

    digitalWrite(izqRG, LOW);
    digitalWrite(izqRY, LOW);
    digitalWrite(izqRR, LOW);
  }else if(temperatura >= 10 && temperatura <=14){
    digitalWrite(izqLR, LOW);
    digitalWrite(izqLY, HIGH);
    digitalWrite(izqLG, LOW);

    digitalWrite(izqRG, LOW);
    digitalWrite(izqRY, LOW);
    digitalWrite(izqRR, LOW);
  }else if(temperatura >= 15 && temperatura <= 20){
    digitalWrite(izqLR, LOW);
    digitalWrite(izqLY, LOW);
    digitalWrite(izqLG, HIGH);

    digitalWrite(izqRG, HIGH);
    digitalWrite(izqRY, LOW);
    digitalWrite(izqRR, LOW);
  }else if (temperatura >= 21 && temperatura <= 25){
    digitalWrite(izqLR, LOW);
    digitalWrite(izqLY, LOW);
    digitalWrite(izqLG, LOW);

    digitalWrite(izqRG, LOW);
    digitalWrite(izqRY, HIGH);
    digitalWrite(izqRR, LOW);
  }else{
    digitalWrite(izqLR, LOW);
    digitalWrite(izqLY, LOW);
    digitalWrite(izqLG, LOW);

    digitalWrite(izqRG, LOW);
    digitalWrite(izqRY, LOW);
    digitalWrite(izqRR, HIGH);
  }
}

```
