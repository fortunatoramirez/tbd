### Práctica 4: Interfaz Web y Conexión con MySQL para Gestión de Pacientes

#### Objetivo:
Desarrollar una aplicación web básica con Node.js y MySQL que permita a los estudiantes conectarse a una base de datos, realizar consultas y almacenar información de pacientes desde una interfaz web.

---

### Introducción:
En esta práctica aprenderás a conectar una página web a una base de datos MySQL a través de un servidor Node.js utilizando el framework Express. Vamos a construir una aplicación paso a paso que te permitirá ingresar y consultar información de pacientes y sus signos vitales.

---

### Requisitos:
- Tener instalado Node.js y MySQL.
- Tener un editor de código (VSCode recomendado).
- Instalar las dependencias necesarias (`express`, `mysql2`, `body-parser`).

---

### Paso 1: Crear el servidor y la página web básica

1. **Crea la estructura de archivos**:

   - Crea una carpeta para tu proyecto llamada `gestion-pacientes`.
   - Dentro de esta carpeta, crea un archivo llamado `server.js`.
   - Crea otra carpeta llamada `public` y dentro de ella, un archivo llamado `index.html`.

2. **Instala Express y otras dependencias**:
   Abre una terminal en la carpeta del proyecto y ejecuta:
   ```bash
   npm init -y
   npm install express mysql2 body-parser
   ```

3. **Escribe el código básico del servidor**:

   En `server.js`, agrega el siguiente código para crear el servidor Express y servir la página HTML básica:

   ```javascript
   const express = require('express');
   const app = express();
   const path = require('path');
   const bodyParser = require('body-parser');

   app.use(bodyParser.urlencoded({ extended: true }));

   // Servir archivos estáticos (HTML)
   app.use(express.static(path.join(__dirname, 'public')));

   // Ruta para la página principal
   app.get('/', (req, res) => {
     res.sendFile(path.join(__dirname, 'public', 'index.html'));
   });

   // Iniciar el servidor
   app.listen(3000, () => {
     console.log('Servidor corriendo en http://localhost:3000');
   });
   ```

4. **Escribe el HTML básico**:

   En `public/index.html`, agrega lo siguiente:

   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Gestión de Pacientes</title>
   </head>
   <body>
     <h1>Bienvenido a la Gestión de Pacientes</h1>
     <p>Esta es una página básica para administrar pacientes y sus signos vitales.</p>
   </body>
   </html>
   ```

5. **Ejecuta el servidor y prueba**:

   - En la terminal, ejecuta:
     ```bash
     node server.js
     ```
   - Abre un navegador y ve a `http://localhost:3000`. Deberías ver la página con el mensaje de bienvenida.

---

### Paso 2: Agregar un formulario a la página y procesarlo en el servidor

1. **Modificar el HTML para incluir un formulario**:
   
   En `public/index.html`, agrega un formulario debajo del texto de bienvenida para registrar a un paciente:
   
   ```html
   <form action="/submit-data" method="POST">
     <label for="name">Nombre del paciente:</label>
     <input type="text" id="name" name="name">
     
     <label for="age">Edad:</label>
     <input type="number" id="age" name="age">

     <label for="heart-rate">Frecuencia Cardiaca (bpm):</label>
     <input type="number" id="heart-rate" name="heart_rate">

     <button type="submit">Guardar</button>
   </form>
   ```

2. **Agregar la ruta en el servidor para procesar el formulario**:

   En `server.js`, agrega la ruta para recibir los datos del formulario:

   ```javascript
   // Ruta para procesar el formulario
   app.post('/submit-data', (req, res) => {
     const { name, age, heart_rate } = req.body;
     res.send(`Paciente recibido: ${name}, Edad: ${age}, Frecuencia Cardiaca: ${heart_rate}`);
   });
   ```

3. **Prueba el formulario**:

   - Recarga la página en el navegador.
   - Introduce los datos de un paciente y envíalos. Deberías ver un mensaje que muestre los datos ingresados.

---

### Paso 3: Conectar el servidor a MySQL

1. **Crear una base de datos y tabla en MySQL**:

   Abre MySQL desde la terminal y ejecuta los siguientes comandos para crear la base de datos y la tabla de pacientes:

   ```sql
   CREATE DATABASE gestion_pacientes;
   USE gestion_pacientes;
   CREATE TABLE pacientes (
     id INT AUTO_INCREMENT PRIMARY KEY,
     nombre VARCHAR(100),
     edad INT,
     frecuencia_cardiaca INT
   );
   ```

2. **Agregar la conexión a MySQL en el servidor**:

   Modifica `server.js` para conectarse a MySQL y agregar los datos del paciente a la base de datos:

   ```javascript
   const mysql = require('mysql2');

   // Configurar conexión a MySQL
   const connection = mysql.createConnection({
     host: 'localhost',
     user: 'root',
     password: 'password', // Cambia por tu contraseña
     database: 'gestion_pacientes'
   });

   connection.connect(err => {
     if (err) {
       console.error('Error conectando a MySQL:', err);
       return;
     }
     console.log('Conexión exitosa a MySQL');
   });

   // Ruta para guardar datos en la base de datos
   app.post('/submit-data', (req, res) => {
     const { name, age, heart_rate } = req.body;

     const query = 'INSERT INTO pacientes (nombre, edad, frecuencia_cardiaca) VALUES (?, ?, ?)';
     connection.query(query, [name, age, heart_rate], (err, result) => {
       if (err) {
         return res.send('Error al guardar los datos en la base de datos.');
       }
       res.send(`Paciente ${name} guardado en la base de datos.`);
     });
   });
   ```

3. **Prueba la conexión**:

   - Vuelve a iniciar el servidor con `node server.js`.
   - Completa el formulario en la página y verifica que los datos del paciente se guarden en la base de datos.

---

### Paso 4: Consultar los datos desde la base de datos

1. **Agregar una nueva ruta para consultar los datos**:

   En `server.js`, agrega la siguiente ruta para obtener la lista de pacientes guardados:

   ```javascript
   // Ruta para mostrar los datos de la base de datos
   app.get('/pacientes', (req, res) => {
     connection.query('SELECT * FROM pacientes', (err, results) => {
       if (err) {
         return res.send('Error al obtener los datos.');
       }
       res.send(results);
     });
   });
   ```

2. **Prueba la consulta**:

   - Abre tu navegador y ve a `http://localhost:3000/pacientes`. Deberías ver una lista de los pacientes guardados con su nombre, edad y frecuencia cardíaca.

---

### Paso 5: Guardar y consultar datos desde la página

1. **Agregar un botón en el HTML para ver los datos guardados**:

   En `public/index.html`, agrega este botón al final:

   ```html
   <button onclick="window.location.href='/pacientes'">Ver Pacientes Guardados</button>
   ```

2. **Prueba la funcionalidad completa**:

   Ahora deberías poder:
   - Enviar datos de pacientes desde el formulario y guardarlos en la base de datos.
   - Consultar los pacientes guardados usando el botón "Ver Pacientes Guardados".

---

