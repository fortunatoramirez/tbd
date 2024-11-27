# **Práctica 9: Implementación de Funcionalidades Avanzadas en Node.js y MySQL**

## **Objetivos:**
1. Aprender a usar **Nodemon** para evitar reiniciar el servidor manualmente.  
2. Cargar una **barra de navegación** en múltiples páginas y hacer que parezca fija.  
3. Implementar una **búsqueda en tiempo real** usando `keyup` y consultas dinámicas a la base de datos.  
4. Manejar **carga y descarga de archivos Excel** desde y hacia la base de datos.
5. Ajustar el servidor y la base de datos a la zona horaria correcta.  

---

## **Instrucciones:**

### **Paso 1: Configuración del Proyecto**

1. **Crear la carpeta del proyecto:**  
   Crea una carpeta llamada `practica9-avanzada` y ábrela en Visual Studio Code.

2. **Inicializar el proyecto:**  
   En la terminal, ejecuta:  
   ```bash
   npm init -y
   ```

3. **Instalar dependencias necesarias:**  
   Ejecuta:  
   ```bash
   npm install express mysql2 dotenv multer xlsx nodemon bcrypt
   ```
   - **express**: Para crear el servidor.  
   - **mysql2**: Para interactuar con la base de datos.  
   - **dotenv**: Para manejar variables de entorno.  
   - **multer**: Para cargar archivos desde formularios.  
   - **xlsx**: Para manejar archivos Excel.  
   - **nodemon**: Para reiniciar automáticamente el servidor cuando hagas cambios.  
   - **bcrypt**: Para el hashing de contraseñas.


4. **Configurar Nodemon:**  
   - Crea un archivo llamado `nodemon.json` con el siguiente contenido:  
     ```json
     {
       "watch": ["server.js"],
       "ext": "js,json",
       "ignore": ["node_modules"],
       "exec": "node server.js"
     }
     ```
   - Modifica el archivo `package.json` para usar Nodemon:  
     ```json
     "scripts": {
       "start": "nodemon"
     }
     ```

---

### **Paso 2: Configuración del Servidor**

1. Crea un archivo `server.js` con el siguiente código inicial:

   ```js
   const express = require('express');
   const mysql = require('mysql2');
   const multer = require('multer');
   const xlsx = require('xlsx');
   const bcrypt = require('bcrypt');
   const path = require('path');
   const app = express();
   require('dotenv').config();

   // Configuración de MySQL
   const db = mysql.createConnection({
     host: 'localhost',
     user: 'root',
     password: '',
     database: 'biomedica'
   });

   db.connect(err => {
     if (err) throw err;
     console.log('Conectado a la base de datos');
   });

   // Configuración de Middleware
   app.use(express.static(path.join(__dirname, 'public')));
   app.use(express.urlencoded({ extended: true }));
   app.use(express.json());

   // Configuración de puerto
   const PORT = process.env.PORT || 3000;
   app.listen(PORT, () => console.log(`Servidor en funcionamiento en el puerto ${PORT}`));
   ```

2. Crea un archivo `.env` en la raíz del proyecto para almacenar variables de entorno:  
   ```env
   PORT=3000
   DB_HOST=localhost
   DB_USER=root
   DB_PASSWORD=
   DB_NAME=biomedica
   ```

---

### **Paso 3: Estructura del Proyecto**

1. Crea la siguiente estructura de carpetas y archivos:

   ```
   practica9-avanzada/
   ├── public/
   │   ├── index.html
   │   ├── equipos.html
   │   ├── usuarios.html
   │   ├── busqueda.html
   │   ├── navbar.html
   │   └── styles.css
   ├── uploads/
   ├── server.js
   └── .env
   ```

2. **Navbar (public/navbar.html):**  
   ```html
   <nav>
     <ul id="menu"></ul>
   </nav>
   <script>
     window.onload = () => {
       fetch('/menu')
         .then(res => res.json())
         .then(data => {
           const menu = document.getElementById('menu');
           data.forEach(item => {
             const li = document.createElement('li');
             li.innerHTML = `<a href="${item.url}">${item.nombre}</a>`;
             menu.appendChild(li);
           });
         });
     };
   </script>
   ```

3. **Estilo CSS (public/styles.css):**  
   ```css
   body {
     font-family: Arial, sans-serif;
     margin: 0;
     padding: 0;
     background-color: #f4f4f4;
   }

   nav {
     background-color: #333;
     color: white;
     padding: 10px;
   }

   ul {
     list-style: none;
     padding: 0;
     margin: 0;
     display: flex;
   }

   li {
     margin: 0 15px;
   }

   a {
     color: white;
     text-decoration: none;
   }

   a:hover {
     text-decoration: underline;
   }
   ```

4. **Página de Inicio (public/index.html):**  
   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Inicio</title>
     <link rel="stylesheet" href="styles.css">
   </head>
   <body>
     <div id="navbar"></div>
     <h1>Bienvenido al sistema de gestión biomédica</h1>
     <script src="navbar.html"></script>
   </body>
   </html>
   ```

---

### **Paso 4: Configuración de Rutas del Servidor**

1. **Rutas para el menú de navegación (server.js):**  
   Agrega esta ruta para enviar el menú de manera dinámica:

   ```js
   app.get('/menu', (req, res) => {
     const menuItems = [
       { nombre: 'Inicio', url: '/index.html' },
       { nombre: 'Equipos', url: '/equipos.html' },
       { nombre: 'Usuarios', url: '/usuarios.html' },
       { nombre: 'Búsqueda', url: '/busqueda.html' }
     ];
     res.json(menuItems);
   });
   ```

---

### **Paso 5: Implementar Búsqueda en Tiempo Real**

1. **Formulario de Búsqueda (public/busqueda.html):**  
   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Búsqueda en Tiempo Real</title>
     <link rel="stylesheet" href="styles.css">
   </head>
   <body>
     <div id="navbar"></div>
     <h1>Búsqueda en Tiempo Real</h1>
     <input type="text" id="search" placeholder="Buscar usuario..." />
     <ul id="results"></ul>

     <script src="navbar.html"></script>
     <script>
       document.getElementById('search').addEventListener('keyup', function() {
         const query = this.value;
         fetch(`/buscar?query=${query}`)
           .then(res => res.json())
           .then(data => {
             const results = document.getElementById('results');
             results.innerHTML = '';
             data.forEach(usuario => {
               const li = document.createElement('li');
               li.textContent = `${usuario.nombre} (${usuario.correo})`;
               results.appendChild(li);
             });
           });
       });
     </script>
   </body>
   </html>
   ```

2. **Ruta de Búsqueda en el Servidor (server.js):**  
   Agrega esta ruta para manejar la búsqueda:

   ```js
   app.get('/buscar', (req, res) => {
     const query = req.query.query;
     const sql = `SELECT nombre, correo FROM usuarios WHERE nombre LIKE ?`;
     db.query(sql, [`%${query}%`], (err, results) => {
       if (err) throw err;
       res.json(results);
     });
   });
   ```

---

### **Paso 6: Manejo de Archivos Excel**

#### **Carga de Archivos Excel**

1. **Formulario para Subir Archivos (public/equipos.html):**  
   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Equipos</title>
     <link rel="stylesheet" href="styles.css">
   </head>
   <body>
     <div id="navbar"></div>
     <h1>Cargar Equipos desde Excel</h1>
     <form action="/upload" method="POST" enctype="multipart/form-data">
       <input type="file" name="excelFile" accept=".xlsx" />
       <button type="submit">Subir Archivo</button>
     </form>

     <script src="navbar.html"></script>
   </body>
   </html>
   ```

2. **Ruta para Manejar la Carga (server.js):**  
   ```js
   const upload = multer({ dest: 'uploads/' });

   app.post('/upload', upload.single('excelFile'), (req, res) => {
     const filePath = req.file.path;
     const workbook = xlsx.readFile(filePath);
     const sheetName = workbook.SheetNames[0];
     const data = xlsx.utils.sheet_to_json(workbook.Sheets[sheetName]);

     data.forEach(row => {
       const { nombre, descripcion } = row;
       const sql = `INSERT INTO equipos (nombre, descripcion) VALUES (?, ?)`;
       db.query(sql, [nombre, descripcion], err => {
         if (err) throw err;
       });
     });

     res.send('<h1>Archivo cargado y datos guardados</h1><a href="/equipos.html">Volver</a>');
   });
   ```

#### **Descarga de Archivos Excel**

1. **Botón para Descargar Archivos (public/equipos.html):**  
   Agrega este botón debajo del formulario de carga:

   ```html
   <button onclick="window.location.href='/download'">Descargar Equipos</button>
   ```

2. **Ruta para la Descarga (server.js):**  
   ```js
   app.get('/download', (req, res) => {
     const sql = `SELECT * FROM equipos`;
     db.query(sql, (err, results) => {
       if (err) throw err;

       const worksheet = xlsx.utils.json_to_sheet(results);
       const workbook = xlsx.utils.book_new();
       xlsx.utils.book_append_sheet(workbook, worksheet, 'Equipos');

       const filePath = path.join(__dirname, 'uploads', 'equipos.xlsx');
       xlsx.writeFile(workbook, filePath);
       res.download(filePath, 'equipos.xlsx');
     });
   });
   ```

---

### **Paso 7: Configurar Zona Horaria en MySQL**

1. **Comando SQL:**  
   Asegúrate de que MySQL esté configurado con la zona horaria correcta:

   ```sql
   SET GLOBAL time_zone = '-08:00';
   ```

2. **En el Servidor:**  
   Agrega esta línea al crear la conexión en `server.js`:

   ```js
   timezone: 'America/Tijuana'
   ```

---

### **Paso 8: Uso de Usuarios Personalizados en MySQL**

1. **Crear un Usuario con Permisos:**  
   Ejecuta estos comandos en MySQL:

   ```sql
   CREATE USER 'biomedico'@'localhost' IDENTIFIED BY 'password123';
   GRANT ALL PRIVILEGES ON biomedica.* TO 'biomedico'@'localhost';
   FLUSH PRIVILEGES;
   ```

2. **Actualizar la Conexión (server.js):**  
   Cambia el usuario en la configuración de la conexión:

   ```js
   const db = mysql.createConnection({
     host: 'localhost',
     user: 'biomedico',
     password: 'password123',
     database: 'biomedica'
   });
   ```

---

### **Paso 9: Analizar el funcionamiento de cada etapa y realizar las adecuaciones que considere necesarias para el funcionamiento y comprensiónm de cada etapa**

---

### **Paso 10: agregar estas funcionalidades a la Práctica 8**
