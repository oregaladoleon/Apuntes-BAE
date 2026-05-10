### Enunciado:
Simulacro de Examen: Clínica "San Lorenzo"
Requisitos del Modelo Relacional:

    Paciente (paciente)

        id: SERIAL PRIMARY KEY.

        nombre: Texto, no nulo.

        ciudad: Texto, por defecto 'Santander'.

    Medico (medico)

        id: SERIAL PRIMARY KEY.

        nombre: Texto, no nulo.

        especialidad: Texto (CHECK: 'Pediatría', 'Traumatología', 'General').

    Cita (cita) (Intermedia N:M entre Paciente y Médico)

        id_paciente: FK a paciente (CASCADE).

        id_medico: FK a medico (RESTRICT).

        fecha: DATE.

        coste: NUMERIC(6,2) (Debe ser > 0).

        PK compuesta: (id_paciente, id_medico, fecha).

### Resolución:
Creación de tablas:
~~~sql
CREATE TABLE paciente(
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre TEXT NOT NULL,
    ciudad TEXT DEFAULT 'Santander'
);
CREATE TABLE medico (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre TEXT NOT NULL,
    especialidad TEXT,
    CONSTRAINT especialidad_chk CHECK (especialidad IN ('Pediatría','Traumatología','General'))
);
CREATE TABLE cita (
    id_paciente INT,
    id_medico INT,
    fecha DATE,
    coste NUMERIC(6,2),
    CONSTRAINT id_paciente_medico_pk PRIMARY KEY (id_paciente, id_medico, fecha),
    CONSTRAINT id_paciente_fk FOREIGN KEY (id_paciente)
    REFERENCES paciente(id)
    ON UPDATE CASCADE
    ON DELETE CASCADE,
    CONSTRAINT id_medico_fk FOREIGN KEY (id_medico)
    REFERENCES medico(id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,
    CONSTRAINT coste_chk CHECK (coste > 0)
);
~~~

Registros:

~~~sql
INSERT INTO paciente (nombre, ciudad) VALUES 
('Óscar Regalado', 'Santander'), -- Tendrá cita
('Ana López', 'Santander'),      -- NO tendrá cita (Para el EXCEPT)
('Luis García', 'Bilbao');

INSERT INTO medico (nombre, especialidad) VALUES 
('Dr. House', 'General'),
('Dra. Foster', 'Pediatría'),
('Dr. Strange', 'Traumatología'); -- NO tendrá cita (Para el LEFT JOIN)

INSERT INTO cita (id_paciente, id_medico, fecha, coste) VALUES 
(1, 1, '2026-05-10', 600.00), -- Genera > 500 para el HAVING
(1, 2, '2026-05-11', 150.00),
(3, 1, '2026-05-12', 100.00);
~~~

Consultas:
A: Muestra el nombre del médico y el total de ingresos que ha generado (suma de coste), pero solo de aquellos médicos que hayan generado más de 500€ en total.
~~~sql 
SELECT m.nombre AS "Médico", SUM(c.coste) AS "Ingresos"
FROM medico AS m
INNER JOIN cita AS c ON m.id = c.id_medico
GROUP BY
m.nombre
HAVING
SUM(c.coste) > 500;
~~~

B: Queremos un listado de los nombres de todos los pacientes que viven en Santander pero que NUNCA han tenido una cita.  
~~~sql
SELECT p.nombre  
FROM paciente AS p
WHERE ciudad = 'Santander'

EXCEPT

SELECT p.nombre
FROM paciente AS p
INNER JOIN cita AS c ON p.id = c.id_paciente
;
~~~
C: Muestra todos los médicos de la clínica y la fecha de su última cita. Si un médico no tiene citas, debe aparecer su nombre y un NULL en la fecha.  
~~~sql
SELECT m.nombre AS "Médico", MAX(c.fecha) AS "Fecha cita"
FROM medico AS m
LEFT JOIN cita AS c ON m.id = c.id_medico
GROUP BY
m.nombre
;
~~~

D: Crea un listado único de "Personas en la Clínica" que contenga dos columnas: nombre y tipo (donde tipo sea el texto 'PACIENTE' o 'MEDICO'). Ordena por nombre.

~~~sql

~~~