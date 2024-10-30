### Práctica 7: Autenticación de Usuarios y Protección de Rutas 

#### Objetivo:
Implementar un sistema de autenticación básico en la aplicación de gestión de pacientes y médicos, con redireccionamientos entre páginas de inicio de sesión, registro y la página principal. Los usuarios deberán iniciar sesión para acceder a la aplicación. Las rutas estarán protegidas mediante sesiones, y las contraseñas se guardarán en la base de datos de forma segura con hashing.

---

### Requisitos:
- Haber completado la Práctica 6.
- Tener instalados los módulos `express`, `mysql2`, `body-parser`, `bcrypt`, y `express-session`.
- Una estructura de archivos HTML separados para cada página: `index.html` (principal), `login.html` (iniciar sesión), y `registro.html` (registro de usuario).
- La base de datos debe tener una tabla `usuarios` para manejar el acceso.

---

### Paso 1: Crear la tabla `usuarios` para almacenar credenciales

En tu base de datos MySQL, crea una tabla `usuarios` para almacenar la información de autenticación de los usuarios. Incluye las siguientes columnas:

```sql
CREATE TABLE usuarios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre_usuario VARCHAR(50) UNIQUE,
  password_hash VARCHAR(255)
);
```

---

### Paso 2: Crear la página de Registro (registro.html)

1. En la carpeta `public`, crea un archivo llamado `registro.html`. Esta página permitirá que los nuevos usuarios se registren en el sistema. Agrega el siguiente código HTML:

   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <title>Registro de Usuario</title>
   </head>
   <body>
     <h2>Registro de Usuario</h2>
     <form action="/registrar" method="POST">
       <label for="nombre_usuario">Nombre de Usuario:</label>
       <input type="text" id="nombre_usuario" name="nombre_usuario" required>
       <br>
       <label for="password">Contraseña:</label>
       <input type="password" id="password" name="password" required>
       <br>
       <button type="submit">Registrar</button>
     </form>
     <p>¿Ya tienes cuenta? <a href="/login">Inicia sesión</a></p>
   </body>
   </html>
   ```

---

### Paso 3: Crear la página de Inicio de Sesión (login.html)

1. En la carpeta `public`, crea un archivo llamado `login.html`. Esta página será para que los usuarios existentes inicien sesión.

   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <title>Iniciar Sesión</title>
   </head>
   <body>
     <h2>Iniciar Sesión</h2>
     <form action="/login" method="POST">
       <label for="nombre_usuario">Nombre de Usuario:</label>
       <input type="text" id="nombre_usuario" name="nombre_usuario" required>
       <br>
       <label for="password">Contraseña:</label>
       <input type="password" id="password" name="password" required>
       <br>
       <button type="submit">Iniciar Sesión</button>
     </form>
     <p>¿No tienes cuenta? <a href="/registro">Regístrate aquí</a></p>
   </body>
   </html>
   ```

---

### Paso 4: Configurar la lógica del servidor en `server.js`

1. **Instalar y configurar las dependencias necesarias**:

   Ejecuta el siguiente comando para instalar `express-session` y `bcrypt`:

   ```bash
   npm install express-session bcrypt
   ```

2. **Agregar configuración de sesión y rutas de autenticación**:

   En `server.js`, configura las sesiones y crea las rutas para registrar usuarios, iniciar sesión y cerrar sesión.

   ```javascript
   const express = require('express');
   const session = require('express-session');
   const bcrypt = require('bcrypt');
   const mysql = require('mysql2');
   const app = express();

   // Configuración de la sesión
   app.use(session({
     secret: 'secretKey',
     resave: false,
     saveUninitialized: false,
   }));

   app.use(express.urlencoded({ extended: true }));

   // Conexión a MySQL
   const connection = mysql.createConnection({
     host: 'localhost',
     user: 'root',
     password: '',
     database: 'gestion_pacientes'
   });

   connection.connect();

   // Registro de usuario
   app.post('/registrar', async (req, res) => {
     const { nombre_usuario, password } = req.body;
     const passwordHash = await bcrypt.hash(password, 10);

     connection.query('INSERT INTO usuarios (nombre_usuario, password_hash) VALUES (?, ?)', 
       [nombre_usuario, passwordHash], (err) => {
       if (err) {
         return res.send('Error al registrar el usuario.');
       }
       res.redirect('/login');
     });
   });

   // Iniciar sesión
   app.post('/login', (req, res) => {
     const { nombre_usuario, password } = req.body;

     connection.query('SELECT * FROM usuarios WHERE nombre_usuario = ?', 
       [nombre_usuario], async (err, results) => {
       if (err || results.length === 0) {
         return res.send('Usuario no encontrado.');
       }

       const user = results[0];
       const match = await bcrypt.compare(password, user.password_hash);
       if (match) {
         req.session.userId = user.id;
         res.redirect('/');
       } else {
         res.send('Contraseña incorrecta.');
       }
     });
   });

   // Cerrar sesión
   app.get('/logout', (req, res) => {
     req.session.destroy();
     res.redirect('/login');
   });
   ```

---

### Paso 5: Proteger las rutas con autenticación de sesión

1. Agrega un middleware en `server.js` para proteger las rutas. Sólo los usuarios autenticados podrán acceder a ellas.

   ```javascript
   function requireLogin(req, res, next) {
     if (!req.session.userId) {
       return res.redirect('/login');
     }
     next();
   }

   // Ruta protegida (Página principal después de iniciar sesión)
   app.get('/', requireLogin, (req, res) => {
     res.sendFile(__dirname + '/public/index.html');
   });

   // Otras rutas protegidas
   app.get('/buscar-pacientes', requireLogin, (req, res) => {
     // lógica de búsqueda de pacientes
   });

   app.get('/ordenar-pacientes', requireLogin, (req, res) => {
     // lógica de ordenamiento de pacientes
   });

   app.post('/insertar-medico', requireLogin, (req, res) => {
     // lógica de inserción de médicos
   });
   ```

---

### Paso 6: Modificar la página principal (index.html)

1. Actualiza `index.html` en la carpeta `public` para mostrar una opción de cerrar sesión y recibir al usuario.

   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <title>Gestión de Pacientes y Médicos</title>
     <link rel="stylesheet" href="/styles.css">
   </head>
   <body>
     <h2>Bienvenido a la Gestión de Pacientes</h2>
     <a href="/logout">Cerrar Sesión</a>
     <!-- Formularios y botones de insertar/consultar médicos y pacientes -->
   </body>
   </html>
   ```

---

### Paso 7: Prueba de funcionamiento

1. **Prueba el registro de usuario**:
   - Accede a `/registro` y crea una cuenta.
   - Luego, inicia sesión con las credenciales creadas.

2. **Prueba de protección de rutas**:
   - Asegúrate de que solo los usuarios registrados puedan acceder a las rutas de `/`, `/buscar-pacientes`, `/ordenar-pacientes`, y `/insertar-medico`.

3. **Prueba de cierre de sesión**:
   - Usa el enlace de cierre de sesión para terminar la sesión y verificar que las rutas protegidas ya no sean accesibles.

---

### Paso 8: Adecuaciones
1. **Realiza las adecuaciones que consideres necesarias en funcionamiento y estética**

---
