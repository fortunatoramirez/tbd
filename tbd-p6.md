### Práctica 6: Mejoras en la Interfaz Web y Consultas SQL Avanzadas

#### Objetivo:
Desarrollar una interfaz web más interactiva para la gestión de pacientes, integrando consultas SQL más avanzadas y mostrando los resultados en una tabla HTML estilizada. Los estudiantes podrán realizar operaciones de consulta específicas y manipular la forma en que los datos se muestran.

---

### Introducción:
En esta práctica, continuarás mejorando la aplicación web desarrollada en las prácticas anteriores. Ahora aprenderás a realizar consultas más complejas sobre los datos almacenados en la base de datos, además de estilizar los resultados de manera personalizada usando solo CSS. Se espera que los estudiantes dominen el uso de `SELECT` con filtros (`WHERE`), ordenación de resultados (`ORDER BY`), y más consultas SQL avanzadas vistas en las prácticas anteriores.

---

### Requisitos:
- Haber completado la Práctica 5.
- Tener el entorno de Node.js y MySQL listo y funcionando con los archivos creados previamente en la carpeta `gestion-pacientes`.
- Haber instalado las dependencias necesarias (`express`, `mysql2`, `body-parser`).

---

### Paso 1: Modificar el formulario para incluir un filtro de consulta

1. **Agregar campos de búsqueda en el HTML**:

   Vamos a añadir un pequeño formulario en `public/index.html` para que los usuarios puedan buscar pacientes por nombre y edad. Modifica el archivo para incluir los nuevos campos justo debajo del formulario actual:

   ```html
   <h2>Buscar Pacientes</h2>
   <form action="/buscar-pacientes" method="GET">
     <label for="name-search">Nombre del paciente:</label>
     <input type="text" id="name-search" name="name_search">
     
     <label for="age-search">Edad del paciente:</label>
     <input type="number" id="age-search" name="age_search">
     
     <button type="submit">Buscar</button>
   </form>
   ```

---

### Paso 2: Implementar la lógica de búsqueda en el servidor

1. **Agregar una nueva ruta para manejar las búsquedas**:

   En `server.js`, agrega una ruta que procesará la búsqueda en la base de datos según los filtros de nombre y edad introducidos. Usa el siguiente código para manejar la consulta:

   ```javascript
   // Ruta para buscar pacientes según filtros
   app.get('/buscar-pacientes', (req, res) => {
     const { name_search, age_search } = req.query;
     let query = 'SELECT * FROM pacientes WHERE 1=1';

     if (name_search) {
       query += ` AND nombre LIKE '%${name_search}%'`;
     }
     if (age_search) {
       query += ` AND edad = ${age_search}`;
     }

     connection.query(query, (err, results) => {
       if (err) {
         return res.send('Error al obtener los datos.');
       }

       let html = `
         <html>
         <head>
           <link rel="stylesheet" href="/styles.css">
           <title>Resultados de Búsqueda</title>
         </head>
         <body>
           <h1>Resultados de Búsqueda</h1>
           <table>
             <thead>
               <tr>
                 <th>Nombre</th>
                 <th>Edad</th>
                 <th>Frecuencia Cardiaca (bpm)</th>
               </tr>
             </thead>
             <tbody>
       `;

       results.forEach(paciente => {
         html += `
           <tr>
             <td>${paciente.nombre}</td>
             <td>${paciente.edad}</td>
             <td>${paciente.frecuencia_cardiaca}</td>
           </tr>
         `;
       });

       html += `
             </tbody>
           </table>
           <button onclick="window.location.href='/'">Volver</button>
         </body>
         </html>
       `;

       res.send(html);
     });
   });
   ```

   Con esta lógica, los usuarios pueden buscar pacientes por nombre o edad (o ambos). Si no se ingresa ningún filtro, se mostrarán todos los pacientes.

---

### Paso 3: Añadir consultas avanzadas y mostrar los resultados

1. **Agregar un botón para ordenar por frecuencia cardíaca**:

   Vamos a añadir un botón en el HTML para que los usuarios puedan ordenar la lista de pacientes por su frecuencia cardíaca. Modifica el archivo `public/index.html` para incluir este botón:

   ```html
   <button onclick="window.location.href='/ordenar-pacientes'">Ordenar por Frecuencia Cardiaca</button>
   ```

2. **Implementar la ruta para ordenar por frecuencia cardíaca**:

   En `server.js`, añade la lógica para ordenar los resultados de pacientes por su frecuencia cardíaca:

   ```javascript
   // Ruta para ordenar pacientes por frecuencia cardiaca
   app.get('/ordenar-pacientes', (req, res) => {
     const query = 'SELECT * FROM pacientes ORDER BY frecuencia_cardiaca DESC';

     connection.query(query, (err, results) => {
       if (err) {
         return res.send('Error al obtener los datos.');
       }

       let html = `
         <html>
         <head>
           <link rel="stylesheet" href="/styles.css">
           <title>Pacientes Ordenados</title>
         </head>
         <body>
           <h1>Pacientes Ordenados por Frecuencia Cardiaca</h1>
           <table>
             <thead>
               <tr>
                 <th>Nombre</th>
                 <th>Edad</th>
                 <th>Frecuencia Cardiaca (bpm)</th>
               </tr>
             </thead>
             <tbody>
       `;

       results.forEach(paciente => {
         html += `
           <tr>
             <td>${paciente.nombre}</td>
             <td>${paciente.edad}</td>
             <td>${paciente.frecuencia_cardiaca}</td>
           </tr>
         `;
       });

       html += `
             </tbody>
           </table>
           <button onclick="window.location.href='/'">Volver</button>
         </body>
         </html>
       `;

       res.send(html);
     });
   });
   ```

---

### Paso 4: Insertar médicos en la base de datos

En esta práctica, también integraremos la inserción de **médicos** en la base de datos.

1. **Crear una nueva tabla de médicos** en MySQL:

   En la terminal de MySQL, crea la tabla `medicos`:

   ```sql
   CREATE TABLE medicos (
     id INT AUTO_INCREMENT PRIMARY KEY,
     nombre VARCHAR(100),
     especialidad VARCHAR(100)
   );
   ```

2. **Modificar el HTML para permitir la inserción de médicos**:

   En `public/index.html`, agrega un formulario para insertar médicos:

   ```html
   <h2>Registrar Médico</h2>
   <form action="/insertar-medico" method="POST">
     <label for="medico-name">Nombre del médico:</label>
     <input type="text" id="medico-name" name="medico_name">

     <label for="especialidad">Especialidad:</label>
     <input type="text" id="especialidad" name="especialidad">

     <button type="submit">Guardar Médico</button>
   </form>
   ```

3. **Agregar la lógica en el servidor para insertar médicos**:

   En `server.js`, añade la lógica para insertar médicos en la base de datos:

   ```javascript
   // Ruta para insertar un nuevo médico
   app.post('/insertar-medico', (req, res) => {
     const { medico_name, especialidad } = req.body;
     const query = 'INSERT INTO medicos (nombre, especialidad) VALUES (?, ?)';

     connection.query(query, [medico_name, especialidad], (err, result) => {
       if (err) {
         return res.send('Error al insertar el médico.');
       }
       res.send(`Médico ${medico_name} guardado exitosamente.`);
     });
   });
   ```

---

### Paso 5: Estilizar los resultados con CSS

Como en la Práctica 5, asegúrate de que los resultados mostrados en tablas estén bien estilizados. Puedes reutilizar o mejorar los estilos que ya tienes en `styles.css`.

---

### Paso 6: Probar y finalizar

1. **Ejecuta el servidor** con `node server.js`.
2. **Prueba** las nuevas funcionalidades:
   - Busca pacientes por nombre y edad.
   - Ordena los pacientes por su frecuencia cardíaca.
   - Inserta médicos en la base de datos y verifica que se guarden correctamente.

---
