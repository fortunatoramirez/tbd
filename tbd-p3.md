### **Práctica 3: Consultas Avanzadas y Optimización en MySQL**

---

### **Conceptos Generales**

1. **Índices:**
   Un índice en MySQL es una estructura de datos que mejora la velocidad de las operaciones de selección en una tabla a expensas de una mayor utilización de espacio y tiempo durante las operaciones de inserción y actualización. Los índices funcionan como un índice de un libro, facilitando la búsqueda rápida de registros en una tabla.

   **Diagrama de Concepto de Índices:**

   ```
   Tabla sin índice:                    Tabla con índice:
   +----+--------+--------+             +----+--------+--------+--------+
   | ID | Nombre | Apellido|             | ID | Nombre | Apellido | Índice|
   +----+--------+--------+             +----+--------+--------+--------+
   | 1  | Juan   | Pérez   |             | 1  | Juan   | Pérez   |   P   |
   | 2  | Ana    | Gómez   |    --->     | 2  | Ana    | Gómez   |   G   |
   | 3  | Luis   | Martínez|             | 3  | Luis   | Martínez|   M   |
   +----+--------+--------+             +----+--------+--------+--------+
   ```

   Los índices permiten encontrar rápidamente valores como `Pérez` al organizar los datos internamente.

---

2. **Subconsultas:**
   Las subconsultas son consultas SQL anidadas dentro de otras consultas. Se utilizan para obtener resultados intermedios que luego se utilizan en la consulta externa.

   **Diagrama de Concepto de Subconsultas:**

   ```
   Consulta externa:
   SELECT * FROM empleados WHERE salario > (...);

   Subconsulta:
   (SELECT AVG(salario) FROM empleados)
   ```

   La subconsulta calcula un valor (el salario promedio en este caso) que luego es utilizado por la consulta externa para obtener los empleados cuyo salario es mayor que el promedio.

---

3. **Vistas:**
   Una vista en MySQL es una tabla virtual que resulta de una consulta. No almacena datos físicamente, pero facilita la reutilización de consultas complejas y permite organizar mejor las consultas.

   **Diagrama de Concepto de Vistas:**

   ```
   Vista creada:                 Consulta original:
   CREATE VIEW empleados_vista    SELECT empleados.nombre, departamentos.nombre 
   AS SELECT ...                  FROM empleados JOIN departamentos ON ...;

   Consulta a la vista:
   SELECT * FROM empleados_vista;
   ```

   La vista permite simplificar consultas repetitivas.

---

4. **Transacciones:**
   Una transacción en MySQL es un conjunto de operaciones SQL que se ejecutan como una unidad. Si alguna de las operaciones falla, todas las demás se revierten, garantizando que la base de datos quede en un estado coherente.

   **Diagrama de Concepto de Transacciones:**

   ```
   START TRANSACTION;             -- Inicia la transacción
   INSERT INTO empleados ...;     -- Operación 1
   INSERT INTO empleados ...;     -- Operación 2

   Si todo es correcto:
   COMMIT;                        -- Confirma los cambios

   Si hay un error:
   ROLLBACK;                      -- Revierte todos los cambios
   ```

   Las transacciones permiten garantizar la integridad de los datos.

---

5. **Optimización de Consultas (EXPLAIN):**
   La instrucción `EXPLAIN` en MySQL se utiliza para analizar cómo se ejecuta una consulta y sugerir mejoras, como el uso de índices.

   **Diagrama de Concepto de `EXPLAIN`:**

   ```
   Consulta: SELECT * FROM empleados WHERE salario > 50000;

   Salida de EXPLAIN:
   +----+-------------+------------+------+---------------+------+---------+
   | id | select_type | table      | type | possible_keys | key  | rows    |
   +----+-------------+------------+------+---------------+------+---------+
   |  1 | SIMPLE      | empleados  | ALL  | NULL          | NULL | 10000   |
   +----+-------------+------------+------+---------------+------+---------+
   ```

   `EXPLAIN` muestra cómo MySQL procesa una consulta y ayuda a encontrar áreas de mejora, como la adición de índices.

---

6. **Manejo de Errores:**
   Los errores en MySQL pueden ocurrir por sintaxis incorrecta o por intentos de ejecutar operaciones no permitidas. Al aprender a gestionar errores, los estudiantes pueden depurar sus consultas y mejorar su código SQL.

   **Diagrama de Errores Comunes:**

   ```
   Consulta con error: 
   SELECT * FORM empleados;

   Error generado: 
   Error: You have an error in your SQL syntax...
   
   Consulta corregida:
   SELECT * FROM empleados;
   ```

   Aprender a identificar y corregir errores en consultas es fundamental para la depuración.

---

### **Objetivo General de la Práctica 3:**
El objetivo de esta práctica es que el estudiante explore técnicas avanzadas de MySQL, desde el uso de índices y subconsultas hasta la optimización de consultas y el manejo de transacciones, aplicando estos conceptos para gestionar bases de datos de manera eficiente.

---

### **Sección 1: Uso de Índices**

**Objetivo:** Mejorar el rendimiento de las consultas utilizando índices en MySQL.

1. **Crear un índice en la tabla `empleados`:**
   - El estudiante debe crear un índice en la columna `apellido` para acelerar las consultas que la involucren:
   
     ```sql
     CREATE INDEX idx_apellido ON empleados(apellido);
     ```

2. **Consultar utilizando el índice:**
   - Realizar una consulta que aproveche el índice creado para encontrar empleados por su apellido:
   
     ```sql
     SELECT * FROM empleados WHERE apellido = 'Pérez';
     ```

3. **Mostrar los índices de una tabla:**
   - El estudiante debe listar los índices disponibles en la tabla `empleados`:
   
     ```sql
     SHOW INDEX FROM empleados;
     ```

---

### **Sección 2: Subconsultas**

**Objetivo:** Aprender a utilizar subconsultas para realizar consultas más complejas y obtener resultados más específicos.

1. **Subconsulta simple:**
   - El estudiante debe encontrar los empleados cuyo salario es mayor al promedio de todos los empleados:
   
     ```sql
     SELECT nombre, apellido, salario 
     FROM empleados 
     WHERE salario > (SELECT AVG(salario) FROM empleados);
     ```

2. **Subconsulta con `IN`:**
   - El estudiante debe listar todos los empleados que pertenecen a los departamentos que están en la planta 2:
   
     ```sql
     SELECT nombre, apellido 
     FROM empleados 
     WHERE departamento_id IN (SELECT id FROM departamentos WHERE ubicacion = 'Planta 2');
     ```

---

### **Sección 3: Vistas**

**Objetivo:** Facilitar consultas repetitivas y mejorar la organización de consultas complejas mediante la creación de vistas.

1. **Crear una vista para mostrar empleados y sus departamentos:**
   - El estudiante debe crear una vista que combine la información de las tablas `empleados` y `departamentos`:
   
     ```sql
     CREATE VIEW vista_empleados_departamentos AS
     SELECT empleados.nombre, empleados.apellido, departamentos.nombre AS departamento
     FROM empleados
     JOIN departamentos ON empleados.departamento_id = departamentos.id;
     ```

2. **Consultar la vista creada:**
   - El estudiante debe consultar la vista para obtener los datos combinados:
   
     ```sql
     SELECT * FROM vista_empleados_departamentos;
     ```

3. **Modificar una vista:**
   - El estudiante debe actualizar la vista para incluir el salario de los empleados:
   
     ```sql
     CREATE OR REPLACE VIEW vista_empleados_departamentos AS
     SELECT empleados.nombre, empleados.apellido, empleados.salario, departamentos.nombre AS departamento
     FROM empleados
     JOIN departamentos ON empleados.departamento_id = departamentos.id;
     ```

---

### **Sección 4: Transacciones y Control de Integridad**

**Objetivo:** Comprender el uso de transacciones en MySQL para asegurar la consistencia y la integridad de los datos.

1. **Iniciar una transacción:**
   - El estudiante debe iniciar una transacción, realizar una inserción en la tabla `empleados` y luego revertirla:
   
     ```sql
     START TRANSACTION;
     INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES ('María', 'Fernández', 70000, 1);
     ROLLBACK;
     ```

2. **Confirmar cambios usando `COMMIT`:**
   - El estudiante debe realizar una inserción y confirmar los cambios con `COMMIT`:
   
     ```sql
     START TRANSACTION;
     INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES ('Pedro', 'Sánchez', 65000, 2);
     COMMIT;
     ```

---

### **Sección 5: Optimización de Consultas**

**Objetivo:** Mejorar la eficiencia de las consultas mediante la utilización de buenas prácticas y herramientas de MySQL.

1. **Uso de `EXPLAIN`:**
   - El estudiante debe analizar el plan de ejecución de una consulta para optimizarla:
   
     ```sql
     EXPLAIN SELECT * FROM empleados WHERE salario > 50000;
     ```

2. **Optimización de una consulta con índices:**
   - Crear un índice en la columna `salario` y ejecutar de nuevo la consulta para verificar la mejora:
   
     ```sql
     CREATE INDEX idx_salario ON empleados(salario);
     SELECT * FROM empleados WHERE salario > 50000;
     ```

---

### **Sección 6: Manejo de Errores y Depuración**

**Objetivo:** Comprender cómo gestionar errores en MySQL y depurar consultas SQL.

1. **Captura de errores:**
   - El estudiante debe ejecutar una consulta incorrecta y observar cómo se maneja el error:
   
     ```sql
     SELECT * FORM empleados;  -- Error en la palabra FORM
     ```

2. **Revisión y corrección de errores comunes:**
   - El estudiante debe identificar el error en la consulta anterior y corregirlo:
   
     ```sql
     SELECT * FROM empleados;
     ```
