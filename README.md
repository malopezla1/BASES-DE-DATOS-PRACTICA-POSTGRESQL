# BASES-DE-DATOS-PRACTICA-POSTGRESQL

Para la siguiente practica en postgreSQL, se creo una base de datos bastante simple de la estructura de una universidad en su proceso de registro de notas, estudiantes, profesores, etc..., a partir de su respectivo diagrama MER y MR por lo cual no fue necesario normalizar.

Se le aplicara a la base de datos, editada en pgAdmin4, los procesos de DDL, DML y secuencias PSQL.

# DDL (Data Definition Language): 
Es el proceso de definir, crear y modificar la estructura de los objetos en una base de datos, es decir, columnas. 

```sql
CREATE TABLE carrera (
    cod_carrera int PRIMARY KEY,
    nombre VARCHAR(100),
    facultad VARCHAR(100));
CREATE TABLE materias (
    cod_materia int PRIMARY KEY,
    nombre VARCHAR(100),
    facultad VARCHAR(100));
CREATE TABLE profesor (
    cedula int PRIMARY KEY,
    nombre VARCHAR(100),
    facultad VARCHAR(100));
CREATE TABLE estudiante (
    cod_estudiante int PRIMARY KEY,
    nombre_completo VARCHAR(150),
    email VARCHAR (100),
    fecha_nacimiento DATE,
    estado VARCHAR(50),
    cod_carrera int,
    FOREIGN KEY (cod_carrera) REFERENCES carrera(cod_carrera));
CREATE TABLE matricula (
    cod_estudiante int,
    cod_materia int,
    ano int,
    semestre int,
    nota_final float,
    PRIMARY KEY (cod_estudiante, cod_materia, ano, semestre),
    FOREIGN KEY (cod_estudiante) REFERENCES estudiante(cod_estudiante),
    FOREIGN KEY (cod_materia) REFERENCES materias(cod_materia));
```

Hemos definido y creado la estructura, solo queda el proceso de modificar, asi que para ejemplificar el ultimo paso de el DLL (modificar), haremos uso de ```DROP``` y ```ALTER``` para eliminar una columna, luego volverla a crear. Aunque esto se puede usar usando simplemente la función ```RENAME```, lo hacemos por el metodo largo para mostrar el uso de estas funciones tan usadas.

## Eliminar Columna "Ano" de la tabla matricula
```sql
ALTER TABLE matricula DROP COLUMN ano;
```

## Crear la nueva Columna "Anio"
```sql
ALTER TABLE matricula ADD anio int;
```

Con esto finalizaria la explicación y ejemplificación de lo que consiste aplica DLL en las bases de datos.
Seguimos con la aplicación de DML.

# DML (Data Manipulation Lenguage)
Consiste en la gestión, modificación y recuperación de datos dentro de una base da datos. Siendo sus cuatro comandos basicos: ```SELECT```, ```INSERT```, ```UPDATE``` y ```DELETE```.

Insertamos datos en cada tabla de la siguiente forma, haciendo uso de la función ```INSERT```:
(Las siguientes lineas de código muestran el proceso de insertar datos antes del cambio de nombre de la columna ano a anio)

## Tabla Carrera
```sql
INSERT INTO carrera (cod_carrera, nombre, facultad)
VALUES
	('1', 'Ingenieria de Sistemas y Computación', 'Ingenieria'),
	('2', 'Ingenieria Industrial', 'Ingenieria'),
	('3', 'Ciencias Políticas', 'Derecho, Ciencias Políticas y Sociales'),
	('4', 'Arquitectura', 'Artes');
```

## Tabla Profesor
```sql
INSERT INTO profesor (cedula, nombre, facultad)
VALUES
	('1050607920', 'Julian Garcia', 'Ingenieria'),
	('1060701981', 'Santiago Niño', 'Ingenieria'),
	('1050607123', 'David Moreno', 'Derecho, Ciencias Políticas y Sociales'),
	('1141562190', 'Luna Santis', 'Artes');
```

## Tabla Materias
```sql
INSERT INTO materias (cod_materia, nombre, facultad)
VALUES
	('101', 'Bases de Datos', 'Ingenieria'),
	('102', 'Calculo Integral', 'Ingenieria'),
	('103', 'Teorías del estado', 'Derecho, Ciencias Políticas y Sociales'),
	('104', 'Procesos Políticos de América Latina', 'Derecho, Ciencias Políticas y Sociales'),
	('105', 'Proyecto Arquitectonico I', 'Artes'),
	('106', 'Representación I', 'Artes');
```

## Tabla Estudiante
```sql
INSERT INTO estudiante (cod_estudiante, nombre_completo, email, fecha_nacimiento, estado, cod_carrera)
VALUES
	(100100, 'John Alexander Guzman Barreto', 'joabare@unal.edu.co', '2007-07-27', 'Activo', 2),
	(120180, 'David Jesus Quintero Forero', 'daquiefo@unal.edu.co', '2007-01-16', 'Activo', 1),
	(110120, 'Maria Danna Niño Galindo', 'madaniga@unal.edu.co', '2008-01-21', 'Activo', 3),
	(100056, 'Laura Valeria Garcia Daza', 'ladaza@unal.edu.co', '2005-04-12', 'Inactivo', 4);
```

## Tabla Matricula
```sql
INSERT INTO matricula(cod_estudiante, cod_materia, ano, semestre, nota_final)
VALUES 
	(100056, 105, 2025, 1, 2.5),
	(100056, 106, 2025, 2, 1.7),
	(100100, 101, 2026, 1, 4.5),
	(100100, 102, 2026, 1, 4.0),
	(110120, 103, 2025, 2, 4.2),
	(110120, 104, 2026, 1, 4.5),
	(120180, 101, 2026, 1, 3.0),
	(120180, 102, 2026, 1, 4.8);
```

Entonces si teniamos una columna "ano" rellena de datos y la eliminamos, al crear la columna que la reempleza llamada "anio", nos toca de forma manual volver a reinsertar los datos eliminados, de la siguiente forma:

## Rellenar esa nueva columna
```sql
UPDATE matricula SET anio = 2025 WHERE cod_estudiante = 100056 AND cod_materia = 105;
UPDATE matricula SET anio = 2025 WHERE cod_estudiante = 100056 AND cod_materia = 106;
UPDATE matricula SET anio = 2026 WHERE cod_estudiante = 100100 AND cod_materia = 101;
UPDATE matricula SET anio = 2026 WHERE cod_estudiante = 100100 AND cod_materia = 102;
UPDATE matricula SET anio = 2025 WHERE cod_estudiante = 110120 AND cod_materia = 103;
UPDATE matricula SET anio = 2026 WHERE cod_estudiante = 110120 AND cod_materia = 104;
UPDATE matricula SET anio = 2026 WHERE cod_estudiante = 120180 AND cod_materia = 101;
UPDATE matricula SET anio = 2026 WHERE cod_estudiante = 120180 AND cod_materia = 102;
```

¿Qué ocurre con este metodo de actualización de una columna sin usar la función ```RENAME```?

Ocurre que esta forma de hacer el renombramiento es mas tardada, desgastaste e inefectiva, ya que nuestra columna original estaba ubicada en una posición 3, pero al eliminarla y crear una con nombre nuevo y mismos datos, cambio su posición hasta ultimo lugar. Por lo cual siempre sera mejor usar ```RENAME```, aunque no se usara para mostrar las funcionalidades de las otras 2 funciones.

Ahora vamos a ver el uso de nuestras 3 funciones basicas restantes.

## Select

Quiero saber la nota final de cada estudiante, para esto hacemos una consulta con ```SELECT```.
```sql
SELECT cod_estudiante, nota_final FROM matricula;
```
Esto mostrara la nota final junto a el codigo de cada estudiante, pero si queremos algo mas estructurado y facil de leer por un cliente podemos hacer uso de otras herramientos como los joins, que juntas datos de dos o mas tablas para una consulta mas completa, en este caso se usara el ```INNER JOIN```, que nos permite combinar filas de dos o mas tablas, para usar elementos que solo esten presentes en las dos (intersección de conjuntos).

## Select + Inner Join
```sql
SELECT 
	e.cod_estudiante,
	e.nombre_completo,
	m.cod_materia,
	m.nota_final
FROM estudiante e INNER JOIN matricula m ON e.cod_estudiante = m.cod_estudiante;
```
La e es un alias que se le asigan a la tabla estudiante para tener que evitar escribir la palabra completa, lo mismo pasa con la m.

Esto nos dara como salida el codigo de estudiante, su nombre, la materia que curso y su respectiva nota, y para ser mas claros podriamos agregar el nombre de la materia, pero lo mostraremos mas adelante. 

## Update
Pensemos que una nota registrada se hizo mal y necesita cambiarse, para hacerlo usaremos el ```UPDATE``` para modificar valores existentes. Por ejemplo, la estudiante Laura Garcia tiene registrado un 1.7 pero en realidad era un 2.1, cambiaremos ese dato. 
```sql
UPDATE matricula SET nota_final = 2.1 WHERE cod_estudiante = 100056 AND cod_materia = 106;
```
Ahora hagamos la consulta para verificar el cambio, ahora mostrado el nombre de la materia en vez de su codigo.
```sql
SELECT 
	e.cod_estudiante, 
	e.nombre_completo AS estudiante,
	mat.nombre AS materia,
	m.nota_final
FROM estudiante e 
INNER JOIN matricula m ON e.cod_estudiante = m.cod_estudiante
INNER JOIN materias mat ON m.cod_materia = mat.cod_materia
WHERE e.cod_estudiante = 100056 AND m.cod_materia = 106;
```
El ```AS``` es simplemente un apodo que le damos al encabezado original de las columnas para dar un resultado mas limpio y claro, ya que por ejemplo, el "nombre" de la materia en su tabla original es un encabezado que funciona pero aqui puede resultar ambiguo, asi que se renombra solo para esta consulta como "materia".

## Delete
Sirve para eliminar una fila de información especifica. Por lo que crearemos una nueva fila y despues la eliminaremos para ejemplificar.
```sql
INSERT INTO carrera(cod_carrera, nombre, facultad)
VALUES
	('5', 'Medicina', 'Facultad de Medicina');
```
Podemos consultar la tabla para ver que se ha creado una nueva fila para estar claros, lo mismo para saber si se elimino.
```sql
DELETE FROM carrera WHERE cod_carrera = '5';
```
Con esto quedaria eliminado.

## Secuencias PSQL
Son objetos que permiten generar números enteros unicos, lo cual es realmente util para usarse en la formación de claves primarias, como por ejemplo en ```cod_materia```.
```sql
CREATE SEQUENCE sec_cod_carrera START WITH 5;
ALTER TABLE carrera
ALTER COLUMN cod_carrera SET DEFAULT nextval('sec_cod_carrera');
```
Con esto ya no tenemos la necesidad de al ingresar datos, tener que volver a escribir manualmente el codigo de la carrera. Ejemplo:
```sql
INSERT INTO carrera(nombre, facultad)
VALUES ('Medicina', 'Medicina');
```
Ahora con la tabla de materias.
```sql
CREATE SEQUENCE sec_cod_materia START WITH 107;
ALTER TABLE materias
ALTER COLUMN cod_materia SET DEFAULT nextval('sec_cod_materia');

INSERT INTO materias(nombre, facultad)
VALUES 
	('Anatomía Humana y Embriologia', 'Medicina'),
	('Bioquímica y Fisiología', 'Medicina');
```

Ahora se continuara con el apartado de joins y consultas avanzadas.
[Ir a Consultas Avanzadas (Joins)](CONSULTAS_AVANZADAS.md)
