### Práctica 5: Estilización con CSS y Visualización de Datos en una Tabla HTML

#### Objetivo:
Aprender a aplicar estilos básicos a una página web usando CSS y visualizar los datos de una base de datos en una tabla HTML estilizada.

---

### Introducción:
En esta práctica, aprenderás a mejorar la apariencia de tu página web utilizando hojas de estilo en cascada (CSS) y a mostrar los datos consultados de la base de datos en una tabla HTML con un diseño más atractivo. Se te guiará paso a paso para aplicar estilos y crear una tabla que muestre la información de los pacientes de manera clara y visualmente agradable.

---

### Requisitos:
- Práctica 4 completada.
- Archivos y servidor creados en la carpeta `gestion-pacientes`.
- Tener instalados `Node.js`, `MySQL` y las dependencias (`express`, `mysql2`, `body-parser`).

---

### Paso 1: Crear un archivo CSS para la página web

1. **Crea una hoja de estilos CSS**:

   - Dentro de la carpeta `public`, crea un nuevo archivo llamado `styles.css`.
   - Este archivo será el que usemos para estilizar los elementos de la página HTML.

2. **Enlazar el archivo CSS a tu página HTML**:

   Modifica `public/index.html` para que enlace el archivo `styles.css`. Agrega la siguiente línea dentro de la etiqueta `<head>`:

   ```html
   <link rel="stylesheet" href="styles.css">
   ```

3. **Prueba la conexión con el archivo CSS**:

   Agrega algo de estilo básico en `styles.css` para asegurarte de que esté correctamente enlazado:

   ```css
   body {
     font-family: Arial, sans-serif;
     background-color: #f0f0f0;
   }

   h1 {
     color: #333;
     text-align: center;
   }
   ```

4. **Recarga la página**:

   Ejecuta el servidor con `node server.js` si no lo tienes corriendo y ve a `http://localhost:3000`. Deberías ver los cambios básicos de estilo (el color de fondo gris claro y el título centrado).

---

### Paso 2: Aplicar estilos al formulario de registro de pacientes

1. **Estiliza el formulario**:

   Agrega los siguientes estilos en `styles.css` para hacer que el formulario luzca más atractivo:

   ```css
   form {
     width: 300px;
     margin: 0 auto;
     padding: 20px;
     background-color: #fff;
     border-radius: 8px;
     box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
   }

   label {
     display: block;
     margin: 10px 0 5px;
     font-weight: bold;
   }

   input {
     width: 100%;
     padding: 8px;
     margin-bottom: 10px;
     border: 1px solid #ccc;
     border-radius: 4px;
   }

   button {
     width: 100%;
     padding: 10px;
     background-color: #28a745;
     color: white;
     border: none;
     border-radius: 4px;
     cursor: pointer;
   }

   button:hover {
     background-color: #218838;
   }
   ```

2. **Recarga la página**:

   Al recargar la página, el formulario para registrar pacientes debería verse más estilizado con un diseño centrado y limpio.

---

### Paso 3: Crear una tabla estilizada para visualizar los pacientes

1. **Modificar la ruta para obtener los datos en formato HTML**:

   En `server.js`, modifica la ruta `/pacientes` para que muestre los datos dentro de una tabla HTML en lugar de solo texto:

   ```javascript
   // Ruta para mostrar los datos de la base de datos en formato HTML
   app.get('/pacientes', (req, res) => {
     connection.query('SELECT * FROM pacientes', (err, results) => {
       if (err) {
         return res.send('Error al obtener los datos.');
       }

       let html = `
         <html>
         <head>
           <link rel="stylesheet" href="/styles.css">
           <title>Pacientes</title>
         </head>
         <body>
           <h1>Pacientes Registrados</h1>
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

2. **Estilizar la tabla en CSS**:

   Agrega los siguientes estilos en `styles.css` para hacer que la tabla se vea más agradable:

   ```css
   table {
     width: 80%;
     margin: 20px auto;
     border-collapse: collapse;
   }

   th, td {
     padding: 12px;
     text-align: left;
     border-bottom: 1px solid #ddd;
   }

   th {
     background-color: #4CAF50;
     color: white;
   }

   tr:hover {
     background-color: #f5f5f5;
   }

   button {
     display: block;
     margin: 20px auto;
     padding: 10px;
     background-color: #007bff;
     color: white;
     border: none;
     border-radius: 4px;
     cursor: pointer;
   }

   button:hover {
     background-color: #0056b3;
   }
   ```

3. **Prueba la visualización de la tabla**:

   - Guarda los cambios y reinicia el servidor si es necesario.
   - Ve a `http://localhost:3000/pacientes`. Deberías ver los datos de los pacientes dentro de una tabla bien estilizada con encabezados y colores que hacen que la tabla se vea profesional.

---

### Paso 4: Personalización adicional

1. **Mejora el diseño de la página**:
   
   Revisa el archivo de styles.css, ¿Existen estilos para elementos repetidos? ¿Qué otra adecuación le podrías hacer al sistema para que la apariencia en general sea más agradable? Modifica los estilos, personaliza los colores o el diseño usando más propiedades CSS. Podrían agregar más estilos como:
   
   - **Colores personalizados**.
   - **Fuente diferente**.
   - **Ajustes para móviles** (usando `media queries`).
   - **Etc.

---
