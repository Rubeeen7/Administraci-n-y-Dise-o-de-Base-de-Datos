# Práctica 1 - Conceptos fundamentales de PostgreSQL

## 1. Creación de la base de datos

Se crea una base de datos llamada `biblioteca`.

### Comando empleado

```sql
CREATE DATABASE biblioteca;
```

### Salida obtenida

```text
CREATE DATABASE
```

### Comprobación

Para comprobar que la base de datos se ha creado correctamente se ejecuta:

```text
\l
```

En la salida aparece la base de datos `biblioteca`.

## 2. Creación de usuarios

### 2.a Creación de los usuarios

Se crean los usuarios `admin_biblio` y `usuario_biblio`.

### Comandos empleados

```sql
CREATE USER admin_biblio WITH PASSWORD 'admin123';

CREATE USER usuario_biblio WITH PASSWORD 'usuario123';
```

### Salida obtenida

```text
CREATE ROLE
CREATE ROLE
```

### Comprobación

Para comprobar que los usuarios se han creado correctamente, se consulta la tabla del sistema `pg_roles`:

```sql
SELECT rolname
FROM pg_roles
WHERE rolname IN ('admin_biblio', 'usuario_biblio');
```

### Salida obtenida

```text
     rolname
-----------------
 admin_biblio
 usuario_biblio
(2 rows)
```
### Asignación de permisos a `admin_biblio`

Se conceden todos los privilegios sobre la base de datos `biblioteca` al usuario `admin_biblio`.

### Comando empleado

```sql
GRANT ALL PRIVILEGES ON DATABASE biblioteca TO admin_biblio;
```

### Salida obtenida

```text
GRANT
```

A continuación, se accede a la base de datos `biblioteca`:

```text
\c biblioteca
```

Y se conceden permisos sobre el esquema `public`:

```sql
GRANT ALL ON SCHEMA public TO admin_biblio;
```

### Salida obtenida

```text
GRANT
```

### Comprobación

Para comprobar los privilegios concedidos sobre la base de datos se ejecuta:

```text
\l biblioteca
```

### Salida obtenida

```text
postgres=CTc/postgres
admin_biblio=CTc/postgres
```

Esto confirma que el usuario `admin_biblio` dispone de privilegios sobre la base de datos `biblioteca`.

### 2.b Creación del rol `lectores`

Se crea el rol `lectores`, que tendrá permisos únicamente de consulta sobre las tablas de la base de datos.

### Comandos empleados

```sql
CREATE ROLE lectores;
```

```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lectores;
```

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO lectores;
```

### Salida obtenida

```text
CREATE ROLE
GRANT
ALTER DEFAULT PRIVILEGES
```

### Comprobación

Para comprobar que el rol se ha creado correctamente se consulta la tabla del sistema `pg_roles`:

```sql
SELECT rolname
FROM pg_roles
WHERE rolname = 'lectores';
```

### Salida obtenida

```text
 rolname
----------
 lectores
(1 row)
```
También se conceden al rol `lectores` los permisos necesarios para conectarse a la base de datos `biblioteca` y acceder al esquema `public`.

```sql
GRANT CONNECT ON DATABASE biblioteca TO lectores;
```

### Salida obtenida

```text
GRANT
```

```sql
GRANT USAGE ON SCHEMA public TO lectores;
```

### Salida obtenida

```text
GRANT
```

### 2.c Asignación del usuario al rol `lectores`

Se asigna el usuario `usuario_biblio` al rol `lectores`. De esta forma, el usuario hereda los permisos de consulta concedidos previamente a dicho rol.

### Comando empleado

```sql
GRANT lectores TO usuario_biblio;
```

### Salida obtenida

```text
GRANT ROLE
```

### Comprobación

Para comprobar que `usuario_biblio` pertenece al rol `lectores` se realiza la siguiente consulta:

```sql
SELECT r.rolname AS rol, u.rolname AS usuario
FROM pg_auth_members m
JOIN pg_roles r ON m.roleid = r.oid
JOIN pg_roles u ON m.member = u.oid
WHERE r.rolname = 'lectores'
  AND u.rolname = 'usuario_biblio';
```

### Salida obtenida

```text
   rol    |    usuario
----------+-----------------
 lectores | usuario_biblio
(1 row)
```
### 2.d Consulta de los usuarios y roles creados

Se consulta la tabla del sistema `pg_roles` para comprobar los usuarios y roles creados durante la práctica.

### Comando empleado

```sql
SELECT rolname
FROM pg_roles
WHERE rolname IN ('admin_biblio', 'usuario_biblio', 'lectores');
```

### Salida obtenida

```text
     rolname
-----------------
 admin_biblio
 lectores
 usuario_biblio
(3 rows)
```
### 2.e Cambio de contraseña de `usuario_biblio`

Se modifica la contraseña del usuario `usuario_biblio`.

### Comando empleado

```sql
ALTER USER usuario_biblio WITH PASSWORD 'nueva123';
```

### Salida obtenida

```text
ALTER ROLE
```

### Comprobación

Se comprueba que el usuario continúa existiendo en `pg_roles`:

```sql
SELECT rolname
FROM pg_roles
WHERE rolname = 'usuario_biblio';
```

### Salida obtenida

```text
     rolname
-----------------
 usuario_biblio
(1 row)
```
### Comprobación de acceso

Se comprueba el acceso a la base de datos `biblioteca` con el usuario `usuario_biblio` utilizando la nueva contraseña.

```bash
psql -U usuario_biblio -d biblioteca -h localhost -W
```

Tras introducir la nueva contraseña, se accede correctamente a PostgreSQL como `usuario_biblio`.

### 2.f Restricción del permiso de eliminación

Se configura el usuario `usuario_biblio` para que no pueda eliminar registros de las tablas de la base de datos.

Como el usuario pertenece al rol `lectores`, se revoca el permiso `DELETE` tanto al usuario como al propio rol.

### Comandos empleados

```sql
REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM usuario_biblio;

REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM lectores;
```

### Salida obtenida

```text
REVOKE
REVOKE
```

De esta forma, `usuario_biblio` no dispone de permisos para eliminar registros de las tablas.

## 3. Creación de tablas

### 3.a Creación de las tablas

Se crean las tablas `autores`, `libros` y `prestamos`, definiendo sus respectivas claves primarias.

### Comandos empleados

```sql
CREATE TABLE autores (
    id_autor SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    nacionalidad VARCHAR(50)
);
```

```sql
CREATE TABLE libros (
    id_libro SERIAL PRIMARY KEY,
    titulo VARCHAR(150),
    anio_publicacion INTEGER,
    id_autor INTEGER
);
```

```sql
CREATE TABLE prestamos (
    id_prestamo SERIAL PRIMARY KEY,
    id_libro INTEGER,
    fecha_prestamo DATE,
    fecha_devolucion DATE,
    usuario_prestatario VARCHAR(100)
);
```

### Salida obtenida

```text
CREATE TABLE
CREATE TABLE
CREATE TABLE
```

### Comprobación

Para comprobar que las tablas se han creado correctamente se utiliza:

```text
\dt
```

### Salida obtenida

```text
           List of relations
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | autores   | table | postgres
 public | libros    | table | postgres
 public | prestamos | table | postgres
```
### 3.b Establecimiento de claves foráneas

Se establecen las relaciones entre las tablas mediante claves foráneas.

La tabla `libros` referencia a la tabla `autores` mediante el campo `id_autor`, mientras que la tabla `prestamos` referencia a la tabla `libros` mediante el campo `id_libro`.

### Comandos empleados

```sql
ALTER TABLE libros
ADD CONSTRAINT fk_libros_autores
FOREIGN KEY (id_autor)
REFERENCES autores(id_autor);
```

```sql
ALTER TABLE prestamos
ADD CONSTRAINT fk_prestamos_libros
FOREIGN KEY (id_libro)
REFERENCES libros(id_libro)
ON DELETE CASCADE;
```

### Salida obtenida

```text
ALTER TABLE
ALTER TABLE
```

### Comprobación

Para comprobar las restricciones establecidas se consulta la definición de las tablas:

```text
\d libros
```

```text
\d prestamos
```

En la salida aparecen las claves foráneas `fk_libros_autores` y `fk_prestamos_libros`, confirmando que las relaciones entre las tablas se han creado correctamente.

## 4. Inserción de datos

### 4.a Inserción de datos de ejemplo

Se insertan 5 autores, 8 libros y 5 préstamos de ejemplo en las tablas creadas anteriormente.

### Comandos empleados

```sql
INSERT INTO autores (nombre, nacionalidad) VALUES
('Gabriel Garcia Marquez', 'Colombiana'),
('George Orwell', 'Britanica'),
('Miguel de Cervantes', 'Española'),
('J. K. Rowling', 'Britanica'),
('Stephen King', 'Estadounidense');
```

```sql
INSERT INTO libros (titulo, anio_publicacion, id_autor) VALUES
('Cien años de soledad', 1967, 1),
('El amor en los tiempos del colera', 1985, 1),
('1984', 1949, 2),
('Rebelion en la granja', 1945, 2),
('Don Quijote de la Mancha', 1605, 3),
('Harry Potter y la piedra filosofal', 1997, 4),
('El resplandor', 1977, 5),
('It', 1986, 5);
```

```sql
INSERT INTO prestamos
(id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(1, '2026-09-01', '2026-09-10', 'Ana'),
(2, '2026-09-03', NULL, 'Pedro'),
(3, '2026-09-05', '2026-09-15', 'Lucia'),
(4, '2026-09-07', NULL, 'Carlos'),
(5, '2026-09-10', NULL, 'Ana');
```

### Salida obtenida

```text
INSERT 0 5
INSERT 0 8
INSERT 0 5
```

### Comprobación

Para comprobar los datos insertados se realizan las siguientes consultas:

```sql
SELECT * FROM autores;
```

```sql
SELECT * FROM libros;
```

```sql
SELECT * FROM prestamos;
```

Las consultas muestran los 5 autores, 8 libros y 5 préstamos insertados correctamente.

## 5. Consultas básicas

### 5.a Listado de libros con su autor

Se muestran todos los libros registrados junto con el autor correspondiente.

### Comando empleado

```sql
SELECT libros.titulo, autores.nombre AS autor
FROM libros
JOIN autores
ON libros.id_autor = autores.id_autor;
```

### Salida obtenida

```text
               titulo                |          autor
-------------------------------------+--------------------------
 Cien años de soledad                | Gabriel Garcia Marquez
 El amor en los tiempos del colera   | Gabriel Garcia Marquez
 1984                                | George Orwell
 Rebelion en la granja               | George Orwell
 Don Quijote de la Mancha            | Miguel de Cervantes
 Harry Potter y la piedra filosofal  | J. K. Rowling
 El resplandor                       | Stephen King
 It                                  | Stephen King
(8 rows)
```
### 5.b Préstamos pendientes de devolución

Se muestran los préstamos que todavía no tienen una fecha de devolución registrada.

### Comando empleado

```sql
SELECT *
FROM prestamos
WHERE fecha_devolucion IS NULL;
```

### Salida obtenida

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           2 |        2 | 2026-09-03     |                  | Pedro
           4 |        4 | 2026-09-07     |                  | Carlos
           5 |        5 | 2026-09-10     |                  | Ana
(3 rows)
```
### 5.c Autores con más de un libro registrado

Se obtienen los autores que tienen más de un libro registrado en la base de datos.

### Comando empleado

```sql
SELECT autores.nombre, COUNT(libros.id_libro) AS numero_libros
FROM autores
JOIN libros
ON autores.id_autor = libros.id_autor
GROUP BY autores.id_autor, autores.nombre
HAVING COUNT(libros.id_libro) > 1;
```

### Salida obtenida

```text
        nombre         | numero_libros
-----------------------+--------------
 Stephen King          |            2
 George Orwell         |            2
 Gabriel Garcia Marquez|            2
(3 rows)
```
## 6. Consultas con agregación

### 6.a Número total de préstamos realizados

Se calcula el número total de préstamos registrados en la tabla `prestamos`.

### Comando empleado

```sql
SELECT COUNT(*) AS total_prestamos
FROM prestamos;
```

### Salida obtenida

```text
 total_prestamos
-----------------
               5
(1 row)
```
### 6.b Número de libros prestados por usuario

Se obtiene el número de libros prestados por cada usuario registrado en la tabla `prestamos`.

### Comando empleado

```sql
SELECT usuario_prestatario, COUNT(*) AS libros_prestados
FROM prestamos
GROUP BY usuario_prestatario;
```

### Salida obtenida

```text
 usuario_prestatario | libros_prestados
---------------------+------------------
 Carlos              |                1
 Pedro               |                1
 Ana                 |                2
 Lucia               |                1
(4 rows)
```

## 7. Modificación de datos

### 7.a Actualización de la fecha de devolución

Se actualiza la fecha de devolución de uno de los préstamos que se encontraba pendiente.

### Comando empleado

```sql
UPDATE prestamos
SET fecha_devolucion = '2026-09-20'
WHERE id_prestamo = 2;
```

### Salida obtenida

```text
UPDATE 1
```

### Comprobación

Para comprobar que la fecha de devolución se ha actualizado correctamente se consulta el préstamo modificado:

```sql
SELECT *
FROM prestamos
WHERE id_prestamo = 2;
```

### Salida obtenida

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           2 |        2 | 2026-09-03     | 2026-09-20       | Pedro
(1 row)
```
### 7.b Eliminación de un libro y efecto en los préstamos

Se elimina un libro que tiene préstamos asociados para comprobar el comportamiento de la clave foránea definida con `ON DELETE CASCADE`.

### Comprobación previa

Se consultan los préstamos asociados al libro con `id_libro = 4`:

```sql
SELECT *
FROM prestamos
WHERE id_libro = 4;
```

### Salida obtenida

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           4 |        4 | 2026-09-07     |                  | Carlos
(1 row)
```

### Eliminación del libro

```sql
DELETE FROM libros
WHERE id_libro = 4;
```

### Salida obtenida

```text
DELETE 1
```

### Comprobación posterior

Se vuelve a consultar la tabla `prestamos` para comprobar si siguen existiendo registros asociados al libro eliminado:

```sql
SELECT *
FROM prestamos
WHERE id_libro = 4;
```

### Salida obtenida

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
(0 rows)
```

El préstamo asociado se elimina automáticamente debido a la opción `ON DELETE CASCADE` definida en la clave foránea `fk_prestamos_libros`.

## 8. Creación de vistas

### 8.a Creación de la vista `vista_libros_prestados`

Se crea una vista que muestra el título del libro, el autor y el usuario prestatario de cada préstamo registrado.

### Comando empleado

```sql
CREATE VIEW vista_libros_prestados AS
SELECT libros.titulo,
       autores.nombre AS autor,
       prestamos.usuario_prestatario
FROM prestamos
JOIN libros
ON prestamos.id_libro = libros.id_libro
JOIN autores
ON libros.id_autor = autores.id_autor;
```

### Salida obtenida

```text
CREATE VIEW
```

### Comprobación

Para comprobar el contenido de la vista se realiza la siguiente consulta:

```sql
SELECT * FROM vista_libros_prestados;
```

### Salida obtenida

```text
              titulo               |         autor          | usuario_prestatario
-----------------------------------+------------------------+---------------------
 Cien años de soledad              | Gabriel Garcia Marquez | Ana
 1984                              | George Orwell           | Lucia
 Don Quijote de la Mancha          | Miguel de Cervantes     | Ana
 El amor en los tiempos del colera | Gabriel Garcia Marquez | Pedro
(4 rows)
```
### 8.b Permisos de consulta sobre la vista

Se concede al usuario `usuario_biblio` permiso de consulta sobre la vista `vista_libros_prestados`.

### Comando empleado

```sql
GRANT SELECT ON vista_libros_prestados TO usuario_biblio;
```

### Salida obtenida

```text
GRANT
```

### Comprobación

Para comprobar los privilegios asignados sobre la vista se ejecuta:

```text
\dp vista_libros_prestados
```

### Salida obtenida

```text
                                  Access privileges
 Schema |          Name           | Type |        Access privileges         | Column privileges | Policies
--------+-------------------------+------+----------------------------------+-------------------+----------
 public | vista_libros_prestados | view | postgres=arwdDxt/postgres       +|                   |
        |                         |      | lectores=r/postgres             +|                   |
        |                         |      | usuario_biblio=r/postgres        |                   |
(1 row)
```

El privilegio `r` indica permiso de lectura (`SELECT`) sobre la vista.

## 9. Funciones y consultas avanzadas

### 9.a Función para obtener los libros de un autor

Se crea una función que recibe el nombre de un autor y devuelve todos los libros escritos por dicho autor.

### Comando empleado

```sql
CREATE OR REPLACE FUNCTION libros_por_autor(nombre_autor VARCHAR)
RETURNS TABLE (
    id_libro INTEGER,
    titulo VARCHAR,
    anio_publicacion INTEGER
)
AS $$
BEGIN
    RETURN QUERY
    SELECT l.id_libro,
           l.titulo,
           l.anio_publicacion
    FROM libros l
    JOIN autores a
    ON l.id_autor = a.id_autor
    WHERE a.nombre = nombre_autor;
END;
$$ LANGUAGE plpgsql;
```

### Salida obtenida

```text
CREATE FUNCTION
```

### Comprobación

Se prueba la función con el autor `George Orwell`:

```sql
SELECT *
FROM libros_por_autor('George Orwell');
```

### Salida obtenida

```text
 id_libro | titulo | anio_publicacion
----------+--------+------------------
        3 | 1984   |             1949
(1 row)
```
