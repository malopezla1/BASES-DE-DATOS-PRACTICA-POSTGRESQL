# CONSULTAS AVANZADAS
En este apartado, resolveremos algunos ejercicios a cerca de aplicar consultas avanzadas usando joins.

## Nivel 1 — JOIN

1. Listar el nombre completo de cada estudiante y el nombre de su carrera.
```sql
SELECT 
	e.nombre_completo AS "estudiante",
	c.nombre AS "carrera"
FROM estudiante e 
INNER JOIN carrera c ON e.cod_carrera = c.cod_carrera;
```
2. Listar todas las materias con el nombre de la carrera a la que pertenecen.
```sql
SELECT 
	mat.nombre AS "materia",
	c.nombre AS "carrera"
FROM materias mat 
INNER JOIN carrera c ON mat.facultad = c.facultad;
```
3. Mostrar el nombre del estudiante, la materia que cursó y la nota obtenida.
```sql
SELECT 
	e.nombre_completo AS "estudiante",
	mat.nombre AS "materia",
	m.nota_final AS "nota"
FROM estudiante e
INNER JOIN matricula m ON e.cod_estudiante = m.cod_estudiante 
INNER JOIN materias mat ON m.cod_materia = mat.cod_materia;
```
4. Mostrar el nombre del estudiante, la materia, el profesor que la dictó y la nota.
```sql
SELECT
	p.nombre AS "profesor",
	e.nombre_completo AS "estudiante",
	mat.nombre AS "materia",
	m.nota_final AS "nota"
FROM estudiante e
INNER JOIN matricula m ON e.cod_estudiante = m.cod_estudiante 
INNER JOIN materias mat ON m.cod_materia = mat.cod_materia
INNER JOIN profesor p ON mat.facultad = p.facultad;
```
Aqui encontramos nuestro primer inconveniente, ya que en la base estructurada no tenemos definido que profesor da clase, por lo cual tendremos que arreglarlo, ademas podemos agregar mas profesores para dejar un maestro por materia o hacer que repitan, lo cual seria una solución a medias teniendo en cuenta que solo sea por un semestre esta base y que un profesor no repita materias si es que hay otros que tambien las dan, seria una tabla que necesitaria normalizarse a su forma 3FN, con lo cual podriamos crear una tabla nueva que cubra estos problemas.
```sql
ALTER TABLE materias 
ADD COLUMN cedula_profesor INT,
ADD CONSTRAINT fk_materias_profesor 
    FOREIGN KEY (cedula_profesor) REFERENCES profesor(cedula);

UPDATE materias SET cedula_profesor = 1050607920 WHERE cod_materia = 101;
UPDATE materias SET cedula_profesor = 1060701981 WHERE cod_materia = 102;
UPDATE materias SET cedula_profesor = 1050607123 WHERE cod_materia = 103;
UPDATE materias SET cedula_profesor = 1050607123 WHERE cod_materia = 104;
UPDATE materias SET cedula_profesor = 1141562190 WHERE cod_materia = 105;
UPDATE materias SET cedula_profesor = 1141562190 WHERE cod_materia = 106;

SELECT 
    e.nombre_completo AS "Estudiante", 
    mat.nombre AS "Materia", 
    p.nombre AS "Profesor", 
    m.nota_final AS "Nota"
FROM estudiante e
INNER JOIN matricula m ON e.cod_estudiante = m.cod_estudiante
INNER JOIN materias mat ON m.cod_materia = mat.cod_materia
INNER JOIN profesor p ON mat.cedula_profesor = p.cedula;
```
Aqui ahora si tendriamos la salida correcta teniendo un que un estudiante no mira una materia con dos profesores distintos a la vez, aun asi vamos a crear la nueva tabla ```Curso``` para evitar mas problemas.
```sql
--Eleminamos la columna que creamos antes para crear la nueva tabla curso
ALTER TABLE materias DROP COLUMN IF EXISTS fk_materias_profesor CASCADE;
ALTER TABLE materias DROP COLUMN IF EXISTS cedula_profesor CASCADE;

--Creamos la nueva tabla curso
CREATE TABLE curso (
    id_curso INT PRIMARY KEY,
    cod_materia INT,
    cedula_profesor INT,
    ano INT,
    semestre INT,
    FOREIGN KEY (cod_materia) REFERENCES materias(cod_materia),
    FOREIGN KEY (cedula_profesor) REFERENCES profesor(cedula));

--Agregamos nuevos maestros para que nos quede una profesor por curso
INSERT INTO profesor (cedula, nombre, facultad)
VALUES
	(1020304050, 'Juan Dueñas', 'Medicina'),
	(1030405060, 'Carlos Carreño', 'Medicina'),
  (1040506070, 'Felipe Castro', 'Derecho, Ciencias Políticas y Sociales'),
	(1050607080, 'Sara Romero', 'Artes');

--Agregamos la nueva estudiante de medicina
INSERT INTO estudiante (cod_estudiante, nombre_completo, email, fecha_nacimiento, estado, cod_carrera)
VALUES
	(130001, 'Alejandro Restrepo Urrego', 'alerestrepo@unal.edu.co', '2006-11-05', 'Activo', 5);

--Ahora podemos llenar la tabla curso
INSERT INTO curso (id_curso, cod_materia, cedula_profesor, anio, semestre)
VALUES
    (1, 101, 1050607920, 2026, 1), 
    (2, 102, 1060701981, 2026, 1), 
    (3, 103, 1050607123, 2025, 2), 
    (4, 104, 1040506070, 2026, 1), 
    (5, 105, 1141562190, 2025, 1), 
    (6, 106, 1050607080, 2025, 2), 
    (9, 107, 1020304050, 2026, 1), 
    (10, 108, 1030405060, 2026, 1);

--Como en nuestro nuevo modelo trasladamos el año y semestre a curso, entonces lo mejor es eliminar la tabla matriculas y reinsertar datos
DROP TABLE IF EXISTS matricula;

CREATE TABLE matricula (
	cod_estudiante INT,
	id_curso INT,
	nota_final FLOAT,
	PRIMARY KEY (cod_estudiante, id_curso),
	FOREIGN KEY (cod_estudiante) REFERENCES estudiante(cod_estudiante),
	FOREIGN KEY (id_curso) REFERENCES curso(id_curso));

INSERT INTO matricula (cod_estudiante, id_curso, nota_final)
VALUES
	(100056, 5, 2.5),
	(100056, 6, 2.1),
	(100100, 1, 4.5),
	(100100, 2, 4.0),
	(110120, 3, 4.2),
	(110120, 4, 4.5),
	(120180, 1, 3.0),
	(120180, 2, 4.8),
	(130001, 9, 4.5),
	(130001, 10, 3.8);

--Ahora si podemos hacer la consulta planteada
SELECT 
    e.nombre_completo AS "Estudiante", 
    mat.nombre AS "Materia", 
    p.nombre AS "Profesor", 
    m.nota_final AS "Nota"
FROM estudiante e
INNER JOIN matricula m ON e.cod_estudiante = m.cod_estudiante
INNER JOIN curso c ON m.id_curso = c.id_curso
INNER JOIN materias mat ON c.cod_materia = mat.cod_materia
INNER JOIN profesor p ON c.cedula_profesor = p.cedula;
```
5. Listar todos los estudiantes aunque no tengan matrículas registradas.
Todos nuestros estudiantes estan matriculados pero en caso de que alguno no lo estuviese es donde entra el uso del ```LEFT JOIN```, que mantendra la info de la tabla de la izquierda a si no este totalmente ligada a la tabla de la derecha.
```sql
SELECT 
	e.cod_estudiante AS "código",
	e.nombre_completo AS "estudiante"
FROM estudiante e LEFT JOIN matricula m ON e.cod_estudiante = m.cod_estudiante;
```

6. Mostrar las materias que no tienen ningún estudiante matriculado.
Lo mismo que en la pasada, como todas nuestras materias tienen estudiantes matriculadas, entonces nuestra salida sera vacia, pero si fuese el caso donde ocurriese, añadiriamos una especificiación donde el cod_estudiante sea ```NULL```
```sql
INSERT INTO materias (cod_materia, nombre, facultad)
VALUES (111, 'Neuronatomía Clínica'
SELECT 
	mat.cod_materia AS "codigo",
	mat.nombre AS "materia"
FROM materias mat
LEFT JOIN curso c ON mat.cod_materia = c.cod_materia
LEFT JOIN matricula m ON c.id_curso = m.id_curso
WHERE m.cod_estudiante IS NULL;
```
