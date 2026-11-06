# Práctica 8. Roles de usuario, permisos y navegación del sistema

## Objetivo

Ampliar el sistema de gestión de pacientes y médicos para que cada cuenta tenga un rol: **administrador, médico o paciente**. Incorporar códigos de registro, controlar las operaciones desde el servidor y mostrar un menú acorde con el usuario que inició sesión.

**Punto de partida:** tu proyecto de la práctica 7, terminado hasta el **paso 9: cierre de sesión**. En esta actividad no se realizan pruebas de inyección SQL ni se requiere haber completado los pasos posteriores de esa práctica.

Trabaja con información ficticia. Esta práctica se ejecuta localmente.

## 1. Copiar el proyecto que ya funciona

1. Detén Node.js con **Ctrl+C**.
2. Copia la carpeta de tu proyecto de la práctica 7 y llama a la copia `gestion-pacientes-p8`.
3. Abre **la copia** en VS Code. Conserva `server.js`, `package.json`, `package-lock.json` y toda la carpeta `public`.
4. Puedes omitir `node_modules` al copiar. En la terminal de la carpeta nueva ejecuta `npm install` para instalar las dependencias que ya están en `package.json`. No necesitas ejecutar nuevamente `npm init`.
5. Ejecuta `node server.js` y entra en `http://localhost:3000`. Comprueba que puedes iniciar sesión, consultar pacientes y cerrar sesión.
6. Detén el servidor antes de continuar.

**La copia de la carpeta no copia MySQL.** Usaremos la misma base `gestion_pacientes` y conservaremos sus registros. Los cambios de tablas también serán visibles desde el proyecto anterior. Exporta un respaldo de esa base desde MySQL Workbench antes de modificarla. No vuelvas a crear las tablas `pacientes`, `medicos` o `usuarios`.

### Qué conservar y qué modificar

| Elemento | Trabajo de esta práctica |
| --- | --- |
| Conexión `connection`, Express, sesiones y bcrypt | Conservar |
| `public/login.html` y cierre de sesión `POST /logout` | Conservar |
| Tabla `usuarios` | Agregar rol y vínculo con un paciente |
| Tabla `codigos_acceso` | Crear |
| `public/registro.html` | Agregar un campo al formulario existente |
| `POST /registrar` | Sustituir únicamente esta ruta por su versión ampliada |
| `POST /login` | Cambiar su consulta y agregar dos datos a la sesión |
| Rutas de pacientes y médicos | Agregar comprobaciones de rol conservando su contenido |
| `public/index.html` | Agrupar los controles actuales y agregar el menú |
| `public/navbar.html`, `public/navegacion.js` | Crear |
| `public/styles.css` | Agregar estilos al final |

Los bloques de código de esta guía son **fragmentos para integrar**. No reemplaces todo `server.js` ni toda tu página principal. Debe seguir existiendo una sola conexión, una sola definición de cada ruta y un solo `app.listen(...)`.

## 2. Entender los permisos antes de programarlos

**Autenticación:** comprobar quién inició sesión. Ya la realizas en la práctica 7.

**Autorización:** comprobar qué puede hacer esa persona. La agregarás ahora.

Usaremos estos permisos concretos, correspondientes a funciones existentes o que desarrollarás aquí:

| Operación | `admin` | `medico` | `paciente` |
| --- | :---: | :---: | :---: |
| Abrir inicio y cerrar sesión | Sí | Sí | Sí |
| Registrar pacientes | Sí | Sí | No |
| Consultar, buscar y ordenar todos los pacientes | Sí | Sí | No |
| Registrar médicos | Sí | No | No |
| Consultar la lista de cuentas y sus roles | Sí | No | No |
| Consultar el registro de paciente vinculado a su cuenta | No | No | Sí |

En esta práctica no se agregan edición, eliminación ni un panel de administración de cuentas. El administrador tendrá acceso a todas las funciones de gestión implementadas; la vista personal es exclusiva del rol paciente.

**Ocultar un botón mejora la interfaz, pero los permisos se comprueban en el servidor.** Cada ruta debe rechazar las operaciones que el rol no tenga permitidas.

## 3. Ampliar la base de datos sin perder registros

### 3.1 Revisar la tabla actual

En MySQL ejecuta:

```sql
USE gestion_pacientes;
DESCRIBE usuarios;
DESCRIBE pacientes;
```

En `usuarios` deben existir `id`, `nombre_usuario` y `password_hash`, como en la práctica 7. Conserva estos nombres durante toda la actividad.

### 3.2 Agregar el rol y la relación con pacientes

Ejecuta **una sola vez**, si todavía no existen estas columnas:

```sql
ALTER TABLE usuarios
  ADD COLUMN tipo_usuario ENUM('admin', 'medico', 'paciente')
    NOT NULL DEFAULT 'paciente',
  ADD COLUMN paciente_id INT NULL,
  ADD CONSTRAINT uq_usuario_paciente UNIQUE (paciente_id),
  ADD CONSTRAINT fk_usuario_paciente
    FOREIGN KEY (paciente_id) REFERENCES pacientes(id);
```

- `tipo_usuario` indica los permisos de la cuenta. Las cuentas que ya existían quedan inicialmente como `paciente`.
- `paciente_id` identifica el registro clínico asociado a una cuenta de paciente. Puede ser `NULL` mientras no se asigne uno.
- La llave foránea exige que el paciente exista.
- `UNIQUE` evita vincular el mismo paciente a dos cuentas. MySQL permite que varias cuentas tengan `NULL` en esta columna.

**No confundas los identificadores:** `usuarios.id` identifica una cuenta y `pacientes.id` identifica un paciente. No necesariamente tienen el mismo valor. Tampoco se deben relacionar comparando sus nombres.

Si MySQL indica que una columna ya existe, revisa `DESCRIBE usuarios` y `SHOW CREATE TABLE usuarios`; no borres la tabla ni repitas todo el bloque sin revisar qué se agregó.

### 3.3 Asignar la primera cuenta administradora

Consulta las cuentas:

```sql
SELECT id, nombre_usuario, tipo_usuario, paciente_id FROM usuarios;
```

Elige una cuenta que ya registraste en la práctica 7 y cuyo acceso conozcas. Sustituye `mi_cuenta_admin` por su nombre real:

```sql
UPDATE usuarios
SET tipo_usuario = 'admin'
WHERE nombre_usuario = 'mi_cuenta_admin';
```

Vuelve a consultar para comprobar que **solo la cuenta elegida** tiene el rol esperado. La asignación inicial se hace desde MySQL; el formulario público no permitirá elegir el rol administrador.

### 3.4 Crear códigos de registro

```sql
CREATE TABLE IF NOT EXISTS codigos_acceso (
  codigo VARCHAR(30) PRIMARY KEY,
  tipo_usuario ENUM('medico', 'paciente') NOT NULL
);

INSERT INTO codigos_acceso (codigo, tipo_usuario) VALUES
('MED-P8-2026', 'medico'),
('PAC-P8-2026', 'paciente');
```

Ejecuta el `INSERT` solo una vez. Si los códigos ya existen, consulta la tabla en lugar de insertarlos nuevamente:

```sql
SELECT codigo, tipo_usuario FROM codigos_acceso;
```

Son códigos de ejemplo para el laboratorio y **se pueden reutilizar**. El docente o administrador los proporciona al usuario correspondiente. El código determina el rol de la nueva cuenta; no reemplaza su contraseña y no vincula automáticamente a un paciente. No hay un código público para crear administradores.

**Comprobación:** las tablas anteriores conservan sus datos, tienes una cuenta administradora y existen los dos códigos.

## 4. Agregar el código al formulario de registro

Abre `public/registro.html`. Conserva el formulario de la práctica 7 con:

```html
<form action="/registrar" method="POST">
```

Conserva también los campos `name="nombre_usuario"` y `name="password"`, sus restricciones y el enlace a `/login`.

**Dentro de ese mismo formulario**, antes del botón de registro, agrega:

```html
<label for="codigo_acceso">Código de registro:</label>
<input type="text" id="codigo_acceso" name="codigo_acceso"
       maxlength="30" required>
```

No agregues otro `<form>` dentro del existente ni un selector de roles. El servidor obtendrá el rol de la tabla `codigos_acceso`.

## 5. Ampliar la ruta que registra usuarios

En `server.js`, localiza el bloque completo `app.post('/registrar', ...)` de la práctica 7. **Sustituye únicamente ese bloque** por el siguiente; no dejes ambas versiones. Debe continuar antes de `app.use(requireLogin)`.

```javascript
app.post('/registrar', (req, res, next) => {
  const { nombre_usuario, password, codigo_acceso } = req.body;
  const nombre = typeof nombre_usuario === 'string'
    ? nombre_usuario.trim() : '';
  const codigo = typeof codigo_acceso === 'string'
    ? codigo_acceso.trim() : '';

  if (!nombre || nombre.length > 50 || typeof password !== 'string' ||
      password.length < 8 || Buffer.byteLength(password, 'utf8') > 72 ||
      !codigo || codigo.length > 30) {
    return res.status(400).send('Revisa el usuario, la contraseña y el código.');
  }

  connection.query(
    'SELECT tipo_usuario FROM codigos_acceso WHERE codigo = ?',
    [codigo],
    async (err, codigos) => {
      if (err) return next(err);
      if (codigos.length === 0) {
        return res.status(400).send('Código de registro inválido.');
      }

      const rol = codigos[0].tipo_usuario;
      if (!['medico', 'paciente'].includes(rol)) {
        return res.status(400).send('El código no tiene un rol permitido.');
      }

      try {
        const hash = await bcrypt.hash(password, 10);
        connection.query(
          'INSERT INTO usuarios (nombre_usuario, password_hash, tipo_usuario) VALUES (?, ?, ?)',
          [nombre, hash, rol],
          err => {
            if (err && err.code === 'ER_DUP_ENTRY') {
              return res.status(409).send('Ese nombre de usuario ya existe.');
            }
            if (err) return next(err);
            res.redirect('/login');
          }
        );
      } catch (error) {
        next(error);
      }
    }
  );
});
```

Sigue la secuencia: **recibir campos → validar → consultar código → obtener rol → generar hash → guardar cuenta**. La cuenta conserva `paciente_id = NULL` hasta que se vincule desde MySQL.

Los `?` siguen el formato de las consultas de registro que ya utilizabas. El análisis y las pruebas de inyección SQL se trabajarán en otra actividad.

**Comprobación:** inicia el servidor, registra `medico_prueba` con el código médico y `paciente_prueba` con el código paciente. Usa contraseñas de al menos ocho caracteres y consérvalas para las pruebas. Consulta en MySQL los nombres y roles. Un código inexistente no debe crear una cuenta. Detén el servidor al terminar.

## 6. Guardar el rol al iniciar sesión

No reemplaces toda la ruta `POST /login`. Realiza **dos cambios** en la que ya funciona.

### Cambio A: ampliar el SELECT

Localiza:

```javascript
'SELECT id, password_hash FROM usuarios WHERE nombre_usuario = ?'
```

Sustituye esa cadena por:

```javascript
'SELECT id, nombre_usuario, password_hash, tipo_usuario FROM usuarios WHERE nombre_usuario = ?'
```

Conserva el arreglo `[nombre]` y el callback de la consulta.

### Cambio B: ampliar la sesión

Dentro de `req.session.regenerate(...)`, después de comprobar `err`, localiza `req.session.userId = usuario.id;`. Deja esa parte así:

```javascript
req.session.userId = usuario.id;
req.session.nombreUsuario = usuario.nombre_usuario;
req.session.tipoUsuario = usuario.tipo_usuario;
```

Estas asignaciones van **antes** de `req.session.save(...)`. Conserva la validación, `bcrypt.compare`, `regenerate`, `save` y la redirección a `/` que ya tienes.

Seguimos usando `req.session.userId`, como en la práctica 7. No necesitas crear `req.session.user` ni cambiar `requireLogin`.

**Comprobación:** inicia sesión con una cuenta existente. Debe seguir abriendo el inicio. Cada vez que cambies un rol en MySQL, cierra la sesión y vuelve a entrar: el rol se carga al autenticarte.

## 7. Comprobar los roles en las rutas existentes

### 7.1 Agregar una función de autorización

En `server.js`, debajo de la función `requireLogin` y antes de `app.use(requireLogin)`, agrega:

```javascript
function requireRole(...rolesPermitidos) {
  return (req, res, next) => {
    if (!req.session.userId) return res.redirect('/login');
    if (!rolesPermitidos.includes(req.session.tipoUsuario)) {
      return res.status(403).send('No tienes permiso para realizar esta operación.');
    }
    next();
  };
}
```

`...rolesPermitidos` reúne los argumentos en un arreglo. Por ejemplo, `requireRole('admin', 'medico')` admite cualquiera de esos dos roles. El estado **403** significa que el usuario no tiene permiso.

### 7.2 Modificar el encabezado de cada ruta

Conserva `app.use(requireLogin);` y las rutas existentes debajo de él. Agrega el middleware como argumento antes de la función de cada ruta:

| Ruta existente | Middleware que agregarás |
| --- | --- |
| `POST /submit-data` | `requireRole('admin', 'medico')` |
| `GET /pacientes` | `requireRole('admin', 'medico')` |
| `GET /buscar-pacientes` | `requireRole('admin', 'medico')` |
| `GET /ordenar-pacientes` | `requireRole('admin', 'medico')` |
| `POST /insertar-medico` | `requireRole('admin')` |

Ejemplo: cambia solamente la primera línea de `/pacientes`:

```javascript
app.get('/pacientes', requireRole('admin', 'medico'), (req, res) => {
```

El contenido de la consulta, la construcción de la tabla y el cierre `});` se conservan. Haz lo mismo con las otras cuatro rutas. No pegues el ejemplo como una ruta nueva y vacía.

La página `/`, los archivos estáticos protegidos y `POST /logout` continúan disponibles para los tres roles autenticados.

**Comprobación:** entra como `paciente_prueba` y escribe `/pacientes` directamente en la barra del navegador. Debe aparecer el rechazo. Como médico y administrador sí debe abrirse la tabla.

## 8. Agregar las consultas de cuentas y de datos personales

Estas son **rutas nuevas**. Agrégalas después de `app.use(requireLogin)` y antes del manejador de errores o, si todavía no tienes uno, antes de `app.listen(...)`.

### 8.1 Consultar cuentas como administrador

```javascript
app.get('/ver-usuarios', requireRole('admin'), (req, res, next) => {
  connection.query(
    'SELECT id, nombre_usuario, tipo_usuario, paciente_id FROM usuarios ORDER BY id',
    (err, usuarios) => {
      if (err) return next(err);
      res.json(usuarios);
    }
  );
});
```

Esta consulta devuelve inicialmente JSON para observar los datos. Seleccionamos las columnas necesarias; **no enviamos `password_hash`** al navegador. En una adecuación posterior puedes presentar estos datos en una tabla.

### 8.2 Vincular la cuenta de prueba con un paciente

En MySQL consulta:

```sql
SELECT id, nombre, edad FROM pacientes;
SELECT id, nombre_usuario, tipo_usuario, paciente_id FROM usuarios;
```

Elige un paciente ficticio existente. Si no hay ninguno, inicia sesión como administrador y regístralo con el formulario de pacientes que ya tienes.

En el siguiente ejemplo, **sustituye `3` por el ID real del paciente elegido** y ajusta el nombre de la cuenta si usaste otro:

```sql
UPDATE usuarios
SET paciente_id = 3
WHERE nombre_usuario = 'paciente_prueba'
  AND tipo_usuario = 'paciente';
```

Verifica la relación:

```sql
SELECT u.nombre_usuario, u.tipo_usuario, u.paciente_id, p.nombre
FROM usuarios AS u
LEFT JOIN pacientes AS p ON p.id = u.paciente_id;
```

La vinculación la realiza el administrador desde MySQL para este laboratorio. No agregues al registro público un campo donde cualquier persona elija el ID de otro paciente.

### 8.3 Consultar solamente el registro vinculado

En `server.js`, agrega:

```javascript
app.get('/mis-datos', requireRole('paciente'), (req, res, next) => {
  const sql = `
    SELECT p.id, p.nombre, p.edad, p.frecuencia_cardiaca
    FROM usuarios AS u
    INNER JOIN pacientes AS p ON p.id = u.paciente_id
    WHERE u.id = ?
  `;

  connection.query(sql, [req.session.userId], (err, filas) => {
    if (err) return next(err);
    if (filas.length === 0) {
      return res.status(404).json({
        mensaje: 'Tu cuenta todavía no tiene un paciente vinculado. Solicita la asignación al administrador.'
      });
    }
    res.json(filas[0]);
  });
});
```

`INNER JOIN` relaciona la cuenta con su paciente. `WHERE u.id = ?` usa el ID de **la sesión**, no un ID proporcionado en la dirección del navegador. Así se consulta exclusivamente el registro asociado a quien inició sesión.

**Comprobación:** como paciente, abre `/mis-datos`. Debe mostrar un solo objeto con tus datos ficticios. Una cuenta paciente sin vínculo recibirá el mensaje de asignación pendiente, no una lista de pacientes.

## 9. Informar a la interfaz quién inició sesión

Después de `app.use(requireLogin)`, agrega esta ruta nueva:

```javascript
app.get('/tipo-usuario', (req, res) => {
  res.json({
    nombre_usuario: req.session.nombreUsuario,
    tipo_usuario: req.session.tipoUsuario
  });
});
```

El navegador la consultará con `fetch`. **JSON** es el formato en que se envían los datos. No devuelve contraseñas ni hashes.

**Comprobación:** entra como médico y abre `/tipo-usuario`. Debes ver su nombre de usuario y `"tipo_usuario":"medico"`. Si no aparecen, revisa el paso 6 y vuelve a iniciar sesión.

## 10. Organizar la página principal sin borrar lo anterior

Abre `public/index.html`. Conserva su estructura HTML, encabezado, estilos y formularios. Realiza estos cambios dentro del `body`.

### 10.1 Agregar un espacio para el menú

Debajo de tu encabezado principal agrega:

```html
<div id="navbar"></div>
<p id="estado-sesion" role="status">Cargando opciones de la sesión...</p>
```

### 10.2 Agrupar los controles existentes

Rodea los bloques indicados con estas etiquetas. **Mueve tus controles reales al interior; no los sustituyas por los comentarios.** Cada formulario debe conservar su `action`, `method`, campos y botón.

```html
<section id="panel-pacientes" hidden>
  <!-- Tu título y formulario para registrar pacientes. -->
  <!-- Tu título y formulario para buscar pacientes. -->
  <!-- Tus botones para consultar y ordenar pacientes. -->
</section>

<section id="panel-medicos" hidden>
  <!-- Tu título y formulario para registrar médicos. -->
</section>

<section id="panel-personal" hidden>
  <h2>Mis datos</h2>
  <p>Consulta el registro que el administrador vinculó a tu cuenta.</p>
  <a href="/mis-datos">Consultar mis datos</a>
</section>
```

Cada ID debe aparecer una sola vez. Si los controles estaban en lugares distintos, muévelos completos al panel correspondiente. No dejes botones de consulta general fuera de `panel-pacientes`.

Conserva **fuera de estos paneles** el formulario `POST /logout` que agregaste en la práctica 7. Debe seguir visible para todos los roles. No lo cambies por un enlace: un enlace envía GET y tu cierre de sesión recibe POST.

### 10.3 Enlazar el nuevo archivo JavaScript

Antes de `</body>`, agrega una sola vez:

```html
<script src="/navegacion.js"></script>
```

## 11. Crear el menú y cargarlo en el orden correcto

### 11.1 Crear `public/navbar.html`

Este archivo es un fragmento de HTML. No necesita `html`, `head` ni `body`:

```html
<nav aria-label="Navegación principal">
  <ul id="menu">
    <li><a href="/">Inicio</a></li>
  </ul>
</nav>
```

### 11.2 Crear `public/navegacion.js`

Agrega el siguiente código al archivo nuevo:

```javascript
async function configurarNavegacion() {
  const estado = document.getElementById('estado-sesion');

  try {
    const respuestaMenu = await fetch('/navbar.html');
    if (respuestaMenu.redirected) {
      window.location.href = '/login';
      return;
    }
    if (!respuestaMenu.ok) throw new Error('No se pudo cargar el menú.');

    document.getElementById('navbar').innerHTML = await respuestaMenu.text();

    const respuestaUsuario = await fetch('/tipo-usuario');
    if (respuestaUsuario.redirected) {
      window.location.href = '/login';
      return;
    }
    if (!respuestaUsuario.ok) throw new Error('No se pudo consultar la sesión.');

    const usuario = await respuestaUsuario.json();
    const rol = usuario.tipo_usuario;
    if (!['admin', 'medico', 'paciente'].includes(rol)) {
      throw new Error('Cierra la sesión y vuelve a entrar para cargar tu rol.');
    }

    const menu = document.getElementById('menu');
    function agregarEnlace(texto, destino) {
      const elemento = document.createElement('li');
      const enlace = document.createElement('a');
      enlace.textContent = texto;
      enlace.href = destino;
      elemento.appendChild(enlace);
      menu.appendChild(elemento);
    }

    if (rol === 'admin' || rol === 'medico') {
      document.getElementById('panel-pacientes').hidden = false;
      agregarEnlace('Pacientes', '/pacientes');
      agregarEnlace('Buscar pacientes', '/#panel-pacientes');
      agregarEnlace('Ordenar pacientes', '/ordenar-pacientes');
    }

    if (rol === 'admin') {
      document.getElementById('panel-medicos').hidden = false;
      agregarEnlace('Registrar médico', '/#panel-medicos');
      agregarEnlace('Ver usuarios', '/ver-usuarios');
    }

    if (rol === 'paciente') {
      document.getElementById('panel-personal').hidden = false;
      agregarEnlace('Mis datos', '/mis-datos');
    }

    estado.textContent = `Usuario: ${usuario.nombre_usuario} | Rol: ${rol}`;
  } catch (error) {
    estado.textContent = error.message;
    console.error(error);
  }
}

configurarNavegacion();
```

Lee el código en este orden:

1. Se solicita `navbar.html`.
2. `await` espera la respuesta y luego se inserta el fragmento en la página.
3. Se consulta `/tipo-usuario`.
4. Se agregan los enlaces y se muestran los paneles permitidos.

Es importante insertar el menú **antes de buscar `id="menu"`**. Si se hacen ambas solicitudes por separado sin coordinarse, se puede intentar modificar un elemento que todavía no existe.

`textContent` coloca el nombre del usuario como texto. `innerHTML` se usa aquí únicamente para insertar el fragmento local `navbar.html` que tú escribiste.

El menú se incorpora a la página de inicio. Las tablas de pacientes conservan su botón **Volver**. Las consultas JSON se pueden abandonar con el botón Atrás del navegador. No se agregan enlaces a funciones que todavía no existen.

## 12. Agregar estilos sin sustituir tu diseño

Al final de `public/styles.css`, agrega:

```css
[hidden] {
  display: none !important;
}

nav {
  background-color: #25364a;
  padding: 12px;
  border-radius: 8px;
}

nav ul {
  list-style: none;
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin: 0;
  padding: 0;
}

nav a {
  display: inline-block;
  color: white;
  padding: 6px 10px;
  text-decoration: none;
}

nav a:hover,
nav a:focus-visible {
  background-color: #425c78;
  border-radius: 4px;
}

#estado-sesion {
  text-align: center;
  font-weight: bold;
}
```

La regla `[hidden]` mantiene ocultos los paneles hasta que JavaScript determine cuáles mostrar, incluso si otra regla CSS les asigna `display`.

## 13. Revisar el orden de `server.js`

Tu archivo debe conservar esta organización:

| Orden | Bloque |
| --- | --- |
| 1 | Importaciones existentes, `app` y conexión `connection` |
| 2 | Procesamiento de formularios, sesiones y caché de la práctica 7 |
| 3 | Rutas públicas: CSS, formularios de acceso, `POST /registrar`, `POST /login` |
| 4 | Funciones `requireLogin` y `requireRole` |
| 5 | `app.use(requireLogin)` |
| 6 | Tu único `express.static(...)`, que sigue protegido |
| 7 | Inicio, rutas anteriores con sus permisos y nuevas rutas protegidas |
| 8 | Tu ruta existente `POST /logout` |
| 9 | Manejador de errores, si ya lo tienes |
| 10 | Tu único `app.listen(...)` |

No necesitas completar los pasos pendientes de la práctica 7 para seguir esta guía. Si aún no tienes un manejador de errores propio, Express cuenta con uno predeterminado para los errores enviados con `next(err)`; consulta la terminal cuando haya un fallo. No dupliques el middleware de sesiones ni muevas el registro detrás de `requireLogin`.

## 14. Probar el sistema por etapas

Guarda todos los archivos, reinicia con `node server.js` y usa `http://localhost:3000`. No abras los archivos con doble clic ni con Live Server. Mantén en ejecución solo una copia del proyecto en el puerto 3000.

### 14.1 Registro y navegación

| Prueba | Resultado esperado |
| --- | --- |
| Registrar con un código inexistente | Mensaje de rechazo; no aparece una cuenta nueva en MySQL |
| Registrar con código médico o paciente | Cuenta con el rol asignado por ese código |
| Repetir un nombre de usuario | Mensaje de nombre existente |
| Entrar como administrador | Paneles de pacientes y médicos; enlace a usuarios |
| Entrar como médico | Panel de pacientes; no aparece registro de médicos ni lista de usuarios |
| Entrar como paciente | Panel personal; no aparecen controles para otros pacientes |
| Cerrar sesión desde cualquier rol | Redirección a `/login` |
| Abrir `/index.html` o `/tipo-usuario` sin sesión | Redirección a `/login` |

Usa cerrar sesión entre cuentas. Las pestañas del mismo navegador normalmente comparten la sesión; abrir otra pestaña no crea un usuario independiente.

### 14.2 Permisos desde la dirección del navegador

Prueba estas rutas escribiéndolas directamente. No basta con comprobar el menú.

| Ruta GET | Administrador | Médico | Paciente |
| --- | --- | --- | --- |
| `/pacientes` | Tabla | Tabla | 403 |
| `/buscar-pacientes` | Resultados | Resultados | 403 |
| `/ordenar-pacientes` | Tabla ordenada | Tabla ordenada | 403 |
| `/ver-usuarios` | JSON sin hashes | 403 | 403 |
| `/mis-datos` | 403 | 403 | Datos propios o mensaje de vínculo pendiente |

### 14.3 Verificar los formularios POST

1. Como administrador, registra un paciente ficticio y un médico ficticio.
2. Como médico, registra un paciente ficticio.
3. Como paciente, comprueba que el servidor rechaza un registro aunque se envíe una petición directamente.

Para esta última comprobación, estando en la página de inicio como paciente, abre **F12 → Consola** y ejecuta:

```javascript
fetch('/submit-data', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: new URLSearchParams({
    name: 'PruebaNoPermitida', age: '20', heart_rate: '70'
  })
}).then(respuesta => console.log('Estado:', respuesta.status));
```

Debe mostrar **403** y no insertar el registro. Como médico, comprueba de igual forma el registro de médicos:

```javascript
fetch('/insertar-medico', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: new URLSearchParams({
    medico_name: 'PruebaNoPermitida', especialidad: 'General'
  })
}).then(respuesta => console.log('Estado:', respuesta.status));
```

También debe mostrar **403**. Estas son pruebas de permisos por rol; no son pruebas de inyección SQL. Comprueba en MySQL que los registros rechazados no se guardaron.

### 14.4 Comprobar los datos propios

1. Registra una segunda cuenta paciente con el código correspondiente.
2. Antes de vincularla, abre `/mis-datos`: debe informar que falta la asignación.
3. Vincúlala desde MySQL con **otro** paciente ficticio.
4. Inicia sesión por separado con ambas cuentas y verifica que cada una recibe únicamente su registro.

## 15. Problemas frecuentes

| Situación | Qué revisar |
| --- | --- |
| `Unknown column 'username'` o `'password'` | Los nombres de la práctica 7 son `nombre_usuario` y `password_hash` |
| El rol no aparece o todos reciben 403 | Incluir `tipo_usuario` en el SELECT de login, asignar `req.session.tipoUsuario` y volver a entrar |
| El menú no aparece | Archivo `navegacion.js`, enlace al script, consola del navegador y carga secuencial con `await` |
| `Cannot read properties of null` | IDs de los paneles y del menú; no duplicarlos ni cambiar su escritura |
| El registro no recibe los datos | Formulario `POST /registrar`; campos `nombre_usuario`, `password`, `codigo_acceso` |
| Una ruta ignora el permiso | Buscar una versión duplicada de la ruta antes de la protegida |
| El paciente ve controles generales | Todos los controles deben estar dentro de sus paneles; revisar `[hidden]` |
| `/mis-datos` informa que no hay vínculo | Revisar `usuarios.paciente_id` y su relación con `pacientes.id` |
| Falla la llave foránea al vincular | El ID de paciente elegido debe existir |
| Aparece un duplicado al vincular | Ese paciente ya está asignado a otra cuenta |
| Cerrar sesión devuelve `Cannot GET /logout` | Usar el formulario con método POST de la práctica 7 |
| `EADDRINUSE` | Detener la otra copia del servidor que usa el puerto 3000 |

## Evidencias de entrega

1. Carpeta del proyecto de la práctica 8, con los archivos anteriores y los cambios integrados. Puedes omitir `node_modules`; conserva los archivos `package.json` y `package-lock.json`.
2. Capturas del inicio con cada uno de los tres roles.
3. Consulta SQL que muestre `nombre_usuario`, `tipo_usuario` y `paciente_id`, sin mostrar contraseñas ni hashes.
4. Evidencia de rechazo de una ruta GET y una operación POST por falta de permiso.
5. Evidencia de dos cuentas paciente que consultan registros distintos y de una cuenta sin vínculo.
6. Explicación breve, con tus palabras:
   - ¿Qué diferencia hay entre iniciar sesión y tener permiso?
   - ¿Por qué no basta con ocultar un botón?
   - ¿Para qué sirve `paciente_id` y por qué no usamos directamente `usuarios.id` como ID de paciente?
   - ¿Por qué esperamos a cargar `navbar.html` antes de modificar el menú?

**Personalización:** mejora colores, distribución y textos conservando los permisos. Como adecuación adicional, presenta `/ver-usuarios` y `/mis-datos` en páginas con tablas y un enlace para volver, reutilizando lo aprendido en la práctica 5. La funcionalidad base debe quedar terminada antes de esta adecuación.

## Material de referencia

- [Práctica 4: servidor y conexión con MySQL](https://github.com/fortunatoramirez/tbd/blob/main/tbd-p4.md)
- [Práctica 5: CSS y tablas HTML](https://github.com/fortunatoramirez/tbd/blob/main/tbd-p5.md)
- [Práctica 6: filtros, ordenación y médicos](https://github.com/fortunatoramirez/tbd/blob/main/tbd-p6.md)
- [Práctica 7: registro y sesiones; usar hasta el paso 9](https://github.com/fortunatoramirez/tbd/blob/main/tbd-p7.md)
