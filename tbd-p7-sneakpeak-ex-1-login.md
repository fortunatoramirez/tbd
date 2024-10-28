### Ejemplo - Loging para Práctica 7: Implementación de Autenticación de Usuarios

---

#### Objetivo

Implementar un sistema de autenticación básica en la aplicación web, permitiendo a los usuarios registrarse y luego iniciar sesión. Esta práctica guiará a los estudiantes en el uso de formularios de usuario y contraseña, configuración de rutas de autenticación, y la integración de conceptos básicos de seguridad. Además, se mostrará cómo manejar y proteger contraseñas.

---

#### Introducción

En este ejemplo, aprenderás a añadir autenticación de usuarios a tu aplicación web, permitiendo que solo los usuarios registrados accedan a ciertas funcionalidades. Esto implica la creación de formularios HTML para el registro e inicio de sesión, así como el uso de rutas de autenticación en el servidor con Node.js y Express. 

Para esto, se abordarán conceptos como autenticación, almacenamiento seguro de contraseñas y rutas protegidas. 

---

#### Requisitos

- Haber completado la Práctica 6.
- Tener el entorno de Node.js y MySQL configurado en la carpeta gestion-pacientes.
- Las dependencias necesarias: `express`, `mysql2`, `body-parser`, y `bcrypt` (para encriptar contraseñas).

---

### Paso 1: Instalación de la Librería `bcrypt`

1. Para almacenar las contraseñas de forma segura, instalaremos `bcrypt`, que permite encriptar contraseñas.
   - Abre tu terminal en la carpeta de tu proyecto (`gestion-pacientes`) y ejecuta el siguiente comando:

     ```bash
     npm install bcrypt
     ```

   Esto instalará `bcrypt` en tu proyecto.

---

### Paso 2: Crear la Tabla `usuarios` en MySQL

1. En la terminal de MySQL, crea una nueva tabla `usuarios` para almacenar los datos de autenticación de los usuarios.

   ```sql
   CREATE TABLE usuarios (
     id INT AUTO_INCREMENT PRIMARY KEY,
     nombre_usuario VARCHAR(100) UNIQUE,
     contrasena VARCHAR(255) NOT NULL
   );
   ```

   - `nombre_usuario`: nombre de usuario único para identificar a cada usuario.
   - `contrasena`: campo en el que se almacenará la contraseña encriptada.

---

### Paso 3: Crear el Formulario de Registro e Inicio de Sesión en HTML

1. En el archivo `public/index.html`, agrega los formularios para el registro y el inicio de sesión. Estos formularios permiten al usuario ingresar su nombre de usuario y contraseña.

   ```html
   <h2>Registro de Usuario</h2>
   <form action="/registrar" method="POST">
     <label for="username">Nombre de Usuario:</label>
     <input type="text" id="username" name="nombre_usuario" required>
     <label for="password">Contraseña:</label>
     <input type="password" id="password" name="contrasena" required>
     <button type="submit">Registrar</button>
   </form>

   <h2>Iniciar Sesión</h2>
   <form action="/login" method="POST">
     <label for="username-login">Nombre de Usuario:</label>
     <input type="text" id="username-login" name="nombre_usuario" required>
     <label for="password-login">Contraseña:</label>
     <input type="password" id="password-login" name="contrasena" required>
     <button type="submit">Iniciar Sesión</button>
   </form>
   ```

   Estos formularios envían datos al servidor para crear un usuario nuevo (registro) o iniciar sesión.

---

### Paso 4: Implementar la Ruta para el Registro de Usuarios en `server.js`

1. Agrega la siguiente lógica para registrar usuarios en el archivo `server.js`.

   ```javascript
   const bcrypt = require('bcrypt'); // Asegúrate de haber instalado bcrypt

   // Ruta para registrar un nuevo usuario
   app.post('/registrar', async (req, res) => {
     const { nombre_usuario, contrasena } = req.body;
     try {
       // Encriptar la contraseña antes de guardarla
       const hashedPassword = await bcrypt.hash(contrasena, 10);
       const query = 'INSERT INTO usuarios (nombre_usuario, contrasena) VALUES (?, ?)';
       connection.query(query, [nombre_usuario, hashedPassword], (err, result) => {
         if (err) {
           return res.send('Error al registrar el usuario. Puede que el nombre de usuario ya esté en uso.');
         }
         res.send('Usuario registrado exitosamente.');
       });
     } catch (error) {
       res.send('Error en la encriptación de la contraseña.');
     }
   });
   ```

   - Aquí se utiliza `bcrypt` para encriptar la contraseña antes de guardarla en la base de datos.

---

### Paso 5: Implementar la Ruta de Inicio de Sesión en `server.js`

1. Agrega la ruta para iniciar sesión y verificar la contraseña ingresada por el usuario.

   ```javascript
   // Ruta para iniciar sesión
   app.post('/login', (req, res) => {
     const { nombre_usuario, contrasena } = req.body;
     const query = 'SELECT * FROM usuarios WHERE nombre_usuario = ?';
     connection.query(query, [nombre_usuario], async (err, results) => {
       if (err) return res.send('Error al iniciar sesión.');
       if (results.length === 0) return res.send('Usuario no encontrado.');

       const user = results[0];
       const match = await bcrypt.compare(contrasena, user.contrasena);

       if (match) {
         res.send('Inicio de sesión exitoso.');
       } else {
         res.send('Contraseña incorrecta.');
       }
     });
   });
   ```

   - Esta lógica busca el nombre de usuario en la base de datos y compara la contraseña ingresada con la almacenada.

---

### Paso 6: Crear Rutas Protegidas

Para proteger ciertas rutas y permitir el acceso solo a usuarios autenticados, deberemos definir una función de middleware simple para la autenticación en `server.js`.

1. Crea la función `isAuthenticated`.

   ```javascript
   const isAuthenticated = (req, res, next) => {
     if (req.session && req.session.userId) {
       return next();
     } else {
       res.send('Por favor, inicie sesión para acceder a esta página.');
     }
   };
   ```

2. Para proteger una ruta, utiliza el middleware `isAuthenticated`:

   ```javascript
   app.get('/ruta-protegida', isAuthenticated, (req, res) => {
     res.send('Bienvenido a la página protegida.');
   });
   ```

---

### Paso 7: Integrar la Autenticación en la Interfaz

- **Registro**: Completa el formulario de registro para crear un usuario nuevo.
- **Inicio de Sesión**: Luego, realiza el inicio de sesión para acceder a rutas protegidas.

### Paso 8: Pruebas Finales y Validación

1. **Ejecuta el servidor**: Ejecuta el servidor en la terminal con el comando:

   ```bash
   node server.js
   ```

2. **Prueba el registro e inicio de sesión**:
   - Ingresa distintos usuarios y contraseñas, y verifica que solo los usuarios autenticados puedan acceder a las rutas protegidas.
