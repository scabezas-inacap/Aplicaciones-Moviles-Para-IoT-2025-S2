# Configuración de Wokwi con VSCode

Descarga el archivo de [firmware](https://github.com/scabezas-inacap/Aplicaciones-Moviles-Para-IoT-2025-S2/blob/main/firmware.uf2) y déjalo en la misma carpeta de tu proyecto.

Haz la ruta de archivos:
```ruta
Mi_Proyecto/
├── diagram.json        <-- Configura el circuito visual (Tu primer bloque)
├── wokwi.toml          <-- Conecta el binario
├── firmware.uf2        <-- Firmware de Arduino
└── sketch.ino          <-- Tu código C++
```

En wokwi.toml:

```toml
[wokwi]
version = 1
firmware = 'firmware.uf2'
```
En diagram.json
```json
{
  "version": 1,
  "author": "Sebastián Cabezas Ríos",
  "editor": "wokwi",
  "parts": [
    { "type": "board-esp32-devkit-c-v4", "id": "esp", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-led", "id": "led1", "top": 15.6, "left": 176.6, "attrs": { "color": "red" } },
    {
      "type": "wokwi-resistor",
      "id": "r1",
      "top": 99.95,
      "left": 144,
      "attrs": { "value": "220" }
    }
  ],
  "connections": [
    [ "esp:TX", "$serialMonitor:RX", "", [] ],
    [ "esp:RX", "$serialMonitor:TX", "", [] ],
    [ "led1:A", "r1:2", "red", [ "v0" ] ],
    [ "led1:C", "esp:GND.3", "black", [ "v0" ] ],
    [ "esp:18", "r1:1", "red", [ "h0" ] ]
  ],
  "dependencies": {}
}
```

En sketch.ino
```ino
int pinLed = 18;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  Serial.println("Hello, ESP32!");
  pinMode(pinLed, OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  blink(pinLed, 1);
}

void blink(int pin, int seconds){
  digitalWrite(pin, HIGH);
  delay(seconds*1000);
  digitalWrite(pin, LOW);
  delay(seconds*1000);
}
```

Start Simulation :)
