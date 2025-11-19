# ESP32 conectado a una RTDB (RealTime Database) de Firebase.

Entenderás cómo desacoplar el "Frontend" (App Android) del "Backend/Hardware" (Arduino + Firebase) que es fundamental en IoT.

No necesitas la App Android todavía. Puedes actuar en modo `Dios` directamente desde la consola de Firebase para simular las peticiones que haría la aplicación.

Utilizando **ESP32** (ya que el Arduino Uno estándar no tiene WiFi nativo, y usar shields complica el código).

-----

## 1\.Antes de Comenzar

* Ten instalado la aplicación de [Arduino](https://www.arduino.cc/en/software/)
* Ten físicamente un ESP32.
* Ten un LED + resistencia de 220 Ohm + Protoboard + 2 cables macho macho.

-----

## 2\. Instalación de Biblioteca del ESP32 en Arduino

Es importante que se instalen las bibliotecas del ESP32 en Arduino, para eso deberás seguir los siguientes pasos:

1\. Copia el siguiente código:

```html
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

2\. Ve a `File` -\> `Preferences`

3\. Ingresa el código del punto uno en `Additional Board Manager URLs`

4\. Termina haciendo clic en `OK`.

-----

## 3\. Configura la nueva placa

1\. Abre el `Boards Manager`. Ve a `Tools` -\> `Board` -\> `Boards Manager`

2\. Busca `ESP32` y presiona en el botón instalar para `ESP32 by Espressif Systems`

3\. Tenemos que esperar a que quede instalado en el sistema. Se demorará unos segundos.

-----

## 4\. Prueba la instalación

Enchufa la placa ESP32 al computador. Cuando el Arduino IDE esté abierto, sigue los siguientes pasos:

1\. Selecciona la placa (board) en `Tools` -\> `Board menu` (en nuestro caso `DOIT ESP32 DEVKIT V1`)

2\. Selecciona el puerto de comunicación (Verás conectado a COM3 o COM4 como también /dev/ttyUSB0)

3\. Crea un nuevo sketch para hacer el código de blink:

```ino
const int ledPin = 32;

void setup() {
  pinMode(ledPin, OUTPUT);
}

void loop(){
  digitalWrite(ledPin, HIGH);
  delay(1000);
  digitalWrite(ledPin, LOW);
  delay(1000);
}
```
4\. Deberíamos tener parpadeando el led.

-----

## 5\. Configuración Crítica en Firebase

Entra a tu proyecto en [Firebase Console](https://console.firebase.google.com/).

### 5.1\. Crea la variable a cambiar en el RealTtime DataBase

1\. Ve a la sección **Realtime Database** en el menú izquierdo.
2\. Verás tu árbol de datos JSON. Pasa el mouse sobre la raíz y haz clic en el botón `+`.
3\. Crea un campo (clave) llamado `estado_led` (o el nombre que quieras).
4\. Asignale un valor inicial: `false`.

### 5.2\. Firebase

Para que esto funcione sin lidiar con autenticación compleja:

#### 5.2.1\. Autenticación

1\. Ve a `Authentication` > `Método de Acceso`

2\. Presiona en `Agregar nuevo proveedor`

3\. Pincha en `Anónimo` y `Habilita` la autenticación

#### 5.2.2\. RealTime DataBase


1.  **Reglas de Base de Datos:** Ve a la pestaña "Reglas" en Realtime Database y cámbialas a modo público (solo para pruebas):
    ```json
    {
      "rules": {
        ".read": true,
        ".write": true
      }
    }
    ```
    
2.  **Credenciales:** Necesitarás dos cosas para el código:
      * **URL de la Base de Datos:** (Ej: `https://tu-proyecto.firebaseio.com/`)
      * **Web API Key:** Está en `Configuración del Proyecto` (engranaje) -\> `Cuentas de servicio` -\> `Secretos de la base de datos` (o en Configuración general -\> `Web API Key`).
      * Puedes descargar el archivo `google-services.json` y ver el `api_key` > `current_key` > `key`

-----

## 6\. Paso a Paso

1\. Agregar la biblioteca `Firebase Arduino Client Library for ESP8266 and ESP32`:
```ino
Firebase Arduino Client Library for ESP8266 and ESP32
```
2\. En el archivo de programación, agrega la biblioteca `Arduino` + `Firebase`:
```ino
// 1. Agregar la biblioteca
#include <Arduino.h>
#include <Firebase_ESP_Client.h>
#if defined(ESP32)
  #include <WiFi.h>
#endif
```
3\. En el archivo de programación, agrega la configuración del Wifi y de Firebase la definición de API KEY + URL y las instancias de los objetos de Firebase
```ino
// 2. Credenciales de RED
#define WIFI_SSID "Wokwi-GUEST"
#define WIFI_PASSWORD ""

// 3. Firebase
#define API_KEY "[API KEY]"
#define DATABASE_URL "[URL]"

// 4. Definición de Objetos Firebase
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;
```

4\. Configuración de LEDs
```ino
// Pin del LED (En NodeMCU D4 suele ser el LED integrado, pero invertido)
// Ajusta según tu placa.
const int ledPin = 2; 

unsigned long sendDataPrevMillis = 0;
bool signupOK = false;
```

5\. Configuración `void setup()`
```ino
void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);

  // Conexión WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Conectando a WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Conectado con IP: ");
  Serial.println(WiFi.localIP());

  // Configuración Firebase
  config.api_key = API_KEY;
  config.database_url = DATABASE_URL;

  // Habilitar reconexión WiFi si se pierde
  Firebase.reconnectWiFi(true);

  /* Registrar usuario anónimo para obtener token (necesario en versiones nuevas de la lib) */
  if (Firebase.signUp(&config, &auth, "", "")) {
    Serial.println("Conexión a Firebase Exitosa");
    signupOK = true;
  } else {
    Serial.printf("%s\n", config.signer.signupError.message.c_str());
  }

  // Inicializar la librería
  Firebase.begin(&config, &auth);
}
```
6\. `void loop`
```ino
void loop() {
  // Verificamos si estamos listos y si ha pasado un pequeño tiempo (para no saturar)
  if (Firebase.ready() && signupOK) {
    
    // LEER el valor desde Firebase (Ruta: /estado_led)
    // Usamos getBool porque simularemos con true/false en la consola
    if (Firebase.RTDB.getBool(&fbdo, "/estado_led")) {
      
      if (fbdo.dataType() == "boolean") {
        bool ledStatus = fbdo.boolData();
        Serial.print("Estado recibido de Firebase: ");
        Serial.println(ledStatus ? "ENCENDIDO" : "APAGADO");

        // Actuar sobre el Hardware
        digitalWrite(ledPin, ledStatus ? HIGH : LOW);
      }

    } else {
      // Si falla la lectura, imprimir el error (útil para debug en clase)
      Serial.println(fbdo.errorReason());
    }
  }
  
  // Pequeño delay para estabilidad
  delay(500);
}
```
-----
## 5\. El Código para Arduino (ESP32)

Este código está listo para copiar y pegar en tu clase. Se conecta al WiFi, luego a Firebase, y se queda escuchando cambios en la variable `/estado_led`.

*Nota: Este ejemplo usa un **bool** (true/false) para simplificar la lógica.*

```cpp
#include <Arduino.h>
#if defined(ESP32)
  #include <WiFi.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>

// 1. Credenciales de RED y FIREBASE
#define WIFI_SSID "NOMBRE_DE_TU_WIFI"
#define WIFI_PASSWORD "CONTRASEÑA_WIFI"

#define API_KEY "PEGA_AQUI_TU_WEB_API_KEY_DE_FIREBASE"
#define DATABASE_URL "https://tu-proyecto-id.firebaseio.com/" 

// 2. Definición de Objetos Firebase
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

// Pin del LED (En NodeMCU D4 suele ser el LED integrado, pero invertido)
// Ajusta según tu placa.
const int ledPin = 2; 

unsigned long sendDataPrevMillis = 0;
bool signupOK = false;

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);

  // Conexión WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Conectando a WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Conectado con IP: ");
  Serial.println(WiFi.localIP());

  // Configuración Firebase
  config.api_key = API_KEY;
  config.database_url = DATABASE_URL;

  // Habilitar reconexión WiFi si se pierde
  Firebase.reconnectWiFi(true);

  /* Registrar usuario anónimo para obtener token (necesario en versiones nuevas de la lib) */
  if (Firebase.signUp(&config, &auth, "", "")) {
    Serial.println("Conexión a Firebase Exitosa");
    signupOK = true;
  } else {
    Serial.printf("%s\n", config.signer.signupError.message.c_str());
  }

  // Inicializar la librería
  Firebase.begin(&config, &auth);
}

void loop() {
  // Verificamos si estamos listos y si ha pasado un pequeño tiempo (para no saturar)
  if (Firebase.ready() && signupOK) {
    
    // LEER el valor desde Firebase (Ruta: /estado_led)
    // Usamos getBool porque simularemos con true/false en la consola
    if (Firebase.RTDB.getBool(&fbdo, "/estado_led")) {
      
      if (fbdo.dataType() == "boolean") {
        bool ledStatus = fbdo.boolData();
        Serial.print("Estado recibido de Firebase: ");
        Serial.println(ledStatus ? "ENCENDIDO" : "APAGADO");

        // Actuar sobre el Hardware
        digitalWrite(ledPin, ledStatus ? HIGH : LOW);
      }

    } else {
      // Si falla la lectura, imprimir el error (útil para debug en clase)
      Serial.println(fbdo.errorReason());
    }
  }
  
  // Pequeño delay para estabilidad
  delay(500);
}
```

Este video es relevante porque demuestra empíricamente que el ESP32 virtual en Wokwi tiene salida real a internet para consultar datos externos.

