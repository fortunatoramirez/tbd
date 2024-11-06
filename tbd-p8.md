### Práctica 8: Gestión de Roles de Usuario, Control de Acceso y Navegación

#### Objetivo
Implementar:
1. Tipos de usuario con distintos permisos y accesos.
2. Un sistema de registro protegido por un código especial de acceso proporcionado por el administrador.
3. Una barra de navegación para facilitar la navegación según el tipo de usuario.

---

### Requisitos
- Haber completado la Práctica 7.

---

### Estructura de Archivos

La estructura del proyecto será la siguiente:
```
proyecto/
├── server.js
├── public/
│   ├── index.html
│   ├── login.html
│   ├── registro.html
│   ├── styles.css
│   └── navbar.html  // Nueva página para la barra de navegación
├── package.json
└── node_modules/
```

---

### Paso 1: Modificar la Base de Datos para los Roles de Usuario

1. **Tabla de Usuarios**: Agrega una columna `tipo_usuario` en la tabla `usuarios`, que defina el rol. Ejemplo:
   - `admin`: Acceso completo (puede ver, editar, eliminar, y gestionar otros usuarios).
   - `medico`: Acceso para consultar y actualizar registros de pacientes.
   - `paciente`: Acceso limitado para ver su propia información.

2. **Tabla de Códigos de Registro**: Crea una tabla para almacenar códigos de acceso con sus permisos:
   ```sql
   CREATE TABLE codigos_acceso (
       codigo VARCHAR(10) PRIMARY KEY,
       tipo_usuario VARCHAR(20) NOT NULL
   );
   ```
   - Los códigos pueden ser únicos para cada tipo de usuario y se requerirán en el formulario de registro.

---

### Paso 2: Configurar Rutas con Control de Acceso en `server.js`

1. **Agregar Middleware para Autorización de Roles**: Define una función `requireRole` para validar que el usuario tiene el rol necesario para acceder a ciertas rutas.

   ```javascript
   function requireRole(role) {
       return (req, res, next) => {
           if (req.session.user && req.session.user.tipo_usuario === role) {
               next();
           } else {
               res.status(403).send('Acceso denegado');
           }
       };
   }
   ```

2. **Rutas Protegidas por Rol**: Configura rutas específicas para cada tipo de usuario. Por ejemplo:

   ```javascript
   // Ruta para que solo admin pueda ver todos los usuarios
   app.get('/ver-usuarios', requireLogin, requireRole('admin'), (req, res) => {
       const query = 'SELECT * FROM usuarios';
       connection.query(query, (err, results) => {
           if (err) return res.send('Error al obtener usuarios');
           res.send(results);
       });
   });

   // Ruta para que los médicos puedan ver pacientes
   app.get('/ver-pacientes', requireLogin, requireRole('medico'), (req, res) => {
       // Lógica de consulta para ver pacientes
   });
   ```

---

### Paso 3: Configurar Registro con Código de Acceso

1. **Modificación en `registro.html`**: Añade un campo para ingresar el código de acceso en el formulario de registro.

   ```html
   <form action="/registro" method="POST">
       <label for="username">Nombre de usuario:</label>
       <input type="text" id="username" name="username" required>
       <label for="password">Contraseña:</label>
       <input type="password" id="password" name="password" required>
       <label for="codigo_acceso">Código de acceso:</label>
       <input type="text" id="codigo_acceso" name="codigo_acceso" required>
       <button type="submit">Registrarse</button>
   </form>
   ```

2. **Validación de Código en el Registro**: En `server.js`, agrega la lógica para verificar el código de acceso.

   ```javascript
   app.post('/registro', (req, res) => {
       const { username, password, codigo_acceso } = req.body;

       const query = 'SELECT tipo_usuario FROM codigos_acceso WHERE codigo = ?';
       connection.query(query, [codigo_acceso], (err, results) => {
           if (err || results.length === 0) {
               return res.send('Código de acceso inválido');
           }

           const tipo_usuario = results[0].tipo_usuario;
           const hashedPassword = bcrypt.hashSync(password, 10);

           const insertUser = 'INSERT INTO usuarios (username, password, tipo_usuario) VALUES (?, ?, ?)';
           connection.query(insertUser, [username, hashedPassword, tipo_usuario], (err) => {
               if (err) return res.send('Error al registrar usuario');
               res.redirect('/login.html');
           });
       });
   });
   ```

---

### Paso 4: Modificación en el Código de Inicio de Sesión

En el código de la ruta de inicio de sesión (`/login`), después de validar que la contraseña es correcta, agrega el `tipo_usuario` a `req.session.user`. Así, cada vez que un usuario inicie sesión, la información de su tipo de usuario se guardará en la sesión y estará disponible para todas las rutas protegidas.

Por ejemplo:

```javascript
app.post('/login', (req, res) => {
    const { username, password } = req.body;

    // Consulta para obtener el usuario y su tipo
    const query = 'SELECT * FROM usuarios WHERE username = ?';
    connection.query(query, [username], (err, results) => {
        if (err) {
            return res.send('Error al obtener el usuario');
        }

        if (results.length === 0) {
            return res.send('Usuario no encontrado');
        }

        const user = results[0];

        // Verificar la contraseña
        const isPasswordValid = bcrypt.compareSync(password, user.password);
        if (!isPasswordValid) {
            return res.send('Contraseña incorrecta');
        }

        // Almacenar la información del usuario en la sesión
        req.session.user = {
            id: user.id,
            username: user.username,
            tipo_usuario: user.tipo_usuario // Aquí se establece el tipo de usuario en la sesión
        };

        // Redirigir al usuario a la página principal
        res.redirect('/');
    });
});
```

### Paso 5: Crear y Configurar una Barra de Navegación Dinámica (`navbar.html`) con Petición AJAX desde el Navegador

En este paso, vamos a crear una barra de navegación dinámica que se adapte al tipo de usuario. La barra de navegación se construirá en un archivo aparte llamado `navbar.html`, y el contenido cambiará según los permisos del usuario (por ejemplo, admin, médico, paciente). Para obtener el tipo de usuario, haremos una solicitud AJAX desde el navegador hacia el servidor, de forma que podamos controlar y mostrar las opciones de navegación apropiadas en la página principal.

1. **Crear el archivo de la barra de navegación `navbar.html`**

   En la carpeta `public`, crea un archivo llamado `navbar.html`. Este archivo contendrá los enlaces de navegación básicos y será el mismo para todos los usuarios. Los enlaces adicionales se añadirán dinámicamente según el tipo de usuario, que será obtenido mediante AJAX desde el servidor.

   **Contenido de `navbar.html`:**

   ```html
   <!-- Barra de Navegación Básica -->
   <nav>
       <ul id="menu">
           <li><a href="/">Inicio</a></li>
       </ul>
   </nav>
   ```

2. **Configurar una ruta en el servidor para enviar el tipo de usuario**

   En `server.js`, crea una nueva ruta que envíe el tipo de usuario (admin, médico, o paciente) al cliente en formato JSON. Esta ruta será solicitada desde el navegador para determinar qué opciones de navegación mostrar.

   ```javascript
   // Ruta para obtener el tipo de usuario actual
   app.get('/tipo-usuario', requireLogin, (req, res) => {
       res.json({ tipo_usuario: req.session.user.tipo_usuario });
   });
   ```

   Esta ruta se protegerá con `requireLogin` para asegurar que solo los usuarios autenticados puedan acceder a la información del tipo de usuario.

3. **Incorporar `navbar.html` en `index.html`**

   Ahora, en `index.html`, incluimos `navbar.html` utilizando JavaScript. Esto permite cargar el menú básico y posteriormente hacer una solicitud AJAX para actualizar el menú según el tipo de usuario.

   **Contenido de `index.html`:**

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Página Principal</title>
       <link rel="stylesheet" href="styles.css">
   </head>
   <body>
       <!-- Incluir barra de navegación desde navbar.html -->
       <div id="navbar"></div>

       <script>
           // Insertar el contenido de navbar.html en el elemento con id "navbar"
           fetch('/navbar.html')
               .then(response => response.text())
               .then(data => {
                   document.getElementById('navbar').innerHTML = data;
               })
               .catch(error => console.error('Error cargando el navbar:', error));
       </script>
   </body>
   </html>
   ```

   En este paso, estamos solicitando `navbar.html` y añadiéndolo al elemento `div` con `id="navbar"`.

4. **Hacer una Solicitud AJAX para Obtener el Tipo de Usuario y Actualizar el Menú de Navegación**

   Después de cargar `navbar.html`, se necesita realizar una segunda solicitud para obtener el tipo de usuario desde el servidor y actualizar el menú de navegación con las opciones apropiadas.

   En el mismo script de `index.html`, añade el siguiente código:

   ```html
   <script>
       // Solicitar el tipo de usuario y ajustar el menú en función de este
       fetch('/tipo-usuario')
           .then(response => response.json())
           .then(data => {
               const menu = document.getElementById('menu');
               const tipoUsuario = data.tipo_usuario;

               // Agregar opciones de menú según el tipo de usuario
               if (tipoUsuario === 'admin') {
                   menu.innerHTML += '<li><a href="/ver-usuarios">Ver Usuarios</a></li>';
                   menu.innerHTML += '<li><a href="/gestionar-registros">Gestionar Registros</a></li>';
               } else if (tipoUsuario === 'medico') {
                   menu.innerHTML += '<li><a href="/ver-pacientes">Ver Pacientes</a></li>';
                   menu.innerHTML += '<li><a href="/editar-pacientes">Editar Pacientes</a></li>';
               } else if (tipoUsuario === 'paciente') {
                   menu.innerHTML += '<li><a href="/ver-mis-datos">Mis Datos</a></li>';
               }

               // Opción de cerrar sesión para todos los tipos de usuario
               menu.innerHTML += '<li><a href="/logout">Cerrar Sesión</a></li>';
           })
           .catch(error => console.error('Error obteniendo el tipo de usuario:', error));
   </script>
   ```

   Con este código, se realiza una solicitud `fetch` a la ruta `/tipo-usuario` para obtener el tipo de usuario y, según su valor, se añaden las opciones correspondientes al menú.

5. **Estilizar el Menú en `styles.css`**

   Puedes añadir estilos a la barra de navegación en `styles.css` para darle un diseño uniforme y atractivo. A continuación se presenta un ejemplo básico:

   ```css
   /* Estilo para el contenedor de la barra de navegación */
   nav {
       background-color: #333;
       color: white;
       padding: 10px;
   }

   /* Estilo para los enlaces del menú */
   nav ul {
       list-style-type: none;
       margin: 0;
       padding: 0;
       display: flex;
   }

   nav ul li {
       margin-right: 20px;
   }

   nav ul li a {
       color: white;
       text-decoration: none;
       padding: 5px 10px;
   }

   nav ul li a:hover {
       background-color: #555;
       border-radius: 4px;
   }
   ```

   Este estilo añadirá un fondo oscuro a la barra de navegación y resaltará las opciones cuando se desplace el cursor sobre ellas.

---

### Paso 6: Prueba de Funcionalidad y Control de Acceso

1. **Registro de Usuarios con Código de Acceso**: Registra usuarios utilizando diferentes códigos de acceso para cada tipo de usuario.

2. **Prueba de Control de Acceso**:
   - Intenta acceder a rutas protegidas con diferentes usuarios y roles.
   - Verifica que el acceso es permitido o denegado correctamente según el tipo de usuario.
   - Agrega las rutas y la funcionalidad de cada ruta de acuerdo a lo que consideres necesario además de la que se tienen de la práctica anterior: ver, ordenar, eliminar, etc., de acuerdo a cada usuario.

3. **Barra de Navegación Condicional**: Comprueba que solo se muestran los enlaces correspondientes al tipo de usuario que ha iniciado sesión.
4. **Realiza las modificaciones que consideres adecuadas de acuerdo a funcionalidad y estética.

---

