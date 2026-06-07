# BASES-DE-DATOS-PRACTICA-POSTGRESQL

Para la siguiente practica en postgreSQL, se creo una base de datos bastante simple de la estructura de una universidad en su proceso de registro de notas, estudiantes, profesores, etc..., a partir de su respectivo diagrama MER y MR por lo cual no fue necesario normalizar.

Se le aplicara a la base de datos, editada en pgAdmin4, los procesos de DDL, DML y PSQL.

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







