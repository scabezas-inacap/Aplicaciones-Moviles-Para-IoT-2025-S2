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

```ino
// 1. Agregar la biblioteca
#include <Arduino.h>
#include <Firebase_ESP_Client.h>
#if defined(ESP32)
  #include <WiFi.h>
#endif

void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  Serial.println("Hello, ESP32!");
}

void loop() {
  // put your main code here, to run repeatedly:
  delay(10); // this speeds up the simulation
}
```

Start Simulation :)
