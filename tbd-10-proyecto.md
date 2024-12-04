### **Proyecto Final: Desarrollo de Sistema para Gestión Biomédica**  

**Objetivo:**  
El proyecto consiste en el desarrollo de un sistema completo que permita gestionar información relacionada con el ámbito biomédico. Los estudiantes deberán aplicar todos los conocimientos adquiridos en las prácticas del curso, además de investigar y agregar funcionalidades adicionales si lo consideran necesario.

---

### **Requerimientos del Proyecto:**  

1. **Usuario personalizado en la base de datos:**  
   - Crear un usuario específico en MySQL con permisos adecuados para el sistema. No se deben utilizar cuentas administrativas.

2. **Gestión de la base de datos:**  
   - Crear al menos una base de datos, tablas con llaves primarias y llaves foráneas (al menos una tabla debe incluir una llave foránea).  
   - Insertar, modificar y eliminar registros.  
   - Modificar las características de las tablas (como agregar o eliminar columnas).

3. **Consultas avanzadas:**  
   - Usar al menos dos funciones de agregación (por ejemplo, `SUM`, `AVG`, `COUNT`, etc.).  
   - Incluir al menos una subconsulta en alguna de las consultas SQL.  
   - Crear y utilizar al menos una vista (view) en la base de datos.  
   - Implementar una transacción que asegure la integridad de los datos en un proceso crítico.

4. **Uso de diferentes tipos de datos:**  
   - Incluir al menos cuatro tipos de datos en las tablas, entre ellos un campo `TIMESTAMP` con la zona horaria correctamente configurada.

5. **Protección y seguridad:**  
   - No exponer usuarios ni contraseñas en el código del servidor. Utilizar variables de entorno con un archivo `.env`.  
   - Proteger todas las rutas del servidor, permitiendo el acceso solo a usuarios autenticados y con el rol adecuado.  
   - Implementar al menos dos tipos de roles de usuario (por ejemplo: administrador y usuario general).

6. **Interfaz de usuario:**  
   - Crear una barra de navegación (menú de opciones) que permita acceder a las diferentes partes del sistema.  
   - Diseñar páginas con estilo agradable e intuitivo.  
   - Agregar estilos personalizados a las páginas de confirmación (mensajes de éxito o error).  
   - Incluir funcionalidad de búsqueda en tiempo real (keyup) en algún formulario.

7. **Manejo de archivos:**  
   - Permitir la subida y descarga de al menos dos tipos de archivos (por ejemplo, Excel y PDF), tanto al servidor como a la base de datos.

8. **Funcionalidad del servidor:**  
   - Utilizar nodemon u otra herramienta similar para reinicio automático del servidor durante el desarrollo.

9. **Características adicionales:**  
   - Se alienta a los estudiantes a agregar características que consideren relevantes para el sistema biomédico que estén desarrollando.  
   - Pueden investigar y agregar funcionalidades adicionales que estén fuera del alcance del curso si lo consideran útil.

---

### **Entregables (2 de 3 opciones): (puntos extra para quien realice las 3 opciones)**  

1. **Sistema completo:**  
   - Subir el proyecto a una cuenta de gidhub.

2. **Reporte del proyecto agregado a la bitacora:**  
   - Incluir un documento en la bitácora de la materia.

3. **Video del funcionamiento:**  
   - Grabar un video de máximo 5 minutos mostrando el funcionamiento del sistema. No es necesario explicar el código, solo demostrar el uso del sistema.

---

### **Criterios de Evaluación:**

| **Criterio**                      | **Puntos** |
|------------------------------------|------------|
| Usuario personalizado y permisos   | 10         |
| Gestión de base de datos (CRUD)    | 15         |
| Uso de llaves foráneas             | 5          |
| Consultas avanzadas (funciones, subconsulta, vista) | 10 |
| Transacción en base de datos       | 5          |
| Diferentes tipos de datos          | 5          |
| Seguridad (variables de entorno, rutas protegidas) | 10 |
| Roles de usuario                   | 5          |
| Interfaz (barra de navegación, estilos, keyup) | 10 |
| Manejo de archivos                 | 10         |
| Reinicio automático del servidor   | 5          |
| Creatividad y funcionalidades adicionales | 5   |
| Presentación del sistema           | 5          |

**Total:** 100 puntos.

**Puntos extra:**  
- Cantidad y calidad en los entregables y originalidad y uso de sus sistema.

---

### **Instrucciones adicionales:**  

- El proyecto es individual.  
- Se permite el uso de recursos externos, pero todo el código debe ser propio y adaptado al proyecto.  
- La fecha de entrega es **el último día que tenemos marcado en nuestro calendario de evaluaciones**.  
- El proyecto será evaluado durante una presentación en clase donde se explicará brevemente su sistema y responderá preguntas sobre su desarrollo.  
