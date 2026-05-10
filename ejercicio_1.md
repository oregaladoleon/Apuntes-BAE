# Ejercicio 1
> Práctica de examen:
----
### Enunciado:
Debes crear el esquema para una base de datos que gestione los cursos de las escuelas. Estas son las tablas y sus restricciones:

    Escuela (escuela)

        id: Entero, Clave Primaria, Autoincremental.

        nombre: Cadena de texto, no puede ser nulo.

        ciudad: Cadena de texto, por defecto 'Somo'.

    Monitor (monitor)

        id: Entero, Clave Primaria.

        nombre: Texto, obligatorio.

        dni: Texto, único y obligatorio.

        id_escuela: Clave Foránea que referencia a escuela. Si se borra la escuela, los monitores deben quedar con este campo en nulo (SET NULL).

    Curso (curso)

        id: Entero, Clave Primaria.

        nivel: Texto, solo puede ser: 'iniciación', 'perfeccionamiento' o 'competición' (CHECK).

        precio: Numérico (5,2), debe ser mayor que 0.

    Asignacion (asignacion) -- Tabla intermedia que relaciona Monitores y Cursos

        id_monitor: FK a monitor.

        id_curso: FK a curso.

        fecha_inicio: Fecha, no nula.

        PK compuesta: formada por (id_monitor, id_curso, fecha_inicio).

        Restricción de borrado: Si se borra un monitor o un curso, sus asignaciones deben borrarse automáticamente (CASCADE).

Consultas:   
DDL: Escribe el código SQL para crear las 4 tablas con todas las restricciones mencionadas.

Consulta A: Muestra el nombre del monitor y el nivel del curso que tiene asignado. Los nombres de los monitores deben aparecer en mayúsculas (Función UPPER).

Consulta B: Muestra el nombre de la escuela y el precio medio de sus cursos asignados. Agrupa por el nombre de la escuela. (Necesitarás un triple JOIN y la función AVG).

Consulta C: Muestra todos los datos de los cursos cuyo precio sea superior a la media de precios de todos los cursos. (Usa una Subconsulta).

### Resolución:
~~~ sql
CREATE TABLE escuela (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    ciudad VARCHAR(50) NOT NULL DEFAULT 'Somo'
);
CREATE TABLE monitor (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    dni TEXT UNIQUE NOT NULL,
    id_escuela INT,
    CONSTRAINT id_escuela_fk FOREIGN KEY (id_escuela)
    REFERENCES escuela (id)
    ON UPDATE CASCADE
    ON DELETE SET NULL
);
CREATE TABLE curso (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nivel TEXT,
    precio NUMERIC(5,2),
    CONSTRAINT nivel_chk CHECK (nivel IN ('iniciacion','perfeccionamiento','competicion')),
    CONSTRAINT precio_chk CHECK (precio > 0)
);
CREATE TABLE asignacion (
    id_monitor INT,
    id_curso INT,
    fecha_inicio DATE NOT NULL,
    CONSTRAINT id_monitor_fk FOREIGN KEY (id_monitor)
    REFERENCES monitor (id)
    ON UPDATE CASCADE
    ON DELETE CASCADE,
    CONSTRAINT id_curso_fk FOREIGN KEY (id_curso)
    REFERENCES curso(id)
    ON UPDATE CASCADE
    ON DELETE CASCADE,
    CONSTRAINT id_monitor_curso_fecha_pk PRIMARY KEY (id_monitor,id_curso,fecha_inicio)
);
~~~
Introducción de valores:
~~~sql
-- 1. Insertar Escuelas
INSERT INTO escuela (nombre, ciudad) VALUES 
('Somo Surf School', 'Somo'),
('Zarautz Academy', 'Zarautz'),
('Lanzarote Pro', 'Famara');

-- 2. Insertar Monitores (vinculados a escuelas)
INSERT INTO monitor (nombre, dni, id_escuela) VALUES 
('Aritz Aranburu', '11111111A', 2),
('Gony Zubizarreta', '22222222B', 1),
('Lucia Martiño', '33333333C', 1),
('Kepa Acero', '44444444D', 3);

-- 3. Insertar Cursos
INSERT INTO curso (nivel, precio) VALUES 
('iniciacion', 50.00),
('perfeccionamiento', 85.50),
('competicion', 120.00),
('iniciacion', 45.00);

-- 4. Insertar Asignaciones (el puente entre monitores y cursos)
-- Vamos a asignar a los monitores de Somo y Zarautz a varios cursos
INSERT INTO asignacion (id_monitor, id_curso, fecha_inicio) VALUES 
(1, 3, '2024-06-01'), -- Aritz en Competición
(2, 1, '2024-06-05'), -- Gony en Iniciación
(3, 2, '2024-06-10'), -- Lucia en Perfeccionamiento
(2, 2, '2024-07-01'), -- Gony repite en Perfeccionamiento
(4, 4, '2024-06-15'); -- Kepa en Iniciación (precio barato)
~~~
Consulta A: Muestra el nombre del monitor y el nivel del curso que tiene asignado. Los nombres de los monitores deben aparecer en mayúsculas (Función UPPER).

~~~sql
SELECT UPPER(m.nombre) AS "Monitor", c.nivel 
FROM monitor AS m
INNER JOIN asignacion AS a ON a.id_monitor = m.id
INNER JOIN curso AS c ON a.id_curso = c.id 
;
~~~
Consulta B: Muestra el nombre de la escuela y el precio medio de sus cursos asignados. Agrupa por el nombre de la escuela. (Necesitarás un triple JOIN y la función AVG).
~~~sql
SELECT e.nombre AS "Escuela", ROUND(AVG(c.precio), 2) AS "Precio medio"
FROM escuela AS e
INNER JOIN monitor AS m ON m.id_escuela = e.id
INNER JOIN asignacion AS a ON a.id_monitor = m.id
INNER JOIN curso AS c ON c.id = a.id_curso
GROUP BY
e.nombre
;
~~~
Consulta C: Muestra todos los datos de los cursos cuyo precio sea superior a la media de precios de todos los cursos. (Usa una Subconsulta).
~~~sql
SELECT c.id, c.nivel, c.precio 
FROM curso AS c
WHERE c.precio > (SELECT AVG(c2.precio) FROM curso AS c2);
~~~
Consulta D: Muestra los cursos y el nombre del monitor, pero incluye también los cursos que no tienen a nadie asignado todavía"
~~~sql
SELECT c.id AS "Curso", m.nombre AS "Monitor"
FROM curso AS c
LEFT JOIN asignacion AS a ON c.id = a.id_curso
LEFT JOIN monitor AS m ON a.id_monitor = m.id
;
~~~