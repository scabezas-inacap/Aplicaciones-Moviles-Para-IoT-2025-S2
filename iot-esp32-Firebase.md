# ESP32 conectado a una RTDB (RealTime Database) de Firebase.

Entenderás cómo desacoplar el "Frontend" (App Android) del "Backend/Hardware" (Arduino + Firebase) que es fundamental en IoT.

No necesitas la App Android todavía. Puedes actuar en modo `Dios` directamente desde la consola de Firebase para simular las peticiones que haría la aplicación.

Utilizando **ESP32** o ***ESP8266*** (ya que el Arduino Uno estándar no tiene WiFi nativo, y usar shields complica el código).

## Antes de Comenzar

* Crea una cuenta en [wokwi.com](https://wokwi.com). Puedes usar tu cuenta *GMAIL* o *GitHub*
* Ten instalado [VSCode](https://code.visualstudio.com/download). Necesitaremos algunas extensiones.

## 1\. ¿Cómo simular la App?

Para encender el LED sin tener la aplicación, usarás la Consola Web de Firebase como si fuera el control remoto:

1. Entra a tu proyecto en [Firebase Console](https://console.firebase.google.com/).
2. Ve a la sección **Realtime Database** en el menú izquierdo.
3. Verás tu árbol de datos JSON. Pasa el mouse sobre la raíz y haz clic en el botón `+`.
4. Crea un campo (clave) llamado `estado_led` (o el nombre que quieras).
5. Asignale un valor inicial: `"OFF"` o `0` o `false`.
6. **La simulación:** Cuando tu Arduino esté corriendo, simplemente cambia ese valor manualmente en esta web a `"ON"` (o `1` o `true`) y presiona Enter. Firebase sincronizará ese cambio con el Arduino casi instantáneamente.

-----

## 2\. Requisitos Previos para el Laboratorio

* **Hardware**: Placa ESP32. Se programa igual que Arduino y tienen WiFi integrado.

* **Software**: Arduino IDE configurado para estas placas.

* **Bibliotecas**: Instala la biblioteca `Firebase Arduino Client Library for ESP8266 and ESP32` de Mobizt desde el Gestor de Bibliotecas. Es la más estable actualmente.

-----

## 3\. Configuración Crítica en Firebase

Para que esto funcione sin lidiar con autenticación compleja (haremos esto sólo al principio):

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
      * **Web API Key:** Está en Configuración del Proyecto (engranaje) -\> Cuentas de servicio -\> Secretos de la base de datos (o en Configuración general -\> Web API Key).

-----
## 4\. Paso a Paso

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
-----
#6\.**Wokwi es perfecto para esto**. De hecho, para un entorno educativo es incluso mejor porque elimina los problemas de drivers USB, cables rotos y permite que los estudiantes practiquen en casa sin hardware físico.

Wokwi tiene una característica llamada "Virtual WiFi" que conecta el ESP32 simulado a internet a través de la conexión de tu computadora.

Aquí tienes los ajustes específicos para que el código anterior funcione en Wokwi:

### 1\. Cambio en las Credenciales WiFi

En Wokwi, la red WiFi simulada **siempre** tiene el mismo nombre y contraseña (vacía). Debes cambiar estas dos líneas en el código que te pasé antes:

```cpp
// Credenciales OBLIGATORIAS para Wokwi
#define WIFI_SSID "Wokwi-GUEST"
#define WIFI_PASSWORD ""
```

### 2\. Cómo instalar la biblioteca en Wokwi

Como no tienes el "Gestor de bibliotecas" tradicional, debes decirle a Wokwi qué bibliotecas usar:

1.  En la interfaz de Wokwi, busca la pestaña **"Library Manager"** (icono de biblioteca a la izquierda) o el archivo `diagram.json` si eres usuario avanzado.
2.  Haz clic en el botón **"+"** (Add a new library).
3.  Escribe y selecciona: **"Firebase Arduino Client Library for ESP8266 and ESP32"** (la misma de Mobizt).

### 3\. Tu esquema de conexión (Diagrama)

Solo necesitas:

  * Una placa **ESP32** (la opción por defecto en Wokwi).
  * Un **LED** (búscalo con el signo `+`).
  * Una **Resistencia** de 220 ohm (opcional en simulación, pero buena práctica).
  * Conecta el **Ánodo** del LED (pata larga) al **Pin D2** (o el que definas en el código).
  * Conecta el **Cátodo** a **GND**.

### Resumen de la dinámica en Wokwi

1.  Pega el código completo (con el cambio de WiFi "Wokwi-GUEST").
2.  Añade la biblioteca desde el gestor.
3.  Dale al botón "Play" (verde).
4.  Verás en la consola "Connecting to WiFi... Connected".
5.  Ve a tu consola de Firebase real, cambia el valor, y verás el LED virtual encenderse en la pantalla.

Aquí tienes un video que muestra cómo funciona la conexión a internet y APIs externas dentro del simulador Wokwi, lo cual valida que podrás conectar con Firebase sin problemas:

[Montaje 20: Programado un ESP32 y conectando a WiFI en Wokwi con APIs](https://www.youtube.com/watch?v=xHyhqIvujdw)

Este video es relevante porque demuestra empíricamente que el ESP32 virtual en Wokwi tiene salida real a internet para consultar datos externos.

