# **Práctica 2: Gestión Avanzada de Tablas y Consultas en MySQL** (Versión Extendida)

### **Objetivo General**
El estudiante adquirirá una comprensión más profunda de la creación y gestión de múltiples tablas en MySQL, así como la realización de consultas avanzadas utilizando funciones y operaciones SQL.

---

## **Sección 1: Creación de Tablas con Relaciones**

### **Objetivo**  
Aprender a crear tablas relacionadas mediante claves foráneas y a insertar datos en ellas.

### **Pasos**  

1. **Crear una nueva base de datos**  
   ```sql
   CREATE DATABASE empresa;
   USE empresa;
   ```
   📌 *Si la base de datos ya existe, eliminarla antes de crearla de nuevo:*  
   ```sql
   DROP DATABASE IF EXISTS empresa;
   CREATE DATABASE empresa;
   USE empresa;
   ```

2. **Crear la tabla `departamentos`**  
   ```sql
   CREATE TABLE departamentos (
       id INT AUTO_INCREMENT PRIMARY KEY,
       nombre VARCHAR(50) NOT NULL,
       ubicacion VARCHAR(100)
   );
   ```

3. **Crear la tabla `empleados` con clave foránea**  
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

4. **Insertar datos en ambas tablas**  
   ```sql
   INSERT INTO departamentos (nombre, ubicacion) VALUES 
   ('Recursos Humanos', 'Planta 1'),
   ('Ventas', 'Planta 2'),
   ('IT', 'Planta 3');

   INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES 
   ('Ana', 'López', 45000, 1),
   ('Juan', 'Pérez', 50000, 2),
   ('Carlos', 'García', 60000, 3);
   ```

---

## **Sección 2: Consultas con Múltiples Tablas (JOINs)**

### **Objetivo**  
Aprender a realizar consultas que combinen datos de múltiples tablas.

### **Ejemplos**

1. **Mostrar empleados con el nombre de su departamento**  
   ```sql
   SELECT empleados.nombre, empleados.apellido, departamentos.nombre AS departamento
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id;
   ```

2. **Consultar empleados de un departamento específico**  
   ```sql
   SELECT empleados.nombre, empleados.apellido 
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id
   WHERE departamentos.nombre = 'IT';
   ```

3. **Consultar departamentos sin empleados** (LEFT JOIN)  
   ```sql
   SELECT departamentos.nombre, empleados.nombre 
   FROM departamentos
   LEFT JOIN empleados ON empleados.departamento_id = departamentos.id
   WHERE empleados.id IS NULL;
   ```

---

## **Sección 3: Uso de Funciones de Agregación**

### **Objetivo**  
Aprender a usar funciones como `COUNT`, `AVG`, `MAX`, `MIN` para obtener estadísticas.

### **Ejemplos**

1. **Contar empleados por departamento**  
   ```sql
   SELECT departamentos.nombre, COUNT(empleados.id) AS num_empleados
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id
   GROUP BY departamentos.nombre;
   ```

2. **Salario promedio por departamento**  
   ```sql
   SELECT departamentos.nombre, AVG(empleados.salario) AS salario_promedio
   FROM empleados
   JOIN departamentos ON empleados.departamento_id = departamentos.id
   GROUP BY departamentos.nombre;
   ```

3. **Salario máximo y mínimo**  
   ```sql
   SELECT MAX(salario) AS salario_maximo, MIN(salario) AS salario_minimo FROM empleados;
   ```

4. **Sumar los salarios de todos los empleados**  
   ```sql
   SELECT SUM(salario) AS total_salarios FROM empleados;
   ```

---

## **Sección 4: Modificación de Tablas**

### **Objetivo**  
Modificar la estructura de las tablas y actualizar datos.

### **Ejemplos**

1. **Agregar una columna `fecha_contratacion` a `empleados`**  
   ```sql
   ALTER TABLE empleados ADD COLUMN fecha_contratacion DATE;
   ```

2. **Actualizar datos en la nueva columna**  
   ```sql
   UPDATE empleados SET fecha_contratacion = '2020-05-10' WHERE nombre = 'Ana';
   UPDATE empleados SET fecha_contratacion = '2019-07-22' WHERE nombre = 'Juan';
   UPDATE empleados SET fecha_contratacion = '2021-01-15' WHERE nombre = 'Carlos';
   ```

3. **Eliminar una columna**  
   ```sql
   ALTER TABLE empleados DROP COLUMN fecha_contratacion;
   ```

---

## **Sección 5: Consultas Condicionales y Ordenación de Datos**

### **Objetivo**  
Filtrar y ordenar registros con `WHERE` y `ORDER BY`.

### **Ejemplos**

1. **Empleados contratados después de 2020**  
   ```sql
   SELECT nombre, apellido, fecha_contratacion 
   FROM empleados 
   WHERE fecha_contratacion > '2020-01-01';
   ```

2. **Ordenar empleados por salario de mayor a menor**  
   ```sql
   SELECT nombre, apellido, salario 
   FROM empleados 
   ORDER BY salario DESC;
   ```

3. **Filtrar empleados con salario entre 45000 y 60000**  
   ```sql
   SELECT * FROM empleados WHERE salario BETWEEN 45000 AND 60000;
   ```

4. **Buscar empleados cuyo apellido comienza con 'G'**  
   ```sql
   SELECT * FROM empleados WHERE apellido LIKE 'G%';
   ```

---

## **Sección 6: Eliminación de Datos y Tablas**

### **Objetivo**  
Eliminar registros y tablas completas.

### **Ejemplos**

1. **Eliminar un empleado específico**  
   ```sql
   DELETE FROM empleados WHERE nombre = 'Carlos';
   ```

2. **Eliminar empleados de un departamento específico**  
   ```sql
   DELETE FROM empleados WHERE departamento_id = 3;
   ```

3. **Eliminar toda la tabla `empleados`**  
   ```sql
   DROP TABLE empleados;
   ```

4. **Eliminar la base de datos completa**  
   ```sql
   DROP DATABASE empresa;
   ```

---

## **Ejercicios Adicionales**

📌 **1. Crea una tabla `proyectos` con la siguiente estructura:**  
   - `id` (INT, clave primaria, auto-incremento).  
   - `nombre` (VARCHAR 100).  
   - `departamento_id` (clave foránea de `departamentos`).  

📌 **2. Inserta al menos 3 proyectos en la tabla.**  

📌 **3. Muestra todos los proyectos junto con el nombre del departamento responsable.**  

📌 **4. Filtra proyectos asignados al departamento de IT.**  

📌 **5. Ordena los proyectos por nombre en orden alfabético.**  

---
