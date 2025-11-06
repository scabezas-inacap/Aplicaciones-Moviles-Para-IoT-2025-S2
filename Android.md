# Programación en Android

Herramientas:
* [Android Studio](https://developer.android.com/studio?hl=es-419)
* [Android Cloud](https://developer.android.com/studio/preview/android-studio-cloud?hl=es-419)
* [Studio Firebase](https://studio.firebase.google.com)

## Módulo 3: Configuración de Firebase

**Creación en Consola Firebase**:

1. Ir a https://console.firebase.google.com.
2. Crear un nuevo proyecto (ej: "`MiAppFirebase`").
3. Dentro del proyecto, añadir una app de Android.
4. IMPORTANTE: El "Package Name" de Android debe coincidir exactamente con el del proyecto de Android Studio (`ej: com.example.miappfirebase`).
5. Descargar el archivo **google-services.json**.

**Integración en Android Studio**:

1. Cambiar la vista de "Android" a "Project" en el explorador de archivos (panel izquierdo).
2. Arrastrar **google-services.json** dentro de la carpeta app/.
3. Verificar los **build.gradle** (Android Studio a veces lo hace automático con el wizard, pero es bueno saber hacerlo manual).
4. Añadir las dependencias de Firebase (Auth y RTDB) en build.gradle (Module :app):
```
// En el bloque 'dependencies'
implementation platform('com.google.firebase:firebase-bom:33.1.0') // Importa el Bill of Materials
implementation 'com.google.firebase:firebase-auth'
implementation 'com.google.firebase:firebase-database'
```
5. Sincronizar Gradle ("Sync Now").

**Habilitar Servicios en Consola**:
1. En la consola de Firebase, ir a "Authentication" -> "Sign-in method" y habilitar "Email/Password".

2. Ir a "Realtime Database" -> "Create Database" (empezar en modo de prueba o test mode para permitir lecturas/escrituras).

## Módulo 4: UI y Autenticación

Aquí creamos la lógica de Login y Registro. Crearemos una nueva Activity para el Login.

1. **Crear Activities y XML**:

* Activity de Login: Click derecho en el paquete (ej: com.example.miappfirebase) -> New -> Activity -> Empty Views Activity. Nombrarla LoginActivity.

* Activity Principal (Home): Renombrar MainActivity a HomeActivity (Refactor -> Rename).

* Actualizar AndroidManifest.xml: Mover el <intent-filter> de HomeActivity a LoginActivity. Esto hace que LoginActivity sea la pantalla de inicio.

```
<activity android:name=".LoginActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity android:name=".HomeActivity" />
```

2. **Diseño XML (activity_login.xml)**:

* Añadir 2 EditText (para email y password).

* Añadir 2 Button (para "Registrar" y "Acceder").

* (Usar un LinearLayout vertical es lo más rápido para la clase).

3. Código Java (LoginActivity.java): Este es el núcleo de la clase.

```
// LoginActivity.java
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.auth.AuthResult;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;

public class LoginActivity extends AppCompatActivity {

    private EditText etEmail, etPassword;
    private Button btnRegistrar, btnAcceder;
    private FirebaseAuth mAuth; // Variable de Firebase Auth

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        // Inicializar Firebase Auth
        mAuth = FirebaseAuth.getInstance();

        // Referenciar Vistas
        etEmail = findViewById(R.id.etEmail); // Asegúrate que este ID exista en tu XML
        etPassword = findViewById(R.id.etPassword); // Asegúrate que este ID exista en tu XML
        btnRegistrar = findViewById(R.id.btnRegistrar); // Asegúrate que este ID exista en tu XML
        btnAcceder = findViewById(R.id.btnAcceder); // Asegúrate que este ID exista en tu XML

        // Listener para Registrar
        btnRegistrar.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String email = etEmail.getText().toString().trim();
                String password = etPassword.getText().toString().trim();
                registrarUsuario(email, password);
            }
        });

        // Listener para Acceder
        btnAcceder.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String email = etEmail.getText().toString().trim();
                String password = etPassword.getText().toString().trim();
                iniciarSesion(email, password);
            }
        });
    }

    @Override
    protected void onStart() {
        super.onStart();
        // Comprobar si el usuario ya inició sesión
        FirebaseUser currentUser = mAuth.getCurrentUser();
        if (currentUser != null) {
            // Si ya hay sesión, vamos directo al Home
            irAHome();
        }
    }

    private void registrarUsuario(String email, String password) {
        if (email.isEmpty() || password.isEmpty()) {
            Toast.makeText(this, "Email y Password no pueden estar vacíos", Toast.LENGTH_SHORT).show();
            return;
        }

        mAuth.createUserWithEmailAndPassword(email, password)
            .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                @Override
                public void onComplete(@NonNull Task<AuthResult> task) {
                    if (task.isSuccessful()) {
                        // Registro exitoso
                        Toast.makeText(LoginActivity.this, "Registro Exitoso.", Toast.LENGTH_SHORT).show();
                        irAHome();
                    } else {
                        // Falla en el registro
                        Toast.makeText(LoginActivity.this, "Fallo el registro: " + task.getException().getMessage(), Toast.LENGTH_LONG).show();
                    }
                }
            });
    }

    private void iniciarSesion(String email, String password) {
        if (email.isEmpty() || password.isEmpty()) {
            Toast.makeText(this, "Email y Password no pueden estar vacíos", Toast.LENGTH_SHORT).show();
            return;
        }

        mAuth.signInWithEmailAndPassword(email, password)
            .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                @Override
                public void onComplete(@NonNull Task<AuthResult> task) {
                    if (task.isSuccessful()) {
                        // Login exitoso
                        Toast.makeText(LoginActivity.this, "Login Exitoso.", Toast.LENGTH_SHORT).show();
                        irAHome();
                    } else {
                        // Falla en el login
                        Toast.makeText(LoginActivity.this, "Fallo el login: " + task.getException().getMessage(), Toast.LENGTH_LONG).show();
                    }
                }
            });
    }

    // Método para navegar a la siguiente pantalla
    private void irAHome() {
        Intent intent = new Intent(LoginActivity.this, HomeActivity.class);
        // Flags para que no pueda volver al Login presionando "atrás"
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        startActivity(intent);
    }
}
```

## Módulo 5: Realtime Database (Guardar datos)

Ahora, en HomeActivity, vamos a guardar la fecha de nacimiento.

1. **Diseño XML (activity_home.xml)**:

* Renombrar activity_main.xml a activity_home.xml (Refactor -> Rename).

* Añadir un EditText (ej: etFechaNacimiento).

* Añadir un Button (ej: btnGuardarFecha).

* Añadir un Button (ej: btnCerrarSesion).

2. **Código Java (HomeActivity.java)**: Aquí guardamos el dato asociado al UID (User ID) del usuario autenticado.

```
// HomeActivity.java
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;

public class HomeActivity extends AppCompatActivity {

    private EditText etFechaNacimiento;
    private Button btnGuardarFecha, btnCerrarSesion;

    private FirebaseAuth mAuth;
    private DatabaseReference mDatabase; // Variable de Realtime Database

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_home);

        etFechaNacimiento = findViewById(R.id.etFechaNacimiento); // Asignar ID
        btnGuardarFecha = findViewById(R.id.btnGuardarFecha); // Asignar ID
        btnCerrarSesion = findViewById(R.id.btnCerrarSesion); // Asignar ID

        mAuth = FirebaseAuth.getInstance();
        // Apuntamos a la raíz de la BD
        mDatabase = FirebaseDatabase.getInstance().getReference();

        FirebaseUser currentUser = mAuth.getCurrentUser();
        if (currentUser == null) {
            // Si no hay usuario, volver al Login
            irALogin();
            return;
        }

        // (Opcional) Cargar el dato si ya existe
        cargarDatosUsuario(currentUser.getUid());

        btnGuardarFecha.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String fechaNac = etFechaNacimiento.getText().toString().trim();
                if (!fechaNac.isEmpty()) {
                    guardarFechaNacimiento(currentUser.getUid(), fechaNac);
                }
            }
        });

        btnCerrarSesion.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mAuth.signOut();
                irALogin();
            }
        });
    }

    private void guardarFechaNacimiento(String uid, String fechaNac) {
        // Creamos una estructura: "usuarios" -> "UID_del_usuario" -> "fechaNacimiento"
        mDatabase.child("usuarios").child(uid).child("fechaNacimiento").setValue(fechaNac)
            .addOnCompleteListener(new OnCompleteListener<Void>() {
                @Override
                public void onComplete(@NonNull Task<Void> task) {
                    if (task.isSuccessful()) {
                        Toast.makeText(HomeActivity.this, "Fecha guardada", Toast.LENGTH_SHORT).show();
                    } else {
                        Toast.makeText(HomeActivity.this, "Error al guardar", Toast.LENGTH_SHORT).show();
                    }
                }
            });
    }

    // (Actividad Opcional si hay tiempo) Leer el dato
    private void cargarDatosUsuario(String uid) {
        mDatabase.child("usuarios").child(uid).child("fechaNacimiento")
            .addListenerForSingleValueEvent(new ValueEventListener() {
                @Override
                public void onDataChange(@NonNull DataSnapshot snapshot) {
                    if (snapshot.exists()) {
                        String fecha = snapshot.getValue(String.class);
                        etFechaNacimiento.setText(fecha);
                    }
                }

                @Override
                public void onCancelled(@NonNull DatabaseError error) {
                    Toast.makeText(HomeActivity.this, "Error al cargar datos", Toast.LENGTH_SHORT).show();
                }
            });
    }

    private void irALogin() {
        Intent intent = new Intent(HomeActivity.this, LoginActivity.class);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        startActivity(intent);
    }
}
```
