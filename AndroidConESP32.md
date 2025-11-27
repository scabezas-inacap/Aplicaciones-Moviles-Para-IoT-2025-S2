# Programación en Android en JAVA con ESP32

Con esta guía aprenderás a programar en Android con el Lenguaje JAVA, comunicando un ESP32.

Haremos un esquema en TinkerCAD para hacer el encendido y apagado de un LED desde una Aplicación en Android conectada con la RealTime Database de Firebase y un ESP32.

Herramientas de Programación:
* Programación en Android: [Android Studio](https://developer.android.com/studio?hl=es-419)
* Programación ESP32: [Arduino](https://www.arduino.cc/en/software/)
* Diagramas Eléctricos [TinkerCAD](https://www.tinkercad.com/)
* RealTime Database de Firebase [Firebase](https://console.firebase.google.com)
* (Opcional) GitHub [GitHub](https://git-scm.com/install/)

Herramientas Electrónicas:
* ESP32
* Resistencia 220 ohm
* LED
* Protoboard
* Cables

# 1\. Configuración de la Base de Datos

1. Ingresamos a la [Consola de Firebase](https://console.firebase.google.com) y crearemos un `Nuevo Proyecto`.
2. Nos preguntará si queremos la `Asistencia de IA para tu proyecto de Firebase` - Click en `Continuar`.
3. Nos preguntará si queremos `Google Analytics para tu proyecto de Firebase` - Click en `Continuar`.
4. Si aceptaste `Google Analytics`, pedirá  `Elige o crea una cuenta de Google Analytics`. Selecciona la `Default Account for Firebase` - Click en `Crear proyecto`
5. Esperamos a que Firebase prepare nuestro proyecto. Al finalizar nos indicará `Tu proyecto de Firebase está listo` - Click en `Continuar`

## 1.1\. Elementos de Firebase (Autenticación)

El proyecto debe ser conectado de manera segura, es por eso que generaremos una autenticación controlada por `Firebase`.

1. En el menú lateral izquierdo seleccionamos `Compilación` > `Authentication`.
2. Haremos la configuración del `Autenticador`. Haz click en `Comenzar`.
3. Seleccionaremos desde la lista de `Proveedores nativos` > `Correo electrónico\contraseña`.
4. Dejaremos Habilitado en la opción `Habilitar`. Luego click en `Guardar`.
5. Agregaremos un nuevo proveedor en la opción `Agregar proveedor nuevo`.
6. Seleccionaremos desde la lista de `Proveedores nativos` > `Anónimo`.
7. Dejaremos Habilitado en la opción `Habilitar`. Luego click en `Guardar`.

De esta forma podremos controlar el acceso a la aplicación con Usuario y Contraseña, pero además dejaremos una puerta abierta para facilitar el proceso de lectura y escritura de la base de datos con el ESP32 con la autenticación anónima.

## 1.2\. Elementos de Firebase (Realtime Database)

El proyecto debe tener una base de datos, es por eso que generaremos una en tiempo real entregada por `Firebase`.

1. En el menú lateral izquierdo seleccionamos `Compilación` > `Realtime Database`.
2. Haremos la configuración de la `Base de Datos`. Haz click en `Crear una base de datos`.
3. Preguntará por `El parámetro de configuración de la ubicación determina en qué lugar se almacenarán tus datos de Realtime Database` y la `Ubicación de Realtime Database`. Seleccione `Estados Unidos (us-central1)`.
4. Cuando definas la estructura de los datos, deberás crear reglas para protegerlos. Seleccione `Comenzar en **modo de prueba**` dejará la base de datos protegida por 30 días.

# 2\. Aplicación en Android

Haremos una aplicación con al menos 2 activities. Una para ingresar y registrar los datos del usuario (`LoginActivity`) y otra para interactuar con el ESP32 (`HomeActivity`).

1. Abre Android Studio y Crea un nuevo Proyecto (`New Project`)
2. Selecciona `Empty Views Activity`.
3. Ingresa el nombre de tu aplicación, en mi caso `AppAndroidConFirebase`.
4. En `Package name` te darás cuenta que genera un nombre automáticamente, en mi caso `com.example.com.appandroidconfirebase`.
5. En `Save location` puedes dejar la ruta actual, quedará en la carpeta raíz del usuario en la carpeta `AndroidStudioProjects` y el nombre de su aplicación.
6. En `Language` selecciona `Java`.
7. En `Minimum SDK` selecciona `API 24`
8. En `Build configuration language` selecciona `Groovy DSL (build.gradle)`.
9. Finalmente, antes de hacer click en finalizar, copia el `Package name` para llevarlo a `Firebase`. Luego haz click en `Finish` y espera a que se cree el proyecto Android.

## 2.1\. Archivo de Configuración de la Aplicación

Accede a la consola de `Firebase` para obtener el archivo de configuración. Esto permite conectar la `Autenticación` y la `Base de datos` con nuestra aplicación.

1. En el menú lateral izquierdo seleccionamos `La rueda de configuración` > `Configuración del proyecto`.
2. Comenzaremos en la pestaña `General`. Hay que bajar hasta `Tus apps`.
3. Seleccionará `Android` en el ícono del robot.
4. Se abrirá una pestaña para `Agregar Firebase a tu app para Android`.
5. Ingresa el nombre del paquete de android. en mi caso `com.example.appandroidconfirebase`.
6. Luego ingresa el nombre de la App, en mi caso `App Android con Firebase` y click en `Registrar App`.
7. Nos permitirá descargar el archivo de configuración en el botón llamado `Descargar google-services.json` y click en `Siguiente`.
8. Nos indicará que debemos arrastrar el archivo a la carpeta `app` de nuestro proyecto en Android Studio. Arrastra el archivo `google-services.json` que descargaste hasta Android Studio como se muestra en la imagen de Firebase.
9. Haz click en `Refactor`
10. De las instrucciones disponibles en Firebase, haremos lo siguiente: Agrega el complemento como una dependencia a tu archivo build.gradle de nivel de proyecto: **Archivo de Gradle de nivel de raíz (nivel de proyecto) (<project>/build.gradle): **

    En `plugins` agregarás abajo de `alias(libs.plugins.android.application) apply false` el siguiente código:
    
    ```bash
    // Add the dependency for the Google services Gradle plugin
    id 'com.google.gms.google-services' version '4.4.4' apply false
    ```
11. En el otro archivo `<app>/build.gradle`, en `plugins` abajo de `alias(libs.plugins.android.application)` agregarás el siguiente código:
    ```bash
    // Add the Google services Gradle plugin
    id 'com.google.gms.google-services'
    ```
12. En este mismo archivo `<app>/build.gradle` colocarás al final antes que terminen las dependencias (debe estar en el bloque dependencias) el siguiente código:
    ```bash
    // En el bloque 'dependencies'
    implementation platform('com.google.firebase:firebase-bom:34.6.0')
    implementation 'com.google.firebase:firebase-auth'
    implementation 'com.google.firebase:firebase-database'
    ```
13. Haz click en `Sync Now`, espera un momento y estarás listo para comenzar a programar en Android.

## 2.2\. Login

### 2.2.1\. Crear Activity Login

Crearemos el `Activity Login` que incluye una capa de vista (XML) y una capa de negocio (JAVA).

1. Haz click derecho la carpeta `app` del proyecto > `new` > `Activity` > `Empty Views Activity`.
2. Ingresa el nombre `LoginActivity`. Revisa que el `Source Language` sea `Java` y haz click en `Finish`.

#### 2.2.1.1\. Vista de Login Activity

Ve a la carpeta `app` > `src` > `main` > `res` > `layout`  y selecciona `activity_login.xml`. 

Nos abrirá el GUI para ingresar los distintos controles en nuestra app.

1. En la paleta (`Palette`) ve hasta `Layouts` y toma (click con el mouse para arrastrar) el que dice `LinearLayout (vertical)`
