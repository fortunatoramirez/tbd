## **Práctica 2: Gestión Avanzada de Tablas y Consultas en MySQL**

### **Objetivo General de la Práctica:**
El objetivo de esta práctica es que el estudiante adquiera una comprensión más profunda de la creación y gestión de múltiples tablas en MySQL, así como de la realización de consultas avanzadas utilizando funciones y operaciones SQL.

---

### **Sección 1: Creación de Tablas con Relaciones**

#### **Objetivo de la Sección:**
El estudiante aprenderá a crear tablas que contengan relaciones mediante el uso de claves foráneas, y a insertar datos en dichas tablas.

1. **Crear una nueva base de datos**:
   - Crear y seleccionar la base de datos en la que trabajará el estudiante.
   ```sql
   CREATE DATABASE empresa;
   USE empresa;
   ```

2. **Crear una tabla llamada `departamentos`**:
   - Diseñar la estructura de una tabla para almacenar los departamentos de una empresa.
   ```sql
   CREATE TABLE departamentos (
     id INT AUTO_INCREMENT PRIMARY KEY,
     nombre VARCHAR(50) NOT NULL,
     ubicacion VARCHAR(100)
   );
   ```

3. **Crear una tabla llamada `empleados`**:
   - Crear una tabla para almacenar los empleados, vinculada a la tabla `departamentos` mediante una clave foránea.
   ```sql
   CREATE TABLE empleados (
     id INT AUTO_INCREMENT PRIMARY KEY,
     nombre VARCHAR(50) NOT NULL,
     apellido VARCHAR(50) NOT NULL,
     salario DECIMAL(10,2),
     departamento_id INT,
     FOREIGN KEY (departamento_id) REFERENCES departamentos(id)
   );
   ```

4. **Insertar datos en ambas tablas**:
   - Insertar registros en las tablas `departamentos` y `empleados`.
   ```sql
   INSERT INTO departamentos (nombre, ubicacion) VALUES ('Recursos Humanos', 'Planta 1');
   INSERT INTO departamentos (nombre, ubicacion) VALUES ('Ventas', 'Planta 2');
   INSERT INTO departamentos (nombre, ubicacion) VALUES ('IT', 'Planta 3');

   INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES ('Ana', 'López', 45000, 1);
   INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES ('Juan', 'Pérez', 50000, 2);
   INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES ('Carlos', 'García', 60000, 3);
   ```

---

### **Sección 2: Consultas con Múltiples Tablas (JOINs)**

#### **Objetivo de la Sección:**
El estudiante aprenderá a realizar consultas que combinen datos de múltiples tablas mediante el uso de `JOIN`.

1. **Mostrar los empleados junto con el nombre de su departamento**:
   - Realizar una consulta que relacione las tablas `empleados` y `departamentos` usando `JOIN`.
   ```sql
   SELECT empleados.nombre, empleados.apellido, departamentos.nombre AS departamento
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id;
   ```

2. **Consultar empleados de un departamento específico**:
   - Consultar solo los empleados que pertenecen al departamento de IT.
   ```sql
   SELECT empleados.nombre, empleados.apellido
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id
   WHERE departamentos.nombre = 'IT';
   ```

---

### **Sección 3: Uso de Funciones de Agregación**

#### **Objetivo de la Sección:**
El estudiante utilizará funciones de agregación como `COUNT`, `AVG`, `MAX`, y `MIN` para generar estadísticas sobre los datos almacenados.

1. **Contar cuántos empleados hay en cada departamento**:
   - Usar `COUNT` para contar registros agrupados por departamento.
   ```sql
   SELECT departamentos.nombre, COUNT(empleados.id) AS num_empleados
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id
   GROUP BY departamentos.nombre;
   ```

2. **Calcular el salario promedio por departamento**:
   - Usar `AVG` para calcular el salario promedio por departamento.
   ```sql
   SELECT departamentos.nombre, AVG(empleados.salario) AS salario_promedio
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id
   GROUP BY departamentos.nombre;
   ```

3. **Obtener el salario máximo y mínimo de los empleados**:
   - Usar `MAX` y `MIN` para determinar el salario más alto y más bajo.
   ```sql
   SELECT MAX(salario) AS salario_maximo, MIN(salario) AS salario_minimo FROM empleados;
   ```

---

### **Sección 4: Modificación de Tablas**

#### **Objetivo de la Sección:**
El estudiante aprenderá a modificar la estructura de las tablas agregando nuevas columnas y actualizando los datos existentes.

1. **Agregar una columna llamada `fecha_contratacion` a la tabla `empleados`**:
   - Añadir una nueva columna para almacenar la fecha de contratación de cada empleado.
   ```sql
   ALTER TABLE empleados ADD COLUMN fecha_contratacion DATE;
   ```

2. **Actualizar datos en la nueva columna `fecha_contratacion`**:
   - Incluir la fecha de contratación para algunos empleados.
   ```sql
   UPDATE empleados SET fecha_contratacion = '2020-05-10' WHERE nombre = 'Ana';
   UPDATE empleados SET fecha_contratacion = '2019-07-22' WHERE nombre = 'Juan';
   UPDATE empleados SET fecha_contratacion = '2021-01-15' WHERE nombre = 'Carlos';
   ```

---

### **Sección 5: Consultas Condicionales y Ordenación de Datos**

#### **Objetivo de la Sección:**
El estudiante realizará consultas condicionales utilizando `WHERE` y aprenderá a ordenar los resultados con `ORDER BY`.

1. **Consultar empleados contratados después del 1 de enero de 2020**:
   - Utilizar la cláusula `WHERE` para obtener resultados basados en una condición.
   ```sql
   SELECT nombre, apellido, fecha_contratacion 
   FROM empleados 
   WHERE fecha_contratacion > '2020-01-01';
   ```

2. **Ordenar los empleados por salario en orden descendente**:
   - Utilizar `ORDER BY` para ordenar los resultados de mayor a menor.
   ```sql
   SELECT nombre, apellido, salario 
   FROM empleados 
   ORDER BY salario DESC;
   ```

---

### **Sección 6: Eliminar Datos y Tablas**

#### **Objetivo de la Sección:**
El estudiante aprenderá a eliminar registros específicos y tablas completas en MySQL.

1. **Eliminar un empleado de la tabla `empleados`**:
   - Borrar un registro específico de la tabla.
   ```sql
   DELETE FROM empleados WHERE nombre = 'Carlos';
   ```

2. **Eliminar la tabla `empleados`**:
   - Eliminar toda la tabla `empleados`.
   ```sql
   DROP TABLE empleados;
   ```
