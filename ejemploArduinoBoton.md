# Ejemplo en Arduino de Blink + Botón que enciende un led.

En este ejemplo trabajaremos con [TinkerCAD](https://www.tinkercad.com/) 

## Circuito 1: Blink

El **Circuito Blink** es el más conocido, ya que nos lleva a hacer un "Hola Mundo" al lenguaje Arduino (incluso tiene un led interno en el pin 13) para poder probar el funcionamiento de lo que tenemos armado.

Para abrir el código base, puedes copiarlo desde acá o puedes hacerlo directo desde **Arduino IDE** en `file` > `examples` > `01. Basics` > `Blink`.

### 1.1 Circuito Blink

<p align="center">
  <img src="https://www.sebastiancabezas.cl/imgs/externas/circuito_blink.webp" alt="Circuito Blink" width="500">
</p>

### 1.2 Código Blink

```ino
int pinLED = 13; // se define el pin que estaremos ocupando para el led

void setup()
{
  pinMode(pinLED, OUTPUT); // se configura el pin, indicando que es de salida.
}

void loop()
{
	blink(pinLED, 1); // se llama al método blink, le pasamos el ped y la cantidad de segundos que quermos que esté encendido
}

void blink(int pin, int seconds){
  	digitalWrite(pin, HIGH); // escribe de manera digital en el pin: ALTO para encender
  	delay(seconds*1000); // espera durante 1 segundo * 1000 ya que está en milisegundos
  	digitalWrite(pin, LOW); // ahora indica que se apaga
  	delay(seconds*1000); // espera durante 1 segundo más y así sucesivamente
}
```

## Circuito 2: Botón + LED

El **Circuito Botón + LED** será un circuito que nos permitirá leer digitalmente un **PIN** para transformar la señal en el **encendido** de un LED.

### 2.1 Circuito Botón + LED

<p align="center">
  <img src="https://www.sebastiancabezas.cl/imgs/externas/circuito_boton_led.webp" alt="Circuito Boton + LED" width="900">
</p>

### 2.2 Código Blink

```ino
int pinLED = 13;
int btnLED = 10;
int btnValor = 0;
int btnPulsador = 8;

void setup()
{
  Serial.begin(9600); // Se inicializa la comunicación Serial para trabajar con el monitor y obtener salidas.
  pinMode(pinLED, OUTPUT);
  pinMode(btnLED, OUTPUT);
  pinMode(btnPulsador, INPUT_PULLUP); // Declaramos la variable inputPin como entrada y activamos su resistencia interna Pullup.
}

void loop()
{
    btnValor = digitalRead(btnPulsador);
    Serial.print("Valor del BTN: ");
    Serial.println(btnValor);
    if (btnValor == LOW){
      digitalWrite(pinLED, HIGH);
      digitalWrite(btnLED, HIGH);
    }else{
      digitalWrite(pinLED, LOW);
      digitalWrite(btnLED, LOW);
    }
}
```

