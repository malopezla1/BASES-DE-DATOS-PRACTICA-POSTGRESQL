# BASES-DE-DATOS-PRACTICA-POSTGRESQL

Para la siguiente practica en postgreSQL, se creo una base de datos bastante simple de la estructura de una universidad en su proceso de registro de notas, estudiantes, profesores, etc..., a partir de su respectivo diagrama MER y MR por lo cual no fue necesario normalizar.

Se le aplicara a la base de datos, editada en pgAdmin4, los procesos de DDL, DML y PSQL.

# DDL (Data Definition Language): 
Es el proceso de definir, crear y modificar la estructura de los objetos en una base de datos. 

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
