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

### Paso 4: Crear una Barra de Navegación en `navbar.html`

1. **Crear una Barra de Navegación**: Añade en `navbar.html` la barra de navegación con enlaces condicionales según el tipo de usuario.

   ```html
   <nav>
       <ul>
           <li><a href="/">Inicio</a></li>
           <!-- Opciones para Admin -->
           <% if (tipo_usuario === 'admin') { %>
               <li><a href="/ver-usuarios">Ver Usuarios</a></li>
               <li><a href="/gestionar-registros">Gestionar Registros</a></li>
           <% } %>
           <!-- Opciones para Médico -->
           <% if (tipo_usuario === 'medico') { %>
               <li><a href="/ver-pacientes">Ver Pacientes</a></li>
               <li><a href="/editar-pacientes">Editar Pacientes</a></li>
           <% } %>
           <!-- Opciones para Paciente -->
           <% if (tipo_usuario === 'paciente') { %>
               <li><a href="/ver-mis-datos">Mis Datos</a></li>
           <% } %>
           <li><a href="/logout">Cerrar Sesión</a></li>
       </ul>
   </nav>
   ```

2. **Servir la Barra en `index.html`**: Utiliza una función de `include` para integrar la barra en `index.html`. 

   ```html
   <!-- index.html -->
   <header>
       <!-- Incluir barra de navegación -->
       <% include navbar.html %>
   </header>
   ```

---

### Paso 5: Prueba de Funcionalidad y Control de Acceso

1. **Registro de Usuarios con Código de Acceso**: Registra usuarios utilizando diferentes códigos de acceso para cada tipo de usuario.

2. **Prueba de Control de Acceso**:
   - Intenta acceder a rutas protegidas con diferentes usuarios y roles.
   - Verifica que el acceso es permitido o denegado correctamente según el tipo de usuario.

3. **Barra de Navegación Condicional**: Comprueba que solo se muestran los enlaces correspondientes al tipo de usuario que ha iniciado sesión.

---

