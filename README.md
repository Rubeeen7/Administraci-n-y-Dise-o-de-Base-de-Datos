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
