# Práctica 3: Índices, Subconsultas, Vistas, Transacciones y Optimización en MySQL

## Objetivo general

Al finalizar esta práctica, el estudiante será capaz de:

- identificar los índices existentes en una tabla y comprender los indicadores que muestra MySQL;
- crear y eliminar índices;
- utilizar subconsultas para resolver consultas que dependen de otros resultados;
- crear y modificar vistas;
- utilizar transacciones con `COMMIT` y `ROLLBACK`;
- interpretar los campos principales de `EXPLAIN`;
- comprobar que la existencia de un índice **no significa necesariamente que MySQL vaya a utilizarlo**;
- identificar y corregir errores comunes en consultas SQL.

> **Compatibilidad:** los ejemplos están pensados para MySQL 8.0 o superior.  
> `EXPLAIN ANALYZE`, utilizado al final como actividad opcional, está disponible en MySQL 8.0.18 o superior.

---

# 0. Preparación de la práctica

En las prácticas anteriores la base de datos pudo haber sido modificada, e incluso algunas actividades pudieron eliminar tablas. Para evitar que cada estudiante comience con un estado diferente, en esta práctica se reconstruirá una base de datos conocida.

> **Importante:** el siguiente bloque elimina la base de datos `empresa` si ya existe y la vuelve a crear.

```sql
DROP DATABASE IF EXISTS empresa;
CREATE DATABASE empresa;
USE empresa;
```

## 0.1 Crear la tabla `departamentos`

```sql
CREATE TABLE departamentos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    ubicacion VARCHAR(100) NOT NULL
);
```

## 0.2 Crear la tabla `empleados`

```sql
CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    salario DECIMAL(10,2) NOT NULL,
    departamento_id INT NOT NULL,

    INDEX idx_departamento (departamento_id),

    CONSTRAINT fk_empleados_departamento
        FOREIGN KEY (departamento_id)
        REFERENCES departamentos(id)
);
```

En esta ocasión se declaró explícitamente:

```sql
INDEX idx_departamento (departamento_id)
```

Esto permite identificar claramente el índice utilizado por la columna que participa en la llave foránea.

## 0.3 Insertar departamentos

```sql
INSERT INTO departamentos (nombre, ubicacion) VALUES
('Recursos Humanos', 'Planta 1'),
('Ventas', 'Planta 2'),
('TI', 'Planta 3'),
('Producción', 'Planta 2');
```

## 0.4 Insertar empleados

```sql
INSERT INTO empleados (nombre, apellido, salario, departamento_id) VALUES
('Ana',     'López',      45000, 1),
('Juan',    'Pérez',      50000, 2),
('Carlos',  'García',     60000, 3),
('María',   'Hernández',  52000, 2),
('Luis',    'Martínez',   41000, 1),
('Sofía',   'Ramírez',    68000, 3),
('Diego',   'Torres',     47000, 4),
('Laura',   'Pérez',      55000, 4),
('Miguel',  'Sánchez',    39000, 1),
('Elena',   'Gómez',      72000, 3),
('Andrés',  'Rivera',     49000, 2),
('Paola',   'Díaz',       51000, 4);
```

Comprueba los datos:

```sql
SELECT * FROM departamentos;
SELECT * FROM empleados;
```

---

# 1. Índices: qué son y qué significan los indicadores de MySQL

## Objetivo

Comprender qué es un índice, cómo identificar los índices existentes y cómo interpretar los resultados de `DESCRIBE` y `SHOW INDEX`.

## 1.1 ¿Qué es realmente un índice?

Un índice es una estructura adicional que MySQL mantiene para localizar registros con mayor rapidez.

No debe imaginarse como una columna nueva agregada a la tabla.

Por ejemplo, la tabla puede contener:

```text
empleados
+----+--------+----------+
| id | nombre | apellido |
+----+--------+----------+
| 1  | Ana    | López    |
| 2  | Juan   | Pérez    |
| 3  | Carlos | García   |
+----+--------+----------+
```

Mientras que un índice sobre `apellido` puede imaginarse, de manera simplificada, como una estructura separada:

```text
idx_apellido
García  ---> registro 3
López   ---> registro 1
Pérez   ---> registro 2
```

En tablas `InnoDB`, los índices comunes se implementan normalmente mediante estructuras tipo B-tree.

El objetivo es evitar, cuando resulte conveniente, revisar una por una todas las filas de una tabla.

---

## 1.2 Observar primero la estructura actual de la tabla

Ejecuta:

```sql
DESCRIBE empleados;
```

De forma aproximada aparecerá algo semejante a:

```text
+-----------------+---------------+------+-----+---------+----------------+
| Field           | Type          | Null | Key | Default | Extra          |
+-----------------+---------------+------+-----+---------+----------------+
| id              | int           | NO   | PRI | NULL    | auto_increment |
| nombre          | varchar(50)   | NO   |     | NULL    |                |
| apellido        | varchar(50)   | NO   |     | NULL    |                |
| salario         | decimal(10,2) | NO   |     | NULL    |                |
| departamento_id | int           | NO   | MUL | NULL    |                |
+-----------------+---------------+------+-----+---------+----------------+
```

### ¿Qué significa la columna `Key`?

Los valores más importantes son:

| Indicador | Significado |
|---|---|
| `PRI` | La columna pertenece a la llave primaria (`PRIMARY KEY`). |
| `UNI` | La columna inicia un índice `UNIQUE`; los valores deben ser únicos, salvo las reglas aplicables a `NULL`. |
| `MUL` | La columna inicia un índice no único; un mismo valor puede aparecer varias veces. |
| vacío | La columna no aparece como primera columna de uno de esos índices. |

### Entonces, ¿por qué `departamento_id` aparece como `MUL`?

Porque `departamento_id` tiene un índice no único llamado:

```text
idx_departamento
```

Es lógico que sea no único: varios empleados pueden pertenecer al mismo departamento.

> **Muy importante:** `MUL` **no significa “llave foránea”**.  
> `MUL` indica que existe un índice no único sobre esa columna. En este caso ese índice también sirve para la relación de llave foránea.

Para observar la llave foránea de forma explícita utiliza:

```sql
SHOW CREATE TABLE empleados;
```

Busca una parte semejante a:

```sql
CONSTRAINT `fk_empleados_departamento`
FOREIGN KEY (`departamento_id`)
REFERENCES `departamentos` (`id`)
```

---

## 1.3 Ver información detallada de los índices

Ejecuta:

```sql
SHOW INDEX FROM empleados;
```

La salida real contiene varias columnas. Las más importantes para esta práctica son:

| Campo | ¿Qué indica? |
|---|---|
| `Key_name` | Nombre del índice. |
| `Column_name` | Columna incluida en el índice. |
| `Non_unique` | `0` si no permite duplicados; `1` si permite valores repetidos. |
| `Seq_in_index` | Posición de la columna dentro del índice. |
| `Cardinality` | Estimación de cuántos valores diferentes contiene el índice. |
| `Index_type` | Tipo de estructura utilizada, normalmente `BTREE`. |
| `Visible` | Indica si el índice está disponible para el optimizador. |

Con la tabla recién creada deben existir, como mínimo:

```text
PRIMARY            -> id
idx_departamento   -> departamento_id
```

Una representación simplificada sería:

```text
+------------------+-----------------+------------+
| Key_name         | Column_name     | Non_unique |
+------------------+-----------------+------------+
| PRIMARY          | id              | 0          |
| idx_departamento | departamento_id | 1          |
+------------------+-----------------+------------+
```

> **No esperes que `Cardinality` tenga exactamente el mismo valor en todas las computadoras.**  
> Es una estimación basada en estadísticas del optimizador y puede cambiar.

---

## 1.4 Crear un índice sobre `apellido`

Ejecuta:

```sql
CREATE INDEX idx_apellido
ON empleados(apellido);
```

Comprueba nuevamente:

```sql
SHOW INDEX FROM empleados;
```

Ahora debe aparecer también:

```text
idx_apellido -> apellido
```

Vuelve a ejecutar:

```sql
DESCRIBE empleados;
```

La columna `apellido` debe aparecer ahora con el indicador `MUL`, porque el índice permite apellidos repetidos.

Por ejemplo, en nuestra tabla existen dos personas con apellido `Pérez`.

---

## 1.5 Utilizar la columna indexada

Ejecuta:

```sql
SELECT *
FROM empleados
WHERE apellido = 'Pérez';
```

El resultado debe mostrar a los empleados cuyo apellido sea `Pérez`.

> En este momento **no vamos a afirmar que la consulta sea más rápida**.  
> La tabla es demasiado pequeña para demostrar rendimiento de forma confiable. En la sección 5 se comprobará correctamente cuándo MySQL decide utilizar un índice.

---

## Ejercicios de la sección 1

### Ejercicio 1

Ejecuta:

```sql
DESCRIBE empleados;
```

Explica con tus palabras:

1. ¿Por qué `id` muestra `PRI`?
2. ¿Por qué `departamento_id` muestra `MUL`?
3. Después de crear `idx_apellido`, ¿por qué `apellido` también muestra `MUL`?
4. ¿`MUL` significa que la columna es una llave foránea?

### Ejercicio 2

Ejecuta:

```sql
SHOW INDEX FROM empleados;
```

Identifica:

- el nombre de cada índice;
- la columna que utiliza;
- cuál es único y cuál permite valores repetidos;
- qué tipo de índice aparece en `Index_type`.

### Ejercicio 3

Elimina temporalmente el índice de apellido:

```sql
DROP INDEX idx_apellido ON empleados;
```

Comprueba:

```sql
SHOW INDEX FROM empleados;
DESCRIBE empleados;
```

¿Qué cambió?

Después, vuelve a crearlo para continuar la práctica:

```sql
CREATE INDEX idx_apellido
ON empleados(apellido);
```

### Ejercicio 4

Ejecuta:

```sql
SHOW CREATE TABLE empleados;
```

Localiza por separado:

- la llave primaria;
- el índice `idx_departamento`;
- la llave foránea `fk_empleados_departamento`;
- el índice `idx_apellido`.

Explica por qué **índice** y **llave foránea** no son exactamente el mismo concepto.

---

# 2. Subconsultas

## Objetivo

Utilizar consultas internas para producir valores que posteriormente serán utilizados por una consulta externa.

Una subconsulta puede verse de esta manera:

```sql
SELECT ...
FROM ...
WHERE columna > (
    SELECT ...
);
```

Primero se obtiene el resultado de la consulta interna y después ese resultado participa en la consulta externa.

---

## 2.1 Empleados con salario superior al promedio general

Primero observa el promedio:

```sql
SELECT AVG(salario) AS salario_promedio
FROM empleados;
```

Después utiliza ese cálculo dentro de otra consulta:

```sql
SELECT nombre, apellido, salario
FROM empleados
WHERE salario > (
    SELECT AVG(salario)
    FROM empleados
);
```

La consulta interna:

```sql
SELECT AVG(salario)
FROM empleados;
```

produce un solo valor.

La consulta externa compara el salario de cada empleado contra ese valor.

---

## 2.2 Subconsulta con `IN`

Obtén los empleados que pertenecen a departamentos ubicados en `Planta 2`:

```sql
SELECT nombre, apellido, departamento_id
FROM empleados
WHERE departamento_id IN (
    SELECT id
    FROM departamentos
    WHERE ubicacion = 'Planta 2'
);
```

Primero la subconsulta obtiene los identificadores de los departamentos ubicados en `Planta 2`.

Después `IN` pregunta si el `departamento_id` de cada empleado pertenece a ese conjunto.

---

## 2.3 Subconsulta correlacionada

Ahora encuentra empleados cuyo salario sea mayor que el promedio de **su propio departamento**:

```sql
SELECT
    e.nombre,
    e.apellido,
    e.salario,
    e.departamento_id
FROM empleados AS e
WHERE e.salario > (
    SELECT AVG(e2.salario)
    FROM empleados AS e2
    WHERE e2.departamento_id = e.departamento_id
);
```

Esta subconsulta es diferente porque depende de la fila que está siendo evaluada en la consulta externa.

---

## Ejercicios de la sección 2

1. Obtén el empleado o empleados que tienen el salario máximo utilizando una subconsulta.
2. Obtén los empleados que pertenecen a departamentos ubicados en `Planta 3`.
3. Obtén los empleados cuyo salario sea menor al salario promedio general.
4. Explica la diferencia entre una subconsulta normal y una subconsulta correlacionada.

---

# 3. Vistas

## Objetivo

Crear consultas reutilizables mediante vistas.

Una vista puede entenderse como una consulta almacenada a la que posteriormente se puede consultar como si fuera una tabla.

---

## 3.1 Crear una vista de empleados y departamentos

```sql
CREATE VIEW vista_empleados_departamentos AS
SELECT
    e.id,
    e.nombre,
    e.apellido,
    d.nombre AS departamento
FROM empleados AS e
JOIN departamentos AS d
    ON e.departamento_id = d.id;
```

Consulta la vista:

```sql
SELECT *
FROM vista_empleados_departamentos;
```

---

## 3.2 Filtrar una vista

```sql
SELECT *
FROM vista_empleados_departamentos
WHERE departamento = 'TI';
```

El estudiante no tiene que volver a escribir el `JOIN`.

---

## 3.3 Modificar la vista

Utiliza `CREATE OR REPLACE VIEW` para agregar el salario y la ubicación:

```sql
CREATE OR REPLACE VIEW vista_empleados_departamentos AS
SELECT
    e.id,
    e.nombre,
    e.apellido,
    e.salario,
    d.nombre AS departamento,
    d.ubicacion
FROM empleados AS e
JOIN departamentos AS d
    ON e.departamento_id = d.id;
```

Comprueba:

```sql
SELECT *
FROM vista_empleados_departamentos;
```

---

## Ejercicios de la sección 3

1. Consulta desde la vista únicamente a los empleados de `Planta 2`.
2. Muestra desde la vista solo `nombre`, `apellido`, `departamento` y `salario`.
3. Ordena los resultados de mayor a menor salario.
4. Explica qué ventaja tiene una vista cuando una consulta con `JOIN` se utiliza repetidamente.

---

# 4. Transacciones: `COMMIT` y `ROLLBACK`

## Objetivo

Comprobar de manera visible la diferencia entre confirmar y deshacer una operación.

Una transacción permite agrupar operaciones para decidir posteriormente si los cambios deben conservarse.

```text
START TRANSACTION
       |
       v
 operaciones
       |
   +---+---+
   |       |
COMMIT  ROLLBACK
guardar  deshacer
```

---

## 4.1 Probar `ROLLBACK`

Comprueba primero que el registro no existe:

```sql
SELECT *
FROM empleados
WHERE apellido = 'Rollback';
```

Inicia la transacción:

```sql
START TRANSACTION;
```

Inserta un empleado:

```sql
INSERT INTO empleados
(nombre, apellido, salario, departamento_id)
VALUES
('Empleado', 'Rollback', 48000, 2);
```

Antes de revertir, comprueba que el registro puede verse dentro de la sesión:

```sql
SELECT *
FROM empleados
WHERE apellido = 'Rollback';
```

Ahora ejecuta:

```sql
ROLLBACK;
```

Comprueba otra vez:

```sql
SELECT *
FROM empleados
WHERE apellido = 'Rollback';
```

Después del `ROLLBACK`, el registro ya no debe existir.

---

## 4.2 Probar `COMMIT`

Inicia otra transacción:

```sql
START TRANSACTION;
```

Inserta:

```sql
INSERT INTO empleados
(nombre, apellido, salario, departamento_id)
VALUES
('Empleado', 'Commit', 53000, 3);
```

Confirma el cambio:

```sql
COMMIT;
```

Comprueba:

```sql
SELECT *
FROM empleados
WHERE apellido = 'Commit';
```

Ahora el registro debe permanecer almacenado.

---

## 4.3 Actividad adicional: modificar y deshacer

Observa primero el salario de Juan Pérez:

```sql
SELECT nombre, apellido, salario
FROM empleados
WHERE nombre = 'Juan'
  AND apellido = 'Pérez';
```

Ejecuta:

```sql
START TRANSACTION;

UPDATE empleados
SET salario = 90000
WHERE nombre = 'Juan'
  AND apellido = 'Pérez';
```

Comprueba el cambio:

```sql
SELECT nombre, apellido, salario
FROM empleados
WHERE nombre = 'Juan'
  AND apellido = 'Pérez';
```

Después:

```sql
ROLLBACK;
```

Consulta nuevamente el salario y comprueba que regresó al valor anterior.

---

## Ejercicios de la sección 4

1. Inserta un empleado dentro de una transacción y cancela la operación con `ROLLBACK`.
2. Inserta otro empleado y conserva el cambio con `COMMIT`.
3. Modifica un salario y después revierte la modificación.
4. Explica con tus palabras en qué situación utilizarías una transacción.

---

# 5. Optimización de consultas con `EXPLAIN`

## Objetivo

Comprender qué información proporciona `EXPLAIN` y comprobar la diferencia entre:

- que un índice **exista**;
- que un índice sea **candidato** para una consulta;
- que MySQL realmente **decida utilizarlo**.

Esta diferencia es fundamental.

---

## 5.1 ¿Qué hace `EXPLAIN`?

`EXPLAIN` muestra el plan que el optimizador de MySQL pretende utilizar para ejecutar una consulta.

Ejemplo:

```sql
EXPLAIN
SELECT *
FROM empleados
WHERE apellido = 'Pérez';
```

Los campos más importantes para esta práctica son:

| Campo | Interpretación |
|---|---|
| `table` | Tabla que MySQL está analizando. |
| `type` | Método de acceso utilizado. |
| `possible_keys` | Índices que podrían servir para la consulta. |
| `key` | Índice que MySQL decidió utilizar realmente. |
| `rows` | Estimación de filas que tendrá que examinar. |
| `filtered` | Porcentaje estimado que sobrevivirá al filtro. |
| `Extra` | Información adicional sobre el plan. |

### Valores de `type` que veremos con mayor frecuencia

| `type` | Interpretación simplificada |
|---|---|
| `ALL` | Recorrido completo de la tabla. |
| `ref` | Búsqueda mediante un índice no único. |
| `range` | Recorrido de un intervalo dentro de un índice. |
| `const` | Acceso muy específico, normalmente mediante una llave primaria o índice único. |

---

## 5.2 Un resultado que puede confundir: tablas pequeñas

Ya existe:

```sql
idx_apellido
```

Compruébalo:

```sql
SHOW INDEX FROM empleados;
```

Ahora ejecuta:

```sql
EXPLAIN
SELECT *
FROM empleados
WHERE apellido = 'Pérez';
```

Podrían ocurrir dos situaciones:

### Situación A

MySQL decide utilizar `idx_apellido`.

Entonces `key` mostrará:

```text
idx_apellido
```

### Situación B

MySQL decide realizar un recorrido completo.

Entonces podría aparecer:

```text
type = ALL
key  = NULL
```

**Esto no significa que el índice esté mal creado.**

Nuestra tabla contiene muy pocos registros. Para una tabla pequeña, MySQL puede calcular que resulta más barato leerla completa que entrar al índice y después localizar las filas.

Por esta razón, **no es correcto demostrar la utilidad de los índices utilizando únicamente una tabla de 10 o 12 registros**.

Ahora realizaremos una prueba con una tabla mayor.

---

# 5.3 Crear una tabla específica para probar rendimiento

Esta tabla se utilizará únicamente para observar el comportamiento del optimizador.

```sql
DROP TABLE IF EXISTS empleados_prueba;

CREATE TABLE empleados_prueba (
    id INT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    salario DECIMAL(10,2) NOT NULL
);
```

Generaremos 1000 empleados de prueba.

> El objetivo de este bloque no es estudiar todavía las expresiones comunes de tabla (`CTE`).  
> Se utiliza únicamente como una forma rápida de generar muchos registros.

```sql
INSERT INTO empleados_prueba
(id, nombre, apellido, salario)

WITH RECURSIVE numeros AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numeros
    WHERE n < 1000
)

SELECT
    n,
    CONCAT('Empleado_', n),
    CONCAT('Apellido_', n),
    30000 + MOD(n * 137, 50000)
FROM numeros;
```

Comprueba:

```sql
SELECT COUNT(*) AS total_registros
FROM empleados_prueba;
```

El resultado debe ser:

```text
1000
```

Actualiza las estadísticas utilizadas por el optimizador:

```sql
ANALYZE TABLE empleados_prueba;
```

---

# 5.4 Prueba 1: consulta SIN índice

Comprueba primero los índices existentes:

```sql
SHOW INDEX FROM empleados_prueba;
```

Debe existir únicamente el índice asociado a la llave primaria:

```text
PRIMARY -> id
```

Ahora analiza una búsqueda por apellido:

```sql
EXPLAIN
SELECT *
FROM empleados_prueba
WHERE apellido = 'Apellido_777';
```

Como todavía no existe un índice para `apellido`, normalmente se observará algo parecido a:

```text
type          = ALL
possible_keys = NULL
key           = NULL
rows          ≈ 1000
```

La interpretación es:

> MySQL no tiene un índice útil para `apellido`, por lo que debe considerar un recorrido completo de la tabla.

---

# 5.5 Prueba 2: crear el índice

Crea:

```sql
CREATE INDEX idx_prueba_apellido
ON empleados_prueba(apellido);
```

Comprueba que existe:

```sql
SHOW INDEX FROM empleados_prueba;
```

Actualiza las estadísticas:

```sql
ANALYZE TABLE empleados_prueba;
```

Ejecuta exactamente la misma consulta:

```sql
EXPLAIN
SELECT *
FROM empleados_prueba
WHERE apellido = 'Apellido_777';
```

Ahora es razonable observar algo semejante a:

```text
type          = ref
possible_keys = idx_prueba_apellido
key           = idx_prueba_apellido
rows          ≈ 1
```

### ¿Qué cambió?

Antes:

```text
Sin índice
1000 registros
      |
      v
revisar muchas filas
```

Después:

```text
Con índice
Apellido_777
      |
      v
idx_prueba_apellido
      |
      v
registro correspondiente
```

Lo importante no es memorizar números exactos.

Lo importante es comparar:

```text
possible_keys
key
type
rows
```

---

# 5.6 Diferencia entre `possible_keys` y `key`

Supongamos que aparece:

```text
possible_keys = idx_prueba_apellido
key           = NULL
```

Eso significa:

1. MySQL reconoce que existe un índice que **podría** utilizar.
2. Sin embargo, el optimizador decidió que para esa consulta concreta existe un plan más conveniente.

Por tanto:

```text
possible_keys = índice candidato
key           = índice realmente utilizado
```

Esta diferencia es una de las partes más importantes de `EXPLAIN`.

---

# 5.7 Un índice no siempre debe utilizarse

Crea un índice de salario:

```sql
CREATE INDEX idx_prueba_salario
ON empleados_prueba(salario);
```

Actualiza estadísticas:

```sql
ANALYZE TABLE empleados_prueba;
```

Prueba una condición muy amplia:

```sql
EXPLAIN
SELECT *
FROM empleados_prueba
WHERE salario > 30000;
```

Como una gran cantidad de registros cumple la condición, MySQL puede decidir que usar el índice no ofrece una ventaja suficiente.

Ahora prueba una condición mucho más específica:

```sql
EXPLAIN
SELECT *
FROM empleados_prueba
WHERE salario BETWEEN 70000 AND 70100;
```

Compara:

- `type`;
- `possible_keys`;
- `key`;
- `rows`.

La utilidad de un índice depende también de qué tan **selectiva** sea la condición.

---

# 5.8 `EXPLAIN ANALYZE` — actividad opcional

`EXPLAIN` muestra estimaciones.

En versiones modernas de MySQL también puede utilizarse:

```sql
EXPLAIN ANALYZE
SELECT *
FROM empleados_prueba
WHERE apellido = 'Apellido_777';
```

A diferencia de `EXPLAIN`, `EXPLAIN ANALYZE` **sí ejecuta la consulta** y muestra información adicional, como:

- filas estimadas;
- filas realmente obtenidas;
- tiempo de ejecución;
- número de ciclos o `loops`.

Para esta práctica utilízalo solamente con consultas `SELECT`.

---

## Ejercicios de la sección 5

### Ejercicio 1

Ejecuta:

```sql
EXPLAIN
SELECT *
FROM empleados
WHERE apellido = 'Pérez';
```

Anota:

- `type`;
- `possible_keys`;
- `key`;
- `rows`.

Explica por qué MySQL puede decidir no utilizar el índice en una tabla pequeña.

### Ejercicio 2

En `empleados_prueba`, ejecuta la búsqueda por:

```text
Apellido_500
```

antes y después de crear un índice de apellido.

Compara los campos de `EXPLAIN`.

### Ejercicio 3

Responde:

1. ¿Qué diferencia existe entre `possible_keys` y `key`?
2. ¿Qué significa `type = ALL`?
3. ¿Qué representa `rows`?
4. ¿Por qué `rows` no debe interpretarse siempre como un número exacto?
5. ¿Tener un índice obliga a MySQL a utilizarlo?

### Ejercicio 4

Compara:

```sql
EXPLAIN
SELECT *
FROM empleados_prueba
WHERE salario > 30000;
```

con:

```sql
EXPLAIN
SELECT *
FROM empleados_prueba
WHERE salario BETWEEN 70000 AND 70100;
```

Explica por qué el mismo índice puede resultar más útil para una consulta que para otra.

---

# 6. Identificación y depuración de errores

## Objetivo

Aprender a interpretar errores comunes de MySQL y corregir la causa del problema.

En esta sección **se espera que algunas instrucciones fallen intencionalmente**.

Lee el mensaje de MySQL antes de corregir la consulta.

---

## 6.1 Error de sintaxis

Ejecuta:

```sql
SELECT *
FORM empleados;
```

Localiza el problema.

Consulta correcta:

```sql
SELECT *
FROM empleados;
```

---

## 6.2 Columna inexistente

Ejecuta:

```sql
SELECT nombre, puesto
FROM empleados;
```

La columna `puesto` no existe.

Comprueba la estructura:

```sql
DESCRIBE empleados;
```

Corrige la consulta utilizando columnas que sí existan.

---

## 6.3 Violación de llave primaria

Ejecuta:

```sql
INSERT INTO empleados
(id, nombre, apellido, salario, departamento_id)
VALUES
(1, 'Prueba', 'Duplicada', 40000, 1);
```

El `id = 1` ya existe y la llave primaria no permite duplicados.

La forma normal de insertar un empleado es permitir que `AUTO_INCREMENT` genere el identificador:

```sql
INSERT INTO empleados
(nombre, apellido, salario, departamento_id)
VALUES
('Prueba', 'Correcta', 40000, 1);
```

---

## 6.4 Violación de llave foránea

Ejecuta:

```sql
INSERT INTO empleados
(nombre, apellido, salario, departamento_id)
VALUES
('Empleado', 'DepartamentoInexistente', 45000, 999);
```

El departamento `999` no existe.

Comprueba:

```sql
SELECT *
FROM departamentos;
```

Después realiza la inserción utilizando un `departamento_id` válido.

---

## Ejercicios de la sección 6

Para cada error:

1. copia el mensaje principal mostrado por MySQL;
2. identifica qué parte de la instrucción produjo el error;
3. escribe la consulta corregida;
4. explica qué regla de la base de datos se estaba violando.

---

# 7. Actividad integradora

Realiza las siguientes actividades sin copiar directamente los ejemplos anteriores.

## Parte A — Índices

1. Crea un índice llamado `idx_nombre` para la columna `nombre` de `empleados`.
2. Comprueba su existencia con `SHOW INDEX`.
3. Comprueba qué indicador aparece ahora en `DESCRIBE empleados`.
4. Elimina `idx_nombre`.
5. Comprueba que desapareció.

## Parte B — Subconsulta

Obtén los empleados que ganan más que el salario promedio de los empleados del departamento de `Ventas`.

## Parte C — Vista

Crea una vista llamada:

```text
vista_salarios_altos
```

que muestre:

- nombre;
- apellido;
- salario;
- departamento;

únicamente para empleados con salario mayor a `50000`.

## Parte D — Transacción

1. Inicia una transacción.
2. Aumenta en `5000` el salario de un empleado.
3. Comprueba el nuevo valor.
4. Ejecuta `ROLLBACK`.
5. Comprueba que el salario regresó a su valor original.

## Parte E — Optimización

En `empleados_prueba`:

1. elimina `idx_prueba_apellido`;
2. ejecuta `EXPLAIN` para buscar `Apellido_900`;
3. registra los valores de `type`, `possible_keys`, `key` y `rows`;
4. vuelve a crear `idx_prueba_apellido`;
5. ejecuta `ANALYZE TABLE empleados_prueba`;
6. repite el mismo `EXPLAIN`;
7. realiza una comparación de los resultados.

---

# 8. Entregable

El reporte de la práctica debe incluir:

1. captura o evidencia de `DESCRIBE empleados`;
2. captura o evidencia de `SHOW INDEX FROM empleados`;
3. explicación de `PRI`, `UNI` y `MUL`;
4. resultado de al menos una subconsulta;
5. consulta y resultado de la vista;
6. evidencia del funcionamiento de `ROLLBACK`;
7. evidencia del funcionamiento de `COMMIT`;
8. comparación de `EXPLAIN` antes y después de crear el índice en `empleados_prueba`;
9. respuesta a las preguntas de interpretación de la sección 5;
10. evidencia de al menos dos errores intencionales y su corrección.

---

# 9. Preguntas de cierre

Contesta con tus propias palabras:

1. ¿Qué problema intenta resolver un índice?
2. ¿Cuál es la diferencia entre una llave primaria, una llave foránea y un índice?
3. ¿Qué significa `MUL` dentro de `DESCRIBE`?
4. ¿Por qué una llave foránea suele necesitar un índice?
5. ¿Qué diferencia existe entre `possible_keys` y `key` en `EXPLAIN`?
6. ¿Por qué MySQL podría ignorar un índice en una tabla pequeña?
7. ¿Qué diferencia existe entre `COMMIT` y `ROLLBACK`?
8. ¿Qué ventaja proporciona una vista?
9. ¿Qué problema resuelve una subconsulta?
10. ¿Por qué no es recomendable crear índices indiscriminadamente en todas las columnas?

---

# Referencias oficiales de MySQL

- `DESCRIBE` / `SHOW COLUMNS`:  
  https://dev.mysql.com/doc/refman/8.4/en/show-columns.html

- `SHOW INDEX`:  
  https://dev.mysql.com/doc/refman/8.4/en/show-index.html

- Optimización e índices:  
  https://dev.mysql.com/doc/refman/8.4/en/optimization-indexes.html

- Uso de índices:  
  https://dev.mysql.com/doc/refman/8.4/en/mysql-indexes.html

- Recorridos completos de tablas (`type = ALL`):  
  https://dev.mysql.com/doc/refman/8.4/en/table-scan-avoidance.html

- `EXPLAIN` y `EXPLAIN ANALYZE`:  
  https://dev.mysql.com/doc/refman/8.4/en/explain.html

- Llaves foráneas:  
  https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html
