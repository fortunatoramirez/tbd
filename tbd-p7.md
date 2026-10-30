# Práctica 7: Autenticación de usuarios y protección de rutas

## Objetivo

Incorporar registro, inicio y cierre de sesión al sistema de gestión de pacientes y médicos desarrollado en las prácticas anteriores. Utilizar sesiones para controlar el acceso y almacenar contraseñas mediante hashing.

## Requisitos

- Tener funcionando el proyecto de la práctica 6.
- Conservar la base de datos `gestion_pacientes`, sus tablas y registros.
- Trabajar sobre el mismo proyecto y guardar una copia de respaldo antes de comenzar.

En esta práctica **agregarás funciones a tu sistema actual**. Conserva la conexión a MySQL, los formularios, las consultas y los estilos que ya desarrollaste. Los fragmentos siguientes indican qué incorporar y dónde hacerlo; no constituyen un archivo `server.js` completo.

## Paso 1. Instalar las dependencias

Detén el servidor con **Ctrl+C**. En la terminal de tu proyecto ejecuta:

```bash
npm install express-session bcrypt
```

- **express-session:** permite que el servidor recuerde que un usuario inició sesión.
- **bcrypt:** genera y verifica hashes de contraseñas. Un hash no se descifra para recuperar la contraseña.

Al principio de `server.js`, junto a las importaciones existentes, agrega:

```javascript
const session = require('express-session');
const bcrypt = require('bcrypt');
const crypto = require('crypto');
```

Comprueba que ya tengas `const path = require('path');`. Agrégalo solo si falta. No repitas las declaraciones de `express`, `app`, `mysql` ni `connection`.

## Paso 2. Crear la tabla de usuarios

En MySQL ejecuta:

```sql
USE gestion_pacientes;

CREATE TABLE IF NOT EXISTS usuarios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre_usuario VARCHAR(50) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL
);
```

`UNIQUE` evita registrar dos veces el mismo nombre de usuario. `password_hash` almacenará el resultado de bcrypt, no la contraseña escrita en el formulario.

Las cuentas de esta tabla pertenecen a la aplicación. Son distintas de la cuenta de MySQL que utiliza `connection`; conserva los datos de conexión que ya te funcionan.

## Paso 3. Preparar los formularios de acceso

Crea `login.html` y `registro.html` dentro de `public`. En cada archivo escribe la estructura básica de una página HTML, indica el idioma español y la codificación UTF-8, agrega un título y enlaza tu hoja de estilos:

```html
<link rel="stylesheet" href="/styles.css">
```

Dentro del `body` de **registro.html**, incorpora:

```html
<h2>Registro de usuario</h2>
<form action="/registrar" method="POST">
  <label for="nombre_usuario">Nombre de usuario:</label>
  <input type="text" id="nombre_usuario" name="nombre_usuario"
         maxlength="50" autocomplete="username" required>

  <label for="password">Contraseña:</label>
  <input type="password" id="password" name="password"
         minlength="8" autocomplete="new-password" required>

  <p>Usa al menos 8 caracteres y máximo 72 bytes.
     Los acentos y emojis pueden ocupar varios bytes.</p>
  <button type="submit">Registrar</button>
</form>
<p>¿Ya tienes cuenta? <a href="/login">Inicia sesión</a></p>
```

Para **login.html**, construye un formulario similar y realiza estas adecuaciones:

- Título: **Iniciar sesión**.
- Destino del formulario: `action="/login"`, con método `POST`.
- Conserva los campos `name="nombre_usuario"` y `name="password"`.
- En la contraseña utiliza `autocomplete="current-password"`.
- Texto del botón: **Iniciar sesión**.
- Agrega un enlace a `/registro` con el texto **Crear cuenta**.

El atributo `name` determina la clave que recibes en `req.body`. Los nombres de los campos deben coincidir con los utilizados por el servidor.

## Paso 4. Configurar las sesiones

Localiza la línea que procesa los formularios, por ejemplo:

```javascript
app.use(bodyParser.urlencoded({ extended: true }));
```

Consérvala. Si utilizas `express.urlencoded(...)`, cumple la misma función; basta con uno de los dos.

**Después de esa línea y antes de cualquier ruta**, agrega:

```javascript
app.use(session({
  secret: process.env.SESSION_SECRET || crypto.randomBytes(32).toString('hex'),
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    sameSite: 'lax',
    secure: false
  }
}));

app.use((req, res, next) => {
  res.set('Cache-Control', 'no-store');
  next();
});
```

La cookie identifica la sesión del navegador; no contiene la contraseña. `secure: false` permite trabajar con HTTP en localhost. `no-store` indica al navegador que no almacene las respuestas para reutilizarlas después.

En este ejercicio las sesiones se guardan en memoria. Al reiniciar Node.js tendrás que iniciar sesión otra vez, pero tus usuarios seguirán en MySQL. El secreto generado también cambia al reiniciar si no defines `SESSION_SECRET`.

## Paso 5. Crear las rutas públicas

Una ruta **GET** muestra la página y una ruta **POST** recibe el formulario. Aunque compartan una dirección, debes definir ambas operaciones.

Después de configurar las sesiones, agrega:

```javascript
app.get('/styles.css', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'styles.css'));
});

app.get(['/login', '/login.html'], (req, res) => {
  if (req.session.userId) return res.redirect('/');
  res.sendFile(path.join(__dirname, 'public', 'login.html'));
});

app.get(['/registro', '/registro.html'], (req, res) => {
  if (req.session.userId) return res.redirect('/');
  res.sendFile(path.join(__dirname, 'public', 'registro.html'));
});
```

Estas páginas deben abrirse sin haber iniciado sesión. El CSS también estará disponible para que los formularios conserven el diseño del proyecto.

## Paso 6. Registrar usuarios

Debajo de las rutas públicas, agrega la ruta que recibe el formulario de registro:

```javascript
app.post('/registrar', async (req, res, next) => {
  const { nombre_usuario, password } = req.body;
  const nombre = typeof nombre_usuario === 'string' ? nombre_usuario.trim() : '';

  if (!nombre || nombre.length > 50 || typeof password !== 'string' ||
      password.length < 8 || Buffer.byteLength(password, 'utf8') > 72) {
    return res.status(400).send('Revisa el usuario y la contraseña. Regresa al formulario.');
  }

  try {
    const hash = await bcrypt.hash(password, 10);

    connection.query(
      'INSERT INTO usuarios (nombre_usuario, password_hash) VALUES (?, ?)',
      [nombre, hash],
      err => {
        if (err && err.code === 'ER_DUP_ENTRY') {
          return res.status(409).send('Ese usuario ya existe. Elige otro nombre.');
        }
        if (err) return next(err);
        res.redirect('/login');
      }
    );
  } catch (err) {
    next(err);
  }
});
```

Observa la secuencia: validar → generar el hash → insertar → mostrar el inicio de sesión. Los signos `?` separan los valores enviados por el usuario de la estructura de la consulta SQL.

Registrar una cuenta no inicia la sesión automáticamente: el usuario deberá entrar con sus credenciales.

## Paso 7. Iniciar sesión

Agrega esta ruta debajo de la anterior:

```javascript
app.post('/login', (req, res, next) => {
  const { nombre_usuario, password } = req.body;
  const nombre = typeof nombre_usuario === 'string' ? nombre_usuario.trim() : '';

  if (!nombre || nombre.length > 50 || typeof password !== 'string' ||
      !password || Buffer.byteLength(password, 'utf8') > 72) {
    return res.status(400).send('Introduce un usuario y una contraseña válidos.');
  }

  connection.query(
    'SELECT id, password_hash FROM usuarios WHERE nombre_usuario = ?',
    [nombre],
    async (err, results) => {
      if (err) return next(err);

      try {
        const usuario = results[0];

        if (!usuario || !(await bcrypt.compare(password, usuario.password_hash))) {
          return res.status(401).send('Usuario o contraseña incorrectos.');
        }

        req.session.regenerate(err => {
          if (err) return next(err);

          req.session.userId = usuario.id;
          req.session.save(err => {
            if (err) return next(err);
            res.redirect('/');
          });
        });
      } catch (error) {
        next(error);
      }
    }
  );
});
```

- `bcrypt.compare()` comprueba la contraseña contra el hash almacenado.
- `regenerate()` crea un identificador de sesión nuevo al autenticar al usuario.
- `userId` identifica al usuario autenticado.
- `save()` termina de guardar la sesión antes de redirigir a la página principal.
- `next(err)` envía el error al manejador que agregarás más adelante.

## Paso 8. Proteger las funciones que ya tienes

Después de las rutas de registro e inicio de sesión, incorpora:

```javascript
function requireLogin(req, res, next) {
  if (!req.session.userId) {
    return res.redirect('/login');
  }
  next();
}

app.use(requireLogin);
```

Un **middleware** es una función que se ejecuta durante el procesamiento de la petición. Aquí decide si el usuario puede continuar.

Ahora organiza tu archivo:

1. Localiza la línea `app.use(express.static(path.join(__dirname, 'public')));` que utilizas desde las prácticas anteriores.
2. **Muévela debajo de `app.use(requireLogin);`**. Debe quedar una sola vez. De esta manera, abrir archivos como `/index.html` también requiere una sesión.
3. Coloca después de esa comprobación todas tus rutas de pacientes, médicos y página principal. **Mueve los bloques completos; conserva su contenido y no crees copias.**

Verifica que estas rutas queden protegidas:

| Método | Ruta | Función que ya desarrollaste |
| --- | --- | --- |
| GET | `/` | Mostrar la página principal |
| POST | `/submit-data` | Guardar pacientes |
| GET | `/pacientes` | Consultar pacientes |
| GET | `/buscar-pacientes` | Buscar con filtros |
| GET | `/ordenar-pacientes` | Ordenar por frecuencia cardiaca |
| POST | `/insertar-medico` | Guardar médicos |

Como todas pasan por `app.use(requireLogin)`, no necesitas repetir el middleware en cada ruta.

**Importante:** las rutas públicas de los pasos 5, 6 y 7 deben quedar **antes** de esta comprobación. Si colocas el inicio de sesión detrás de ella, el usuario no podrá llegar al formulario para autenticarse.

### Orden de los bloques en `server.js`

Esta tabla es una guía para organizar tu código:

| Orden | Bloque |
| --- | --- |
| 1 | Importaciones, creación de `app` y conexión a MySQL existentes |
| 2 | Procesamiento de formularios |
| 3 | Configuración de sesiones y caché |
| 4 | Rutas públicas de CSS, registro e inicio de sesión |
| 5 | Función `requireLogin` y `app.use(requireLogin)` |
| 6 | `express.static(...)` y rutas existentes de pacientes y médicos |
| 7 | Ruta de cierre de sesión |
| 8 | Manejador de errores |
| 9 | Tu único `app.listen(...)` |

Express procesa los manejadores en el orden de declaración. Una ruta que ya envió una respuesta no pasa automáticamente al siguiente manejador.

## Paso 9. Incorporar el cierre de sesión

Después de las rutas protegidas, agrega:

```javascript
app.post('/logout', (req, res, next) => {
  req.session.destroy(err => {
    if (err) return next(err);

    res.clearCookie('connect.sid', { path: '/' });
    res.redirect('/login');
  });
});
```

`destroy()` elimina la sesión del servidor; después se borra la cookie y se regresa al formulario.

En tu `public/index.html`, agrega este formulario cerca del encabezado, **fuera de los demás formularios**:

```html
<form action="/logout" method="POST">
  <button type="submit">Cerrar sesión</button>
</form>
```

Conserva tus formularios de pacientes y médicos, los botones de consulta y tus estilos.

## Paso 10. Manejar los errores

Después de todas las rutas y antes de `app.listen(...)`, agrega:

```javascript
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send('Ocurrió un error en el servidor. Revisa la terminal de Node.js.');
});
```

Este middleware recibe cuatro argumentos. Los errores enviados con `next(err)` llegarán aquí y se mostrarán en la terminal.

## Paso 11. Revisar las consultas del sistema

El control de sesión limita quién puede acceder, pero también debes cuidar cómo utilizas los datos enviados por los formularios.

En la ruta `/buscar-pacientes` de tu práctica 6:

1. Conserva la consulta inicial y crea un arreglo vacío para los valores.
2. Para el filtro de nombre, agrega `AND nombre LIKE ?` a la consulta y guarda el texto de búsqueda entre signos `%` en el arreglo.
3. Para el filtro de edad, valida que sea un entero no negativo, agrega `AND edad = ?` y guarda la edad en el arreglo.
4. Pasa el arreglo como segundo argumento de `connection.query`. Conserva el callback que construye tu tabla HTML.

Por ejemplo, el filtro de nombre se construye así:

```javascript
query += ' AND nombre LIKE ?';
valores.push('%' + name_search + '%');
```

Adapta la llamada existente para que tenga la forma `connection.query(query, valores, callback)`, utilizando **tu función actual** como callback. No concatenes directamente las entradas dentro del SQL.

Para mostrar textos de la base de datos en tu tabla HTML, agrega esta función auxiliar:

```javascript
function escapeHtml(value) {
  return String(value ?? '').replace(/[&<>"']/g, caracter => ({
    '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
  }[caracter]));
}
```

En los textos que insertas en HTML, utiliza, por ejemplo, `escapeHtml(paciente.nombre)` en lugar de `paciente.nombre`. Aplica el mismo criterio a los mensajes que muestran nombres introducidos en formularios. Esto permite mostrar los caracteres como texto.

## Paso 12. Probar el sistema

Guarda los archivos, inicia el servidor con `node server.js` y abre **http://localhost:3000**. Utiliza ese servidor; no abras los HTML con doble clic ni mediante Live Server.

| Prueba | Resultado esperado |
| --- | --- |
| Abrir `/` en una ventana privada | Mostrar el inicio de sesión |
| Ir al registro y crear una cuenta | Guardar el usuario y regresar al inicio de sesión |
| Registrar el mismo nombre otra vez | Informar que el nombre ya existe |
| Iniciar sesión con una contraseña equivocada | Rechazar el acceso |
| Iniciar sesión con los datos correctos | Mostrar tu página principal |
| Registrar, consultar, buscar y ordenar pacientes | Mantener las funciones de la práctica 6 |
| Buscar un nombre con apóstrofo, como `O'Connor` | Ejecutar la consulta sin errores de sintaxis SQL |
| Registrar un médico | Guardar los datos en la tabla `medicos` |
| Cerrar sesión y abrir `/index.html` | Redirigir a `/login` |
| Sin sesión, abrir las rutas de consulta | Redirigir a `/login` |
| Reiniciar Node.js | Solicitar inicio de sesión nuevamente |

Comprueba también que las inserciones estén protegidas. Desde otra terminal, sin enviar la cookie del navegador, ejecuta:

```bash
curl -i -X POST http://localhost:3000/submit-data -d "name=PruebaSinSesion&age=20&heart_rate=70"
curl -i -X POST http://localhost:3000/insertar-medico -d "medico_name=PruebaSinSesion&especialidad=General"
```

En PowerShell puedes usar `curl.exe`. Ambas peticiones deben responder con una redirección **302** a `/login` y no insertar registros. Escribir la dirección en el navegador envía GET; no prueba una ruta POST.

Si algo no funciona, revisa el orden de los bloques, la coincidencia de los nombres de campos y que no haya rutas duplicadas. Reinicia Node.js después de modificar `server.js`.

## Evidencias y adecuaciones

Entrega:

1. Capturas del registro, inicio y cierre de sesión.
2. Evidencia de que una ruta protegida no permite entrar sin sesión.
3. Evidencia de que las funciones de pacientes y médicos siguen trabajando.
4. Una explicación breve: ¿qué diferencia hay entre GET y POST?, ¿qué guarda la sesión?, ¿por qué no basta con ocultar un botón?

Personaliza los formularios con tu hoja de estilos. Trabaja con datos ficticios: todos los usuarios autenticados tendrán el mismo acceso en este ejercicio. La configuración de sesiones en memoria es para la práctica local; un despliegue requiere HTTPS y un almacén persistente de sesiones.
