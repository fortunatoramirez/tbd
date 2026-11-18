# Práctica 9. Búsqueda en vivo e inventario de equipos con Excel

En la [práctica 8](tbd-p8-v2.md) cada usuario entraba al sistema de gestión de pacientes y veía opciones según su rol. Ahora el administrador necesita localizar cuentas mientras escribe y recibir el inventario de equipos biomédicos que el laboratorio registra en Excel. Al terminar podrá importar varios equipos en una operación, descargar el inventario y consultar la hora local de la última importación.

Trabaja con **cuentas y equipos ficticios**. Los fragmentos amplían el proyecto existente. Conserva sus rutas, sesiones, formularios y permisos.

## 1. Preparar la copia y probar el reinicio automático

1. Detén el servidor con **Ctrl+C**, copia tu proyecto de la práctica 8 a `gestion-pacientes-p9` y abre esa copia en VS Code. La copia sigue conectada a la misma base `gestion_pacientes`. Exporta un respaldo desde Workbench.
2. En la terminal de la copia ejecuta estos comandos. ExcelJS leerá y escribirá libros, Multer recibirá el archivo del formulario y dotenv cargará la configuración local.

   ```bash
   npm install
   npm install dotenv multer exceljs
   npm install --save-dev nodemon
   npm pkg set "scripts.dev=nodemon server.js"
   npm run dev
   ```

3. Abre `http://localhost:3000`, inicia sesión como administrador y visita `/ver-usuarios`. Identifica un valor de `nombre_usuario` para probar la búsqueda después.
4. Agrega un comentario a `server.js` y guarda. Observa el reinicio en la terminal. Si tu aplicación guarda las sesiones en memoria, vuelve a iniciar sesión después de cada reinicio. Los cambios en HTML y CSS se ven al actualizar el navegador.

## 2. Preparar MySQL y conectar con un usuario de la aplicación

En Workbench, con tu cuenta de administración de MySQL, ejecuta:

```sql
USE gestion_pacientes;
CREATE TABLE IF NOT EXISTS equipos (
  clave VARCHAR(30) PRIMARY KEY,
  nombre VARCHAR(100) NOT NULL,
  descripcion VARCHAR(200) NOT NULL,
  importado_utc DATETIME NOT NULL
) ENGINE=InnoDB;
DESCRIBE equipos;
```

La **clave de inventario** distingue dos equipos aunque tengan el mismo nombre. InnoDB permite que la importación se guarde completa o se cancele completa.

Crea una cuenta local de MySQL para el servidor. Sustituye el texto de la contraseña por una propia antes de ejecutar el bloque. Si ya creaste la cuenta, continúa desde los permisos usando su contraseña.

```sql
CREATE USER 'app_p9'@'localhost' IDENTIFIED BY 'CAMBIA_ESTA_CLAVE';
GRANT SELECT, INSERT ON gestion_pacientes.usuarios TO 'app_p9'@'localhost';
GRANT SELECT ON gestion_pacientes.codigos_acceso TO 'app_p9'@'localhost';
GRANT SELECT, INSERT ON gestion_pacientes.pacientes TO 'app_p9'@'localhost';
GRANT SELECT, INSERT ON gestion_pacientes.medicos TO 'app_p9'@'localhost';
GRANT SELECT, INSERT ON gestion_pacientes.equipos TO 'app_p9'@'localhost';
SHOW GRANTS FOR 'app_p9'@'localhost';
```

El servidor usa `app_p9` para hablar con MySQL. Los usuarios seguirán iniciando sesión en la página con las cuentas de la tabla `usuarios`. Si adaptaste prácticas anteriores con operaciones adicionales, revisa los permisos que requieren esas rutas.

En la carpeta de `server.js`, agrega estas variables a `.env`. Conserva las variables que ya utilizabas y escribe la contraseña que elegiste entre comillas.

```dotenv
DB_HOST=localhost
DB_USER=app_p9
DB_PASSWORD="CAMBIA_ESTA_CLAVE"
DB_NAME=gestion_pacientes
```

Incluye `.env` y `node_modules/` en `.gitignore`. En `server.js`, carga la configuración antes de crear la conexión. Conserva la importación existente de `mysql2` y sustituye **solo las opciones de tu conexión** por las que se muestran aquí. Sigue usando el mismo objeto `connection` en las rutas anteriores.

```js
require('dotenv').config();
const ExcelJS = require('exceljs');
const multer = require('multer');

const connection = mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});
const db = connection.promise();
```

`db` permite usar `await` con esa misma conexión. Guarda, reinicia y verifica que todavía puedas iniciar sesión y consultar pacientes.

## 3. Buscar una cuenta desde el navegador

Agrega **todas las rutas nuevas** de esta guía en `server.js`, después de `app.use(requireLogin)` y antes del manejador de errores y de `app.listen(...)`. Conserva `GET /ver-usuarios`. La nueva ruta devuelve hasta 20 coincidencias para el texto escrito.

```js
app.get('/api/cuentas', requireRole('admin'), async (req, res, next) => {
  const texto = String(req.query.q || '').trim();
  if (!texto) return res.json([]);
  try {
    const [filas] = await db.execute(
      `SELECT nombre_usuario, tipo_usuario FROM usuarios
       WHERE LOCATE(?, nombre_usuario) > 0 ORDER BY nombre_usuario LIMIT 20`,
      [texto]
    );
    res.json(filas);
  } catch (error) {
    next(error);
  }
});
```

**Comprueba primero la consulta.** Como administrador abre `/api/cuentas?q=ad` y sustituye `ad` por una parte de tu cuenta de prueba. Verás su nombre y rol. Con texto sin coincidencias verás `[]`. El signo `?` en el SQL recibe el valor de `[texto]`.

## 4. Reutilizar el menú y conectar la búsqueda en vivo

1. En `public/navegacion.js`, localiza las tres instrucciones de la práctica 8 que hacen visible un panel. La página nueva compartirá el menú pero tendrá otros elementos. Para cada panel, comprueba que exista antes de mostrarlo. Por ejemplo, dentro del bloque del rol correspondiente:

   ```js
   const panel = document.getElementById('panel-pacientes');
   if (panel) panel.hidden = false;
   ```

   Repite el cambio para `panel-medicos` y `panel-personal` en sus respectivos bloques. Dentro de `if (rol === 'admin')` agrega `agregarEnlace('Cuentas e inventario', '/inventario.html');`. Conserva los demás enlaces.

2. Crea `public/inventario.html`:

   ```html
   <!doctype html>
   <html lang="es">
   <head>
     <meta charset="utf-8">
     <meta name="viewport" content="width=device-width, initial-scale=1">
     <title>Cuentas e inventario</title>
     <link rel="stylesheet" href="/styles.css">
     <script defer src="/navegacion.js"></script>
     <script defer src="/inventario.js"></script>
   </head>
   <body>
     <div id="navbar"></div>
     <p id="estado-sesion" role="status"></p>
     <main>
       <h1>Cuentas e inventario del laboratorio</h1>
       <label for="consulta">Buscar por nombre de usuario</label>
       <input id="consulta" type="search" autocomplete="off">
       <p id="aviso" role="status">Escribe parte del nombre.</p>
       <ul id="resultados"></ul>
       <!-- Aquí agregarás el formulario de Excel en el paso 6. -->
     </main>
   </body>
   </html>
   ```

3. Para mantener visible la barra al desplazarte, agrega a `public/styles.css`:

   ```css
   #navbar { position: sticky; top: 0; z-index: 1; }
   ```

4. Crea `public/inventario.js`. El evento `input` detecta escritura, pegado y borrado. El contador permite descartar respuestas que llegan cuando el usuario ya cambió el texto.

   ```js
   const consulta = document.getElementById('consulta');
   const resultados = document.getElementById('resultados');
   const aviso = document.getElementById('aviso');
   let version = 0;

   consulta.addEventListener('input', async () => {
     const actual = ++version;
     const texto = consulta.value.trim();
     resultados.replaceChildren();
     if (!texto) {
       aviso.textContent = 'Escribe parte del nombre.';
       return;
     }
     aviso.textContent = 'Buscando…';
     try {
       const parametros = new URLSearchParams({ q: texto });
       const respuesta = await fetch('/api/cuentas?' + parametros);
       if (actual !== version) return;
       if (respuesta.redirected) return location.assign('/login');
       if (!respuesta.ok) throw new Error('No se pudo consultar. Revisa tu sesión y rol.');
       const cuentas = await respuesta.json();
       if (actual !== version) return;
       for (const cuenta of cuentas) {
         const fila = document.createElement('li');
         fila.textContent = cuenta.nombre_usuario + ' (' + cuenta.tipo_usuario + ')';
         resultados.append(fila);
       }
       aviso.textContent = cuentas.length ? 'Coincidencias mostradas: ' + cuentas.length : 'Sin coincidencias.';
     } catch (error) {
       if (actual === version) aviso.textContent = error.message;
     }
   });
   ```

**Comprueba la interfaz.** Desde Inicio entra a **Cuentas e inventario**, escribe dos letras, pega un nombre y vacía el campo. Los resultados deben cambiar sin presionar Enter. Usa el enlace **Inicio** y verifica que tus paneles anteriores sigan apareciendo.

## 5. Descargar primero la plantilla de Excel

El administrador descargará el inventario actual. Si aún está vacío, el mismo archivo sirve de plantilla. Agrega esta ruta:

```js
app.get('/api/equipos/excel', requireRole('admin'), async (req, res, next) => {
  try {
    const [filas] = await db.execute('SELECT clave, nombre, descripcion FROM equipos ORDER BY clave');
    const libro = new ExcelJS.Workbook();
    const hoja = libro.addWorksheet('Equipos');
    hoja.columns = [
      { header: 'clave', key: 'clave', width: 18 },
      { header: 'nombre', key: 'nombre', width: 28 },
      { header: 'descripcion', key: 'descripcion', width: 45 }
    ];
    hoja.addRows(filas);
    const archivo = await libro.xlsx.writeBuffer();
    res.attachment('equipos.xlsx').send(Buffer.from(archivo));
  } catch (error) {
    next(error);
  }
});
```

Como administrador, visita `/api/equipos/excel`. Abre el archivo y comprueba los tres encabezados. Para preparar una **carga de altas**, conserva los encabezados y deja únicamente filas con claves nuevas. Usa texto simple en las tres columnas, sin fórmulas. Guarda como `altas.xlsx`.

| clave | nombre | descripcion |
| --- | --- | --- |
| EQ-001 | Oxímetro | Equipo de demostración |
| EQ-002 | Monitor | Simulador del laboratorio |

## 6. Importar y comprobar antes de guardar

En `inventario.html`, coloca este bloque donde dejaste el comentario:

```html
<section>
  <h2>Inventario de equipos</h2>
  <a href="/api/equipos/excel">Descargar inventario o plantilla</a>
  <form action="/api/equipos/excel" method="post" enctype="multipart/form-data">
    <label for="archivo">Altas de equipos (.xlsx, máximo 1 MB)</label>
    <input id="archivo" type="file" name="archivo" accept=".xlsx" required>
    <button>Importar altas</button>
  </form>
</section>
```

En `server.js`, agrega el lector antes de la ruta de importación. Lee la primera hoja y admite hasta 200 filas de datos. Una fila vacía se omite. Una fila incompleta se reporta con el número que el alumno ve en Excel.

```js
async function leerEquipos(buffer) {
  const libro = new ExcelJS.Workbook();
  try {
    await libro.xlsx.load(buffer);
  } catch {
    throw new Error('Abre el archivo en Excel y guárdalo como libro .xlsx.');
  }
  const hoja = libro.worksheets[0];
  const campos = ['clave', 'nombre', 'descripcion'];
  if (!hoja || hoja.columnCount !== 3 ||
      campos.some((campo, i) => hoja.getCell(1, i + 1).value !== campo)) {
    throw new Error('Usa la primera hoja con los encabezados clave, nombre y descripcion, en ese orden.');
  }
  if (hoja.rowCount > 201) throw new Error('Usa hasta 200 filas debajo del encabezado.');
  const equipos = [];
  const limites = [30, 100, 200];
  hoja.eachRow((fila, numero) => {
    if (numero === 1) return;
    const valores = campos.map((campo, i) => {
      const valor = fila.getCell(i + 1).value;
      if (typeof valor !== 'string' || !valor.trim() || valor.trim().length > limites[i]) {
        throw new Error('Fila ' + numero + ': revisa ' + campo + ' (texto de 1 a ' + limites[i] + ' caracteres).');
      }
      return valor.trim();
    });
    equipos.push(valores);
  });
  if (!equipos.length) throw new Error('Agrega al menos un equipo.');
  return equipos;
}
```

Agrega ahora la ruta POST. El nombre `archivo` debe coincidir con el atributo `name` del formulario. Se usa un solo INSERT con parámetros para todo el lote. Si una clave ya existe o se repite dentro del libro, InnoDB rechaza esa operación completa.

```js
const recibirArchivo = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 1024 * 1024, files: 1 }
}).single('archivo');

app.post('/api/equipos/excel', requireRole('admin'), (req, res, next) => {
  recibirArchivo(req, res, async (error) => {
    if (error || !req.file) return res.status(400).send('Selecciona un solo libro .xlsx de hasta 1 MB.');
    let filas;
    try {
      filas = await leerEquipos(req.file.buffer);
    } catch (errorLectura) {
      return res.status(400).type('text').send(errorLectura.message);
    }
    try {
      const grupos = filas.map(() => '(?, ?, ?, UTC_TIMESTAMP())').join(', ');
      await db.execute(
        'INSERT INTO equipos (clave, nombre, descripcion, importado_utc) VALUES ' + grupos,
        filas.flat()
      );
      res.type('text').send('Equipos guardados: ' + filas.length + '. Vuelve al inventario para descargarlos.');
    } catch (errorBD) {
      if (errorBD.code === 'ER_DUP_ENTRY') {
        return res.status(409).send('Hay una clave repetida. No se guardó ninguna fila de este archivo.');
      }
      next(errorBD);
    }
  });
});
```

Importa `altas.xlsx`. Al recibir la confirmación, vuelve con **Atrás**, descarga el inventario y compara las claves. Si envías otra vez el mismo libro, la cantidad de equipos debe permanecer igual.

## 7. Mostrar la hora de Tijuana

MySQL registró cada alta con `UTC_TIMESTAMP()`. Mantén ese valor en UTC y conviértelo al presentarlo. Así el navegador aplica el horario de Tijuana correspondiente a la fecha, aunque la computadora esté configurada en otra zona.

Agrega a `server.js`:

```js
app.get('/api/equipos/ultima-importacion', requireRole('admin'), async (req, res, next) => {
  try {
    const [filas] = await db.execute(
      "SELECT DATE_FORMAT(MAX(importado_utc), '%Y-%m-%dT%H:%i:%sZ') AS utc FROM equipos"
    );
    res.json(filas[0]);
  } catch (error) {
    next(error);
  }
});
```

Debajo del formulario agrega `<p id="ultima-importacion" role="status"></p>`. Al final de `inventario.js` agrega:

```js
async function mostrarUltimaImportacion() {
  const salida = document.getElementById('ultima-importacion');
  try {
    const respuesta = await fetch('/api/equipos/ultima-importacion');
    if (respuesta.redirected) return location.assign('/login');
    if (!respuesta.ok) throw new Error('No se pudo consultar la fecha.');
    const { utc } = await respuesta.json();
    const formato = new Intl.DateTimeFormat('es-MX', {
      timeZone: 'America/Tijuana', dateStyle: 'medium', timeStyle: 'short'
    });
    salida.textContent = utc ? 'Última importación (Tijuana): ' + formato.format(new Date(utc))
                             : 'Aún no hay equipos importados.';
  } catch (error) {
    salida.textContent = error.message;
  }
}
window.addEventListener('pageshow', mostrarUltimaImportacion);
```

Actualiza el inventario y compara la hora que muestra con la de Tijuana al realizar la carga. En Workbench puedes ver el dato original con `SELECT clave, importado_utc FROM equipos ORDER BY clave;`.

## 8. Demostrar que funciona

| Prueba | Resultado esperado |
| --- | --- |
| Buscar una cuenta existente, una inexistente y vaciar el campo | Nombre y rol, mensaje sin coincidencias e indicación inicial, respectivamente |
| Importar dos claves nuevas y descargar | Confirmación de dos altas y ambas filas en el libro descargado |
| Enviar una clave nueva junto con otra ya registrada | Mensaje de clave repetida. La clave nueva tampoco aparece al descargar |
| Dejar vacía la descripción de la fila 3 | El mensaje señala la fila y la columna. El inventario conserva su contenido anterior |
| Entrar como médico y como paciente | El menú conserva sus opciones de la práctica 8. Al abrir directamente la descarga o la búsqueda de cuentas, el servidor devuelve 403 |
| Cerrar sesión y abrir la descarga | El sistema dirige al inicio de sesión |

**Entrega:** proyecto con sus archivos de dependencias, un libro de altas ficticias y capturas de la búsqueda, la descarga posterior a la importación y el rechazo de un lote mixto. Omite `.env` y `node_modules`. Explica con el ejemplo de tu lote mixto por qué la confirmación se envía después de que MySQL termina el INSERT.
