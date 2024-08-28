## **Práctica 1: Introducción a MySQL**

### **Objetivo General:**
Familiarizar al estudiante con los conceptos básicos y las operaciones fundamentales en MySQL, desde la instalación y configuración hasta la creación y manipulación de bases de datos, tablas, y datos.

---

### **Sección 1: Instalación y Configuración**

**Objetivo:** Asegurarse de que MySQL esté instalado y configurado correctamente en el sistema del estudiante.

1. **Instalación de MySQL:**
   - **Windows:** El estudiante debe descargar el instalador de MySQL desde la [página oficial](https://dev.mysql.com/downloads/installer/). Debe seguir las instrucciones del instalador para la configuración básica.
   - **macOS:** El estudiante instalará MySQL utilizando Homebrew con el siguiente comando:

     ```bash
     brew install mysql
     ```

   - **Linux (Ubuntu):** El estudiante instalará MySQL utilizando apt con los comandos:

     ```bash
     sudo apt update
     sudo apt install mysql-server
     ```

2. **Verificación de la instalación:**
   - El estudiante debe verificar que MySQL está instalado ejecutando el siguiente comando en la terminal:

     ```bash
     mysql -V
     ```

   - Luego, deberá iniciar sesión en MySQL como root para confirmar que todo funciona correctamente:

     ```bash
     mysql -u root -p
     ```

---

### **Sección 2: Conceptos Básicos de MySQL**

**Objetivo:** Introducir al estudiante en los conceptos fundamentales de bases de datos relacionales y el lenguaje SQL.

1. **Bases de Datos Relacionales:**
   - Una **base de datos** es una colección organizada de datos.
   - Una **tabla** es una estructura dentro de una base de datos que almacena datos en filas y columnas.
   - **SQL (Structured Query Language)** es el lenguaje estándar para interactuar con bases de datos relacionales.

2. **Componentes Clave:**
   - **Filas (Registros):** Cada fila representa una entrada única en la tabla.
   - **Columnas (Campos):** Cada columna representa un atributo de los datos, como nombre, edad, etc.
   - **Claves Primarias:** Un campo (o conjunto de campos) que identifica de manera única cada registro en la tabla.

---

### **Sección 3: Iniciando Sesión y Gestión de Usuarios**

**Objetivo:** Aprender a iniciar sesión en MySQL y manejar usuarios y permisos.

1. **Iniciar sesión en MySQL:**
   - El estudiante debe iniciar sesión como el usuario root con el siguiente comando:

     ```bash
     mysql -u root -p
     ```

2. **Crear un nuevo usuario:**
   - El estudiante creará un usuario llamado `nuevo_usuario` utilizando la siguiente instrucción SQL:

     ```sql
     CREATE USER 'nuevo_usuario'@'localhost' IDENTIFIED BY 'contraseña_segura';
     ```

3. **Asignar permisos:**
   - Se deben otorgar todos los privilegios al nuevo usuario sobre todas las bases de datos:

     ```sql
     GRANT ALL PRIVILEGES ON *.* TO 'nuevo_usuario'@'localhost' WITH GRANT OPTION;
     ```

4. **Aplicar cambios:**
   - El estudiante debe asegurarse de que los cambios en los permisos se apliquen correctamente:

     ```sql
     FLUSH PRIVILEGES;
     ```

---

### **Sección 4: Creación y Gestión de Bases de Datos**

**Objetivo:** Aprender a crear, modificar y eliminar bases de datos.

1. **Crear una base de datos:**
   - El estudiante debe crear una base de datos llamada `mi_base_de_datos` con el siguiente comando:

     ```sql
     CREATE DATABASE mi_base_de_datos;
     ```

2. **Mostrar bases de datos existentes:**
   - Se deben listar todas las bases de datos disponibles en el sistema:

     ```sql
     SHOW DATABASES;
     ```

3. **Seleccionar una base de datos:**
   - El estudiante debe usar la base de datos recién creada con el siguiente comando:

     ```sql
     USE mi_base_de_datos;
     ```

4. **Eliminar una base de datos:**
   - Finalmente, se eliminará la base de datos usando el siguiente comando:

     ```sql
     DROP DATABASE mi_base_de_datos;
     ```

---

### **Sección 5: Creación y Gestión de Tablas**

**Objetivo:** Crear, modificar y eliminar tablas en MySQL.

1. **Crear una tabla:**
   - Dentro de la base de datos `mi_base_de_datos`, el estudiante debe crear una tabla llamada `empleados` con la siguiente estructura:

     ```sql
     CREATE TABLE empleados (
         id INT AUTO_INCREMENT PRIMARY KEY,
         nombre VARCHAR(50),
         apellido VARCHAR(50),
         salario DECIMAL(10, 2)
     );
     ```

2. **Mostrar tablas existentes:**
   - Se deben listar todas las tablas en la base de datos actual con el siguiente comando:

     ```sql
     SHOW TABLES;
     ```

3. **Modificar la estructura de la tabla:**
   - El estudiante debe agregar una columna `fecha_contratacion` a la tabla `empleados` con el siguiente comando:

     ```sql
     ALTER TABLE empleados ADD fecha_contratacion DATE;
     ```

4. **Eliminar una tabla:**
   - Finalmente, se eliminará la tabla `empleados` con el siguiente comando:

     ```sql
     DROP TABLE empleados;
     ```

---

### **Sección 6: Manipulación de Datos**

**Objetivo:** Insertar, actualizar, consultar y eliminar datos en las tablas.

1. **Insertar datos en una tabla:**
   - El estudiante debe insertar registros en la tabla `empleados` utilizando las siguientes instrucciones SQL:

     ```sql
     INSERT INTO empleados (nombre, apellido, salario, fecha_contratacion) VALUES ('Juan', 'Pérez', 3000.00, '2023-01-15');
     INSERT INTO empleados (nombre, apellido, salario, fecha_contratacion) VALUES ('Ana', 'Gómez', 3200.00, '2023-03-22');
     INSERT INTO empleados (nombre, apellido, salario, fecha_contratacion) VALUES ('Luis', 'Martínez', 2800.00, '2023-02-10');
     ```

2. **Consultar datos de la tabla:**
   - Se deben recuperar todos los registros de la tabla `empleados` con el siguiente comando:

     ```sql
     SELECT * FROM empleados;
     ```

   - El estudiante también debe filtrar los registros donde el salario es mayor a 3000:

     ```sql
     SELECT * FROM empleados WHERE salario > 3000;
     ```

3. **Actualizar registros:**
   - El estudiante debe aumentar el salario de Ana Gómez a 3500 con el siguiente comando:

     ```sql
     UPDATE empleados SET salario = 3500.00 WHERE nombre = 'Ana' AND apellido = 'Gómez';
     ```

4. **Eliminar registros:**
   - Finalmente, se eliminará el registro de Luis Martínez con el siguiente comando:

     ```sql
     DELETE FROM empleados WHERE nombre = 'Luis' AND apellido = 'Martínez';
     ```

---

Esta práctica cubre los conceptos y operaciones fundamentales que todo estudiante debe conocer para empezar a trabajar con MySQL de manera efectiva. Al completar esta práctica, el estudiante habrá adquirido una base sólida para realizar operaciones básicas de gestión de bases de datos y tablas.
