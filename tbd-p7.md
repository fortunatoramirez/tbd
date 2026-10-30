# Práctica 7: Autenticación de usuarios y protección de rutas

## Objetivo

Agregar registro, inicio y cierre de sesión a la aplicación de las prácticas 4, 5 y 6. Solo los usuarios con una sesión activa podrán registrar pacientes y médicos, consultar pacientes, buscarlos y ordenarlos. Las contraseñas se almacenarán como hashes usando `bcrypt`.

## Antes de empezar: correcciones e integración

Esta versión corrige los siguientes problemas de la práctica original:

- Faltaban las rutas **GET `/login` y GET `/registro`**. Un formulario con `POST /login` no crea la página que se abre con `GET /login`.
- `express.static(...)`, utilizado en la práctica 4, podía entregar `/` o `/index.html` antes de comprobar la sesión.
- Agregar otra ruta con el mismo método y dirección no sustituye la anterior: una ruta antigua sin protección podía responder primero.
- Faltaba proteger `/pacientes` y `/submit-data`.
- Había bloques con comentarios en lugar de la lógica completa y un `index.html` que podía eliminar los formularios anteriores.
- La contraseña de MySQL se cambiaba a una cadena vacía sin explicar que debía conservarse la configuración anterior.
- El cierre de sesión no esperaba a que terminara `destroy()`. Ahora también se regenera y guarda la sesión al iniciar sesión, y se manejan los errores.
- La búsqueda heredada de la práctica 6 concatenaba entradas del usuario en SQL. Ahora utiliza parámetros `?`. Los valores mostrados en HTML también se escapan.

**Trabaja sobre una copia de `gestion-pacientes`. Reemplaza completamente `server.js` con el código de esta práctica; no lo pegues al final del servidor anterior.** Se conservan los nombres de tablas, campos y formularios de las prácticas anteriores. No borres tus tablas ni sus registros.

Esta es una aplicación didáctica local: usa datos ficticios. Todos los usuarios autenticados tienen el mismo acceso; todavía no implementamos roles ni permisos por paciente.

## Paso 1. Preparar las dependencias y la base de datos

1. Detén el servidor anterior con **Ctrl+C**.
2. Abre la terminal en la carpeta `gestion-pacientes` y ejecuta:

```bash
npm install express mysql2 express-session bcrypt
```

`express.urlencoded()` procesa los formularios, por lo que este servidor ya no necesita importar `body-parser`. No es necesario desinstalarlo.

3. En MySQL, selecciona la base utilizada desde la práctica 4 y crea la tabla de usuarios:

```sql
USE gestion_pacientes;

CREATE TABLE IF NOT EXISTS usuarios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre_usuario VARCHAR(50) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL
);

SHOW TABLES;
DESCRIBE usuarios;
```

Deben existir `pacientes`, `medicos` y `usuarios`. La tabla `pacientes` debe tener `id`, `nombre`, `edad` y `frecuencia_cardiaca`; `medicos`, `id`, `nombre` y `especialidad`.

Si ya creaste `usuarios` con la versión original, puedes conservarla: sus columnas son compatibles. `IF NOT EXISTS` no modifica una tabla existente. Los usuarios deben registrarse mediante la página para que su contraseña se guarde como hash; no escribas una contraseña en texto plano en `password_hash`.

## Paso 2. Organizar los archivos

| Archivo | Función |
| --- | --- |
| `server.js` | Servidor completo, sesiones y consultas SQL |
| `public/index.html` | Página principal protegida |
| `public/login.html` | Formulario de inicio de sesión |
| `public/registro.html` | Formulario de registro de usuarios |
| `public/styles.css` | Conserva los estilos de las prácticas 5 y 6 |

Aunque `index.html` está dentro de `public`, **no publicaremos toda esa carpeta con `express.static`**. El servidor enviará cada HTML desde su ruta correspondiente y comprobará la sesión cuando sea necesario. El CSS sí estará disponible sin iniciar sesión.

## Paso 3. Crear `public/registro.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Registro de usuario</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <h1>Registro de usuario</h1>
  <form action="/registrar" method="POST">
    <label for="nombre_usuario">Nombre de usuario:</label>
    <input id="nombre_usuario" name="nombre_usuario" type="text"
           maxlength="50" autocomplete="username" required>
    <label for="password">Contraseña:</label>
    <input id="password" name="password" type="password"
           minlength="8" autocomplete="new-password" required>
    <p>Utiliza al menos 8 caracteres y máximo 72 bytes. Los acentos y emojis pueden ocupar varios bytes.</p>
    <button type="submit">Registrar</button>
  </form>
  <p>¿Ya tienes cuenta? <a href="/login">Inicia sesión</a></p>
</body>
</html>
```

El atributo `name` determina la clave que recibirá `req.body`. Por ejemplo, `name="nombre_usuario"` corresponde a `req.body.nombre_usuario`. La validación del navegador se complementa con validación en el servidor.

## Paso 4. Crear `public/login.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Iniciar sesión</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <h1>Iniciar sesión</h1>
  <form action="/login" method="POST">
    <label for="nombre_usuario">Nombre de usuario:</label>
    <input id="nombre_usuario" name="nombre_usuario" type="text"
           maxlength="50" autocomplete="username" required>
    <label for="password">Contraseña:</label>
    <input id="password" name="password" type="password"
           autocomplete="current-password" required>
    <button type="submit">Iniciar sesión</button>
  </form>
  <p>¿No tienes cuenta? <a href="/registro">Regístrate aquí</a></p>
</body>
</html>
```

## Paso 5. Actualizar `public/index.html`

Puedes reemplazarlo con esta versión completa, que incluye las funciones anteriores. Si conservas tu diseño, mantén los mismos `action`, `method` y `name`, e incorpora el formulario de cierre de sesión.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gestión de pacientes y médicos</title>
  <link rel="stylesheet" href="/styles.css">
</head>
<body>
  <h1>Gestión de pacientes y médicos</h1>
  <form action="/logout" method="POST">
    <button type="submit">Cerrar sesión</button>
  </form>

  <h2>Registrar paciente</h2>
  <form action="/submit-data" method="POST">
    <label for="name">Nombre del paciente:</label>
    <input id="name" name="name" type="text" maxlength="100" required>
    <label for="age">Edad:</label>
    <input id="age" name="age" type="number" min="0" max="150" step="1" required>
    <label for="heart-rate">Frecuencia cardiaca (bpm):</label>
    <input id="heart-rate" name="heart_rate" type="number" min="1" max="300" step="1" required>
    <button type="submit">Guardar paciente</button>
  </form>

  <h2>Buscar pacientes</h2>
  <form action="/buscar-pacientes" method="GET">
    <label for="name-search">Nombre del paciente:</label>
    <input id="name-search" name="name_search" type="text">
    <label for="age-search">Edad:</label>
    <input id="age-search" name="age_search" type="number" min="0" max="150" step="1">
    <button type="submit">Buscar</button>
  </form>
  <p><a href="/pacientes">Ver pacientes guardados</a></p>
  <p><a href="/ordenar-pacientes">Ordenar por frecuencia cardiaca</a></p>

  <h2>Registrar médico</h2>
  <form action="/insertar-medico" method="POST">
    <label for="medico-name">Nombre del médico:</label>
    <input id="medico-name" name="medico_name" type="text" maxlength="100" required>
    <label for="especialidad">Especialidad:</label>
    <input id="especialidad" name="especialidad" type="text" maxlength="100" required>
    <button type="submit">Guardar médico</button>
  </form>
</body>
</html>
```

El cierre de sesión ahora se realiza con **POST `/logout`**: reemplaza el antiguo enlace `<a href="/logout">` por el formulario. Los rangos numéricos son límites didácticos para validar entradas; no representan criterios clínicos.

## Paso 6. Reemplazar completamente `server.js`

**No conserves las rutas anteriores, el `express.static` anterior ni otro `app.listen`.** El siguiente código ya incluye todas las funciones de las prácticas 4 a 6.

Cambia `TU_CONTRASENA_MYSQL` por la contraseña con la que ya te conectabas. No es la contraseña del usuario que registrarás en la página: son cuentas distintas.

```javascript
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');
const mysql = require('mysql2');
const path = require('path');
const crypto = require('crypto');
const app = express();

// Conserva los datos de conexión que ya funcionaban en la práctica 6.
const connection = mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: 'TU_CONTRASENA_MYSQL', // Usa '' solamente si tu cuenta no tiene contraseña.
  database: 'gestion_pacientes'
});

app.use(express.urlencoded({ extended: false }));
app.use(session({
  secret: process.env.SESSION_SECRET || crypto.randomBytes(32).toString('hex'),
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, sameSite: 'lax', secure: false }
}));

// Publicamos únicamente el CSS. NO uses express.static('public') aquí:
// permitiría abrir el index.html privado sin pasar por requireLogin.
app.get('/styles.css', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'styles.css'));
});

// Evita que el navegador reutilice páginas privadas después de cerrar sesión.
app.use((req, res, next) => {
  res.set('Cache-Control', 'no-store');
  next();
});

function requireLogin(req, res, next) {
  if (!req.session.userId) return res.redirect('/login');
  next();
}

// Convierte caracteres especiales antes de insertar datos en el HTML.
function escapeHtml(value) {
  return String(value ?? '').replace(/[&<>"']/g, char => ({
    '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
  }[char]));
}

function texto(value) {
  return typeof value === 'string' ? value.trim() : '';
}

function enteroEnRango(value, min, max) {
  return typeof value === 'string' && /^\d+$/.test(value) &&
    Number.isSafeInteger(Number(value)) && Number(value) >= min && Number(value) <= max;
}

function mostrarPacientes(res, titulo, results) {
  const filas = results.map(p => `<tr>
    <td>${escapeHtml(p.nombre)}</td>
    <td>${escapeHtml(p.edad)}</td>
    <td>${escapeHtml(p.frecuencia_cardiaca)}</td>
  </tr>`).join('');
  res.send(`<!DOCTYPE html>
    <html lang="es"><head><meta charset="UTF-8">
      <title>${escapeHtml(titulo)}</title><link rel="stylesheet" href="/styles.css">
    </head><body><h1>${escapeHtml(titulo)}</h1>
      <table><thead><tr><th>Nombre</th><th>Edad</th>
        <th>Frecuencia cardiaca (bpm)</th></tr></thead>
        <tbody>${filas || '<tr><td colspan="3">No se encontraron pacientes.</td></tr>'}</tbody>
      </table><p><a href="/">Volver al inicio</a></p>
    </body></html>`);
}

// Estas páginas son públicas: deben declararse ANTES de app.use(requireLogin).
app.get(['/login', '/login.html'], (req, res) => {
  if (req.session.userId) return res.redirect('/');
  res.sendFile(path.join(__dirname, 'public', 'login.html'));
});

app.get(['/registro', '/registro.html'], (req, res) => {
  if (req.session.userId) return res.redirect('/');
  res.sendFile(path.join(__dirname, 'public', 'registro.html'));
});

app.post('/registrar', async (req, res, next) => {
  const nombre = texto(req.body.nombre_usuario);
  const password = req.body.password;
  if (!nombre || nombre.length > 50 || typeof password !== 'string' ||
      password.length < 8 || Buffer.byteLength(password, 'utf8') > 72) {
    return res.status(400).send('Usa un usuario de 1 a 50 caracteres y una contraseña de al menos 8 caracteres y máximo 72 bytes. Regresa e inténtalo de nuevo.');
  }
  try {
    const hash = await bcrypt.hash(password, 10);
    connection.query(
      'INSERT INTO usuarios (nombre_usuario, password_hash) VALUES (?, ?)',
      [nombre, hash], err => {
        if (err && err.code === 'ER_DUP_ENTRY') {
          return res.status(409).send('Ese nombre de usuario ya existe. Regresa y elige otro.');
        }
        if (err) return next(err);
        res.redirect('/login');
      }
    );
  } catch (err) { next(err); }
});

app.post('/login', (req, res, next) => {
  const nombre = texto(req.body.nombre_usuario);
  const password = req.body.password;
  if (!nombre || nombre.length > 50 || typeof password !== 'string' ||
      !password || Buffer.byteLength(password, 'utf8') > 72) {
    return res.status(400).send('Introduce un usuario y una contraseña válidos.');
  }
  connection.query(
    'SELECT id, password_hash FROM usuarios WHERE nombre_usuario = ?',
    [nombre], async (err, results) => {
      if (err) return next(err);
      try {
        const user = results[0];
        if (!user || !(await bcrypt.compare(password, user.password_hash))) {
          return res.status(401).send('Usuario o contraseña incorrectos. Regresa e inténtalo de nuevo.');
        }
        // Cambia el identificador de sesión al autenticar al usuario.
        req.session.regenerate(err => {
          if (err) return next(err);
          req.session.userId = user.id;
          // Guarda la sesión ANTES de redirigir a la página protegida.
          req.session.save(err => {
            if (err) return next(err);
            res.redirect('/');
          });
        });
      } catch (error) { next(error); }
    }
  );
});

// Todo lo declarado debajo de esta línea requiere una sesión activa.
app.use(requireLogin);

app.get(['/', '/index.html'], (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.post('/logout', (req, res, next) => {
  req.session.destroy(err => {
    if (err) return next(err);
    res.clearCookie('connect.sid', { path: '/' });
    res.redirect('/login');
  });
});

app.post('/submit-data', (req, res, next) => {
  const nombre = texto(req.body.name);
  const { age, heart_rate } = req.body;
  if (!nombre || nombre.length > 100 || !enteroEnRango(age, 0, 150) ||
      !enteroEnRango(heart_rate, 1, 300)) {
    return res.status(400).send('Revisa el nombre (máximo 100 caracteres), la edad (0 a 150) y la frecuencia cardiaca (1 a 300). Usa números enteros.');
  }
  connection.query(
    'INSERT INTO pacientes (nombre, edad, frecuencia_cardiaca) VALUES (?, ?, ?)',
    [nombre, Number(age), Number(heart_rate)], err => {
      if (err) return next(err);
      res.redirect('/pacientes');
    }
  );
});

app.get('/pacientes', (req, res, next) => {
  connection.query('SELECT * FROM pacientes', (err, results) => {
    if (err) return next(err);
    mostrarPacientes(res, 'Pacientes registrados', results);
  });
});

app.get('/buscar-pacientes', (req, res, next) => {
  const nombre = texto(req.query.name_search);
  const edad = texto(req.query.age_search);
  if (edad && !enteroEnRango(edad, 0, 150)) {
    return res.status(400).send('La edad de búsqueda debe ser un entero entre 0 y 150.');
  }
  let query = 'SELECT * FROM pacientes WHERE 1=1';
  const valores = [];
  if (nombre) {
    query += ' AND nombre LIKE ?';
    valores.push(`%${nombre}%`);
  }
  if (edad) {
    query += ' AND edad = ?';
    valores.push(Number(edad));
  }
  connection.query(query, valores, (err, results) => {
    if (err) return next(err);
    mostrarPacientes(res, 'Resultados de búsqueda', results);
  });
});

app.get('/ordenar-pacientes', (req, res, next) => {
  connection.query('SELECT * FROM pacientes ORDER BY frecuencia_cardiaca DESC',
    (err, results) => {
      if (err) return next(err);
      mostrarPacientes(res, 'Pacientes ordenados por frecuencia cardiaca', results);
    }
  );
});

app.post('/insertar-medico', (req, res, next) => {
  const nombre = texto(req.body.medico_name);
  const especialidad = texto(req.body.especialidad);
  if (!nombre || !especialidad || nombre.length > 100 || especialidad.length > 100) {
    return res.status(400).send('Escribe nombre y especialidad, cada uno con máximo 100 caracteres.');
  }
  connection.query('INSERT INTO medicos (nombre, especialidad) VALUES (?, ?)',
    [nombre, especialidad], err => {
      if (err) return next(err);
      res.redirect('/');
    }
  );
});

// Maneja errores de MySQL, bcrypt, sesiones y archivos sin dejar peticiones pendientes.
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send('Ocurrió un error en el servidor. Revisa la terminal de Node.js.');
});

// No abre el servidor si las credenciales de MySQL son incorrectas.
connection.connect(err => {
  if (err) {
    console.error('No se pudo conectar a MySQL:', err.message);
    process.exitCode = 1;
    connection.destroy();
    return;
  }
  app.listen(3000, () => {
    console.log('Conexión a MySQL correcta. Abre http://localhost:3000');
  });
});
```

## Paso 7. Entender el inicio de sesión

1. Al abrir `/`, `requireLogin` comprueba si existe `req.session.userId`. Si no existe, redirige a `/login`.
2. **GET `/login`** entrega el archivo del formulario. **POST `/login`** recibe las credenciales cuando pulsas el botón. Son operaciones diferentes aunque compartan dirección.
3. El servidor busca el usuario en MySQL y utiliza `bcrypt.compare()` para comprobar la contraseña contra su hash. No se descifra el hash ni se genera otro hash para compararlo como texto.
4. Si los datos son correctos, `regenerate()` crea un identificador de sesión nuevo. El servidor guarda `userId`, espera a que `save()` termine y redirige a `/`.
5. El navegador conserva una cookie llamada `connect.sid`, que identifica la sesión. La contraseña no se guarda en esa cookie.
6. Las rutas declaradas después de `app.use(requireLogin)` requieren una sesión activa. Esto incluye las consultas y las inserciones, no solo el HTML principal.
7. Al cerrar sesión, el servidor espera a que `destroy()` termine, elimina la cookie y redirige a `/login`.

**¿Por qué importa el orden?** Express revisa los manejadores en el orden en que se declararon. Si una ruta antigua o un servidor de archivos estáticos entrega la página primero, la comprobación posterior de sesión no se ejecuta.

**Sesiones en esta práctica:** `express-session` utiliza memoria del proceso por defecto. Al reiniciar Node.js, las sesiones se pierden y hay que iniciar sesión otra vez; las cuentas permanecen en MySQL. El secreto generado automáticamente también cambia al reiniciar si no defines `SESSION_SECRET`. `secure: false` permite la cookie en `http://localhost:3000`. Para un despliegue real se requieren HTTPS, un secreto persistente, un almacén de sesiones apropiado y controles adicionales como protección CSRF y límites de intentos de acceso.

## Paso 8. Ejecutar y comprobar

En la terminal, desde `gestion-pacientes`:

```bash
node server.js
```

Abre **http://localhost:3000**. No abras los HTML con doble clic ni con Live Server: los formularios deben comunicarse con este servidor Express.

Realiza las pruebas en este orden:

| Prueba | Resultado esperado |
| --- | --- |
| Abrir `/` sin sesión | Redirige a `/login` y muestra el formulario |
| Seguir el enlace de registro | Abre `/registro` sin errores `Cannot GET` |
| Registrar una cuenta nueva con contraseña válida | Redirige a `/login`; el hash queda guardado en `usuarios` |
| Registrar el mismo nombre otra vez | Informa que ya existe; no detiene el servidor |
| Iniciar sesión con contraseña equivocada | Muestra un error y no permite acceder a `/` |
| Iniciar sesión con la contraseña correcta | Abre la página principal con todos sus formularios |
| Registrar un paciente | Lo guarda y muestra la tabla de pacientes |
| Buscar por nombre, edad y ambos filtros | Devuelve los registros que corresponden; sin filtros muestra todos |
| Buscar un nombre con apóstrofo, por ejemplo `O'Connor` | No produce errores de sintaxis SQL |
| Ordenar por frecuencia cardiaca | Muestra los valores de mayor a menor |
| Registrar un médico | Regresa al inicio; comprueba el registro con `SELECT * FROM medicos;` |
| Cerrar sesión y abrir `/` o `/index.html` | Redirige a `/login` |
| Sin sesión, abrir `/pacientes`, `/buscar-pacientes` o `/ordenar-pacientes` | Redirige a `/login` |
| Reiniciar Node.js y volver a la aplicación | Pide iniciar sesión otra vez; las cuentas siguen existiendo |

Para verificar que también se bloquea una inserción sin sesión, abre **otra terminal** y ejecuta:

```bash
curl -i -X POST http://localhost:3000/submit-data -d "name=PruebaSinSesion&age=20&heart_rate=70"
curl -i -X POST http://localhost:3000/insertar-medico -d "medico_name=PruebaSinSesion&especialidad=General"
```

En Windows PowerShell, utiliza `curl.exe` si `curl` se interpreta como otro comando. Como estas peticiones no llevan la cookie del navegador, ambas deben responder **302** con `Location: /login` y **no insertar registros**. Escribir la dirección en la barra del navegador envía GET y no comprueba una ruta POST.

## Problemas frecuentes

| Síntoma | Qué revisar |
| --- | --- |
| `Cannot GET /login` o `/registro` | Copiaste todo el servidor nuevo y reiniciaste Node.js |
| La página principal abre sin sesión | Quedó un `express.static` o una ruta anterior; reemplaza el servidor completo y prueba en una ventana privada |
| Inicio de sesión correcto pero vuelve al formulario | Usa siempre el mismo host (`localhost`), permite cookies y verifica que no estés reiniciando el servidor |
| `Access denied` al iniciar Node.js | Usuario y contraseña de MySQL; no son los datos del formulario web |
| `Table ... doesn't exist` | Seleccionaste `gestion_pacientes` y creaste las tablas necesarias |
| `EADDRINUSE` | Sigue activo otro servidor en el puerto 3000; detén la terminal anterior con Ctrl+C |
| Error al iniciar sesión con un usuario insertado manualmente | `password_hash` debe contener un hash bcrypt válido; registra una cuenta nueva desde `/registro` |
| Al pulsar Atrás se ve una pantalla anterior | Una vista del historial no prueba que exista acceso; recarga o solicita una ruta protegida para comprobarlo |

## Paso 9. Evidencias y adecuaciones

Entrega capturas del registro, inicio de sesión, tabla de pacientes y redirección al intentar acceder sin sesión. Explica la diferencia entre GET y POST, la función de `requireLogin` y por qué ocultar un botón no protege una ruta.

Personaliza `styles.css` y conserva el funcionamiento de los formularios. No incluyas contraseñas reales en las capturas ni publiques credenciales de MySQL en el repositorio.
