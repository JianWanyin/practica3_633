# Prácticas de Codificación Segura para Mitigar Vulnerabilidades Comunes

## 1. Validación y Sanitización de Entradas

### Descripción de los pasos:
La validación y la sanitización son prácticas esenciales para proteger las aplicaciones web contra ataques maliciosos como inyecciones de código y **Cross-Site Scripting (XSS)**. 

- **Validación**:
  - Verifica que los datos proporcionados por el usuario sean correctos y estén en el formato esperado (por ejemplo, correos electrónicos válidos, fechas correctas, etc.).
  - Utiliza expresiones regulares para asegurarse de que los datos sean válidos y no contengan caracteres peligrosos.
  - Asegúrate de que las entradas no contengan valores maliciosos que puedan comprometer la seguridad de la aplicación.

- **Sanitización**:
  - Limpia los datos proporcionados para evitar que contengan caracteres que puedan ser interpretados como código malicioso por el navegador o el servidor.
  - Codifica caracteres como `<`, `>`, `&` y `"` para prevenir ataques XSS.

### Ejemplo en JavaScript:

```javascript
// Validación de correo electrónico
function validarEmail(email) {
    const regex = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    return regex.test(email);
}

// Sanitización de texto para prevenir ataques XSS
function sanitizarTexto(texto) {
    const div = document.createElement('div');
    div.appendChild(document.createTextNode(texto));
    return div.innerHTML; // Esto codifica los caracteres especiales
}

// Ejemplo de uso
const email = 'usuario@dominio.com';
const textoUsuario = '<script>alert("XSS")</script>';

if (validarEmail(email)) {
    console.log('Correo electrónico válido');
}

const textoSeguro = sanitizarTexto(textoUsuario);
console.log(textoSeguro); // Salida segura para mostrar en una página web

## 2. Control de Acceso y Autenticación Segura

### Beneficios de la Autenticación Multifactor (MFA) y los Tokens JWT:

- **Autenticación Multifactor (MFA)**:
  - La **MFA** añade una capa extra de seguridad al proceso de autenticación, solicitando más de un factor de autenticación. Estos factores pueden ser:
    1. **Algo que sabe el usuario** (contraseña, PIN).
    2. **Algo que tiene el usuario** (un teléfono móvil para recibir un código).
    3. **Algo que es el usuario** (biometría, como huellas dactilares).
  - **Ventajas**:
    - **Mayor seguridad**: Aún si un atacante obtiene una contraseña, necesitará el segundo factor (como un código de un dispositivo).
    - **Protección contra ataques de phishing**: Si un atacante obtiene la contraseña, no podrá acceder sin el segundo factor.

- **Autenticación basada en Tokens JWT**:
  - **JSON Web Tokens (JWT)** son tokens que permiten transmitir información de forma segura entre dos partes (por ejemplo, cliente y servidor) en formato JSON.
  - **Ventajas**:
    - Los JWT pueden almacenar información sobre el usuario (como ID o roles) y expirar en un tiempo determinado, evitando la necesidad de sesiones del lado del servidor.
    - El token es firmado y, por lo tanto, es confiable; no puede ser modificado sin invalidar la firma.
    - Es comúnmente usado para **autenticación en aplicaciones web y móviles**.

### Ejemplo de creación de un JWT en Node.js con la librería `jsonwebtoken`:

```javascript
const jwt = require('jsonwebtoken');

// Datos del usuario
const usuario = { id: 1, nombre: 'Juan' };

// Crear JWT
const token = jwt.sign(usuario, 'claveSecreta', { expiresIn: '1h' });

console.log('Token JWT:', token);


## 3. Gestión de Sesiones y Cookies

### Atributos de seguridad HttpOnly y Secure en Cookies:

- **HttpOnly**:
  - **Definición**: Este atributo indica que la cookie no debe ser accesible a través de JavaScript. Esto protege la cookie de ataques XSS, ya que incluso si el atacante inyecta código malicioso, no podrá acceder a las cookies.
  - **Beneficio**: Previene que las cookies sean robadas o manipuladas mediante scripts maliciosos en el navegador del cliente.

- **Secure**:
  - **Definición**: Las cookies con el atributo `Secure` solo se envían a través de conexiones HTTPS, lo que garantiza que las cookies no se transmitan a través de canales inseguros (HTTP).
  - **Beneficio**: Protege las cookies de ataques de **Man-in-the-Middle (MITM)**, que podrían ocurrir cuando los datos se envían sin cifrado.

### Implementación de configuración de sesión en Express.js:

```javascript
const express = require('express');
const app = express();

// Configuración de cookies con atributos de seguridad
app.use((req, res, next) => {
    res.cookie('sessionId', 'valorDeSesion', { 
        httpOnly: true,  // Previene el acceso JavaScript
        secure: true,    // Solo se transmite a través de HTTPS
        sameSite: 'Strict' // Evita el envío de cookies en solicitudes de otros sitios
    });
    next();
});

app.get('/', (req, res) => {
    res.send('Sesión segura configurada.');
});

app.listen(3000, () => {
    console.log('Servidor en ejecución');
});


## 4. Protección de Datos Sensibles

### Diferencia entre Hashing y Cifrado:

- **Hashing**:
  - **Unidireccional**: El hashing convierte los datos en un valor de longitud fija que es irreconocible desde el valor original. Este proceso no es reversible, lo que significa que no se puede obtener el valor original a partir del hash.
  - **Uso común**: Se utiliza principalmente para almacenar contraseñas de manera segura. Se almacena el hash de la contraseña y, en lugar de verificar la contraseña directamente, se compara el hash de la contraseña ingresada con el almacenado.
  - **Ejemplo**: Funciones como `password_hash()` y `password_verify()` en PHP.
  - **Ventaja**: Aunque un atacante obtenga el hash, no puede obtener la contraseña original sin realizar un ataque de fuerza bruta, lo cual es costoso.

- **Cifrado**:
  - **Bidireccional**: El cifrado convierte los datos en un formato ilegible que solo puede ser revertido con la clave adecuada. El cifrado es reversible, lo que significa que los datos pueden ser descifrados de nuevo a su forma original.
  - **Uso común**: Se utiliza cuando se necesita recuperar los datos originales, como al cifrar comunicaciones o almacenar información confidencial que debe ser descifrada cuando se necesite.
  - **Ejemplo**: AES (Advanced Encryption Standard), RSA.
  - **Ventaja**: Es útil cuando se requiere que los datos se descifren en algún momento para su uso.

### Ejemplo de manejo de contraseñas seguras en PHP con `password_hash` y `password_verify`:

```php
<?php
// Hash de la contraseña
$password = 'miContraseñaSecreta';
$hash = password_hash($password, PASSWORD_DEFAULT);

// Verificación de la contraseña
if (password_verify($password, $hash)) {
    echo 'Contraseña válida';
} else {
    echo 'Contraseña inválida';
}
?>
